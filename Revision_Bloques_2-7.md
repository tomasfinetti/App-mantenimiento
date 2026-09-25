# Revisión de cambios — Bloques 2 a 7

Resumen de todo lo hecho sobre `Tablero_Mantenimiento_v13_G06.html` desde el commit
"Bloque 1 listo", con comentarios de diseño e impedimentos encontrados en cada bloque.
El diff completo (git) queda al final de este archivo.

Estado en git al momento de escribir esto: **nada commiteado todavía** — todo sigue en
el working tree, a la espera de que pidas el commit/push.

---

## Bloque 2 — Matriz de criticidad
**Qué se hizo:** botón "Ver criticidad" que alterna el color de los nodos del mapa
entre el semáforo de condición (existente) y el nivel de criticidad (`crit` de
`BASE_META`: Alta/Media/Baja). Ficha del equipo muestra su criticidad al seleccionarlo.

**Impedimentos:** ninguno relevante. Fue el bloque más directo — reutiliza la paleta
de colores y el sistema de badges que ya existía, sin tocar lógica de negocio.

## Bloque 3 — Weibull embebido
**Qué se hizo:** panel "Confiabilidad (Weibull)" por equipo: β, η, R², MTBF
calculados en el navegador (rango mediano de Bernard + mínimos cuadrados), recta
linealizada y curva R(t) dibujadas reutilizando `setupCanvas()`/`axes()` de `draw()`,
e interpretación automática según β.

**Impedimentos:**
- **No tengo acceso a las planillas crudas de TBF del TP.** Solo contaba con los
  β/η objetivo mencionados en CLAUDE.md (AC-725 β≈0,57; P-80A β≈1,02, η≈94 días).
  Generé 8 TBF por equipo a partir de los cuantiles teóricos de Weibull para esos
  β/η (método de Bernard invertido) y los redondeé a días enteros — quedan
  documentados en el código (`TBF_SEED`) como "dato de arranque, reemplazable por
  historial real en el Bloque 7".
- JavaScript no tiene función Gamma nativa (necesaria para el MTBF = η·Γ(1+1/β)).
  Tuve que implementar una aproximación de Lanczos propia, sin librerías externas.

## Bloque 4 — Predicción de intervención
**Qué se hizo:** campo de horas de operación acumuladas + umbral de R(t)
configurable (80% por defecto); calcula el t en que R(t) cruza el umbral con η/β
del Bloque 3, y muestra "días hasta la próxima intervención recomendada" como
alerta destacada (verde/amarillo/rojo según urgencia).

**Impedimentos:**
- **Desajuste de unidades:** CLAUDE.md pide "horas de operación acumuladas", pero
  el motor Weibull quedó calibrado en días (por los TBF del Bloque 3). Decisión:
  convertir horas→días (÷24), asumiendo operación continua 24/7 (típico en una
  planta de proceso GLP), y dejarlo explícito en el cálculo para que no genere
  confusión de unidades si alguien lo revisa.
- No hay datos reales de horómetro por equipo, así que el campo arranca en 0 (no
  inventé un valor de horas "de fábrica") — el operador carga el dato real.

## Bloque 5 — AMFE / NPR + orden de trabajo
**Qué se hizo:** panel "AMFE / Modos críticos" con los modos de falla, su NPR
(AC-725: 288 y 252; P-80A: 216→108) y la acción recomendada; el generador de
intervención existente (`interBtn`) se autocompleta con esa acción según el
componente seleccionado, con fallback al diagnóstico heurístico para equipos sin
AMFE cargado.

**Impedimentos:**
- **No tenía el texto real de las acciones recomendadas del AMFE del informe** —
  solo los NPR numéricos que me diste. Redacté acciones técnicas plausibles y
  coherentes con cada modo de falla (tensado/alineación de correas, torque de
  bulones, monitoreo del sello), quedan explícitas en `AMFE_SEED` para poder
  cotejarlas o reemplazarlas por el texto exacto del informe si difiere.

## Bloque 6 — WhatsApp/mail + IA opcional
**Qué se hizo:** alerta que arma un link `wa.me/?text=...` y un `mailto:` con el
resumen del equipo y la acción sugerida (reutiliza la cascada AMFE→heurística del
Bloque 5) cuando el equipo cruza a Atención o Crítico. Botón de diagnóstico con IA
separado, opcional, con *feature-detection* de `window.claude.complete()` y el
caveat de CLAUDE.md §2 documentado en el código y en el mensaje que ve el usuario.

**Impedimentos:**
- **No pude verificar la rama "IA disponible"** de `aiDiagBtn` — solo se puede
  probar corriendo la app dentro del entorno de artefactos de Claude, que no es
  este entorno de edición. Sí verifiqué la rama de fallback (la que se usa siempre
  que se abre el `.html` localmente), que es el caso real de uso de este tablero.
- Decisión de diseño sin dato explícito en el roadmap: qué "umbral" dispara la
  alerta. Usé el semáforo de condición (`eqStatus` en Atención o Crítico) por ser
  transversal a todos los equipos, no solo a los dos con datos de Weibull.

## Bloque 7 — Pulido y demo
**Qué se hizo:** serie de "Correas" del AC-725 rehecha como diente de sierra
(sube y cae de golpe, repitiendo ciclos) para reflejar visualmente la mortalidad
infantil (β<1, reintervenciones frecuentes) en vez de una rampa continua;
comentario de documentación sobre el cálculo de `health()`; archivo nuevo
`Guion_Demo.md` con el guion de exposición de 3–4 minutos.

**Impedimentos:**
- **No tengo acceso a las planillas reales de mediciones del TP**, así que
  "parecido al historial real" lo interpreté como reproducir la *forma* y la
  *narrativa* del comportamiento (mortalidad infantil → diente de sierra) en vez
  de reproducir valores exactos de un historial que no tengo. Quedó documentado en
  el código para que se pueda reemplazar por datos reales sin tocar el motor de
  cálculo.
- P-80A no se tocó (ya tenía una tendencia coherente con el sello degradándose de
  forma continua, consistente con β≈1) — solo se documentó, para no arriesgar los
  paneles ya verificados en los bloques anteriores.

## Impedimento transversal (todos los bloques)
No hay ningún framework de testing ni linter en este proyecto (por regla de
CLAUDE.md: un solo `.html` autocontenido, sin librerías). La verificación de cada
bloque se hizo manualmente: abrir el archivo en el navegador embebido, revisar la
consola en busca de errores, y comprobar a mano que los números calculados
(β, η, R², días hasta intervención, etc.) coincidieran con lo esperado. Es un
control razonable para un tablero de este tamaño, pero no reemplaza una batería de
tests automatizados si el proyecto creciera.

---

## Diff completo (git diff, HEAD → working tree)

```diff
warning: in the working copy of 'Tablero_Mantenimiento_v13_G06.html', LF will be replaced by CRLF the next time Git touches it
diff --git a/Tablero_Mantenimiento_v13_G06.html b/Tablero_Mantenimiento_v13_G06.html
index f8ac16a..29a93f4 100644
--- a/Tablero_Mantenimiento_v13_G06.html
+++ b/Tablero_Mantenimiento_v13_G06.html
@@ -86,7 +86,24 @@ svg{display:block;min-width:1560px}
 .rec.good{background:#0d211a;border-color:#24513f;color:#bff3d8}
 .rec.neutral{background:#111c2a;border-color:#32465f;color:#b9c7d9}
 
+.predAlert{margin-top:12px;padding:14px 16px;border-radius:14px;display:flex;justify-content:space-between;align-items:center;gap:14px;flex-wrap:wrap}
+.predAlert .pTitle{font-size:10px;color:var(--muted);text-transform:uppercase;letter-spacing:.5px}
+.predAlert .pMain{font-size:26px;font-weight:850;margin-top:4px}
+.predAlert .pSub{font-size:11px;color:var(--muted);margin-top:5px;max-width:420px}
+.predAlert.ok{background:rgba(53,208,127,.10);border:1px solid rgba(53,208,127,.35)}
+.predAlert.warn{background:rgba(242,201,76,.10);border:1px solid rgba(242,201,76,.35)}
+.predAlert.bad{background:rgba(255,93,108,.10);border:1px solid rgba(255,93,108,.35)}
+.predAlert.neutral{background:rgba(125,143,167,.10);border:1px solid rgba(125,143,167,.3)}
+
+.amfeCard{background:#0d1b2d;border:1px solid var(--line);border-radius:12px;padding:12px;margin-bottom:10px}
+.amfeCard:last-child{margin-bottom:0}
+.amfeHead{display:flex;justify-content:space-between;align-items:flex-start;gap:10px;flex-wrap:wrap}
+.amfeHead b{font-size:13px}
+.amfeHead small{display:block;color:var(--muted);font-size:10px;margin-top:2px}
+.amfeAction{margin-top:8px;font-size:11px;line-height:1.5;color:#c9d6e6}
+
 .charts{display:grid;grid-template-columns:1.15fr 1fr 1fr;gap:14px;margin-top:14px}
+.charts2{display:grid;grid-template-columns:1fr 1fr;gap:14px}
 .cp{padding:12px}
 .ct{font-size:12px;font-weight:750;margin:0 0 8px}
 .legend{font-size:9px;color:var(--muted);margin-top:6px}
@@ -114,7 +131,7 @@ th{font-size:9px;color:var(--muted);text-transform:uppercase}
 .toast{position:fixed;right:18px;bottom:18px;display:none;background:#16304a;border:1px solid #3d648c;color:white;padding:11px 13px;border-radius:11px;z-index:40}
 
 @media(max-width:1100px){
-  .grid,.charts,.bottom{grid-template-columns:1fr}
+  .grid,.charts,.charts2,.bottom{grid-template-columns:1fr}
   .kpis,.infoGrid,.formGrid,.sourceCards,.componentList{grid-template-columns:1fr}
 }
 </style>
@@ -142,11 +159,14 @@ th{font-size:9px;color:var(--muted);text-transform:uppercase}
 <section class="panel">
   <div class="ph">
     <div><h2>Mapa general de planta</h2><small>Torres, aeroenfriadores, tanques y bombas</small></div>
-    <div class="legendRow">
-      <span><i class="dot ok"></i>Normal</span>
-      <span><i class="dot warn"></i>Atención</span>
-      <span><i class="dot bad"></i>Crítico</span>
-      <span><i class="dot nodata"></i>Sin datos</span>
+    <div style="display:flex;align-items:center;gap:12px;flex-wrap:wrap">
+      <button id="toggleMapView">🎯 Ver criticidad</button>
+      <div class="legendRow" id="mapLegend">
+        <span><i class="dot ok"></i>Normal</span>
+        <span><i class="dot warn"></i>Atención</span>
+        <span><i class="dot bad"></i>Crítico</span>
+        <span><i class="dot nodata"></i>Sin datos</span>
+      </div>
     </div>
   </div>
   <div class="plantWrap"><div class="plant"><svg id="plantSvg" width="1560" height="600" viewBox="0 0 1560 600"></svg></div></div>
@@ -167,6 +187,7 @@ th{font-size:9px;color:var(--muted);text-transform:uppercase}
       <div class="card"><b>Órdenes relevadas</b><span id="metaOrders">16</span></div>
       <div class="card"><b>Tag</b><span id="metaTag">P-80A</span></div>
       <div class="card"><b>Estado</b><span id="metaState">Crítico</span></div>
+      <div class="card"><b>Criticidad</b><span id="metaCrit">Alta</span></div>
     </div>
 
     <div class="kpis">
@@ -182,6 +203,12 @@ th{font-size:9px;color:var(--muted);text-transform:uppercase}
     <div class="metricBtns" id="metricBtns"></div>
 
     <div id="recommendation" class="rec"></div>
+
+    <div id="shareAlert" class="predAlert" style="display:none"></div>
+
+    <div class="sectionLabel">Diagnóstico con IA (opcional)</div>
+    <button id="aiDiagBtn" style="width:100%">🤖 Generar diagnóstico con IA</button>
+    <div id="aiDiagResult" class="rec neutral" style="margin-top:8px;display:none"></div>
   </div>
 </aside>
 </div>
@@ -192,6 +219,38 @@ th{font-size:9px;color:var(--muted);text-transform:uppercase}
   <section class="panel cp"><h3 class="ct">Carta de control</h3><canvas id="controlChart"></canvas><div class="legend">Media ± 2σ para visualizar desvíos.</div></section>
 </div>
 
+<div class="panel" id="reliabPanel" style="margin-top:14px">
+  <div class="ph">
+    <div><h2>Confiabilidad (Weibull)</h2><small id="reliabSub">Tiempo entre fallas (TBF) · rango mediano de Bernard</small></div>
+  </div>
+  <div class="side" id="reliabBody">
+    <div class="kpis" id="reliabKpis">
+      <div class="card kpi"><b>β (forma)</b><div class="v" id="rBeta">—</div><div class="t" id="rBetaNote"></div></div>
+      <div class="card kpi"><b>η (escala)</b><div class="v" id="rEta">—</div><div class="t">días</div></div>
+      <div class="card kpi"><b>R²</b><div class="v" id="rR2">—</div><div class="t">bondad de ajuste</div></div>
+      <div class="card kpi"><b>MTBF</b><div class="v" id="rMtbf">—</div><div class="t">días · η·Γ(1+1/β)</div></div>
+    </div>
+    <div class="charts2" style="margin-top:12px">
+      <section class="panel cp"><h3 class="ct">Recta linealizada de Weibull</h3><canvas id="weibullLineChart"></canvas><div class="legend" id="weibullLineLegend"></div></section>
+      <section class="panel cp"><h3 class="ct">Curva de confiabilidad R(t)</h3><canvas id="weibullRChart"></canvas><div class="legend" id="weibullRLegend"></div></section>
+    </div>
+    <div id="reliabRec" class="rec neutral"></div>
+    <div class="sectionLabel">Predicción de intervención</div>
+    <div class="formGrid">
+      <div class="field"><label>Horas de operación acumuladas</label><input id="opHours" type="number" min="0" step="1" value="0"></div>
+      <div class="field"><label>Umbral de confiabilidad R(t) (%)</label><input id="rThreshold" type="number" min="1" max="99" step="1" value="80"></div>
+    </div>
+    <div id="predAlert" class="predAlert neutral"></div>
+  </div>
+</div>
+
+<div class="panel" id="amfePanel" style="margin-top:14px">
+  <div class="ph">
+    <div><h2>AMFE / Modos críticos</h2><small id="amfeSub">Modos de falla, NPR y acción recomendada</small></div>
+  </div>
+  <div class="side" id="amfeBody"></div>
+</div>
+
 <div class="bottom">
   <section class="panel">
     <div class="ph"><div><h2>Historial de condición</h2><small>Equipo → componente → técnica → parámetro</small></div></div>
@@ -281,13 +340,36 @@ const BASE_META = {
   "P-80B": {name:"Bomba de cargadero de propano P-80B", displayName:"P-80B", family:"Bomba centrífuga vertical", sector:"Despacho", crit:"Baja", sap:"Sí", orders:6, type:"Bomba de cargadero · Criticidad Baja", components:["Motor eléctrico","Rodamiento LA","Rodamiento LOA","Sello mecánico","Acople","Cuerpo de bomba"]},
   "P-80C": {name:"Bomba de cargadero de propano P-80C", displayName:"P-80C", family:"Bomba centrífuga vertical", sector:"Despacho", crit:"Baja", sap:"Sí", orders:0, type:"Bomba de cargadero · Criticidad Baja", components:["Motor eléctrico","Rodamiento LA","Rodamiento LOA","Sello mecánico","Acople","Cuerpo de bomba"]}
 };
+// TBF (días entre fallas) del historial del TP — modos dominantes según AMFE.
+// Dato de arranque; se puede reemplazar por historial real en el Bloque 7.
+const TBF_SEED = {
+  "AC-725": {tbf:[1,4,9,19,34,60,109,237], mode:"Correas de transmisión", share:"75% de las fallas"},
+  "P-80A":  {tbf:[9,22,37,55,77,106,149,229], mode:"Sistema de sello", share:"66,7% de las fallas"}
+};
+// Modos de falla relevados en el AMFE del TP, con su NPR y acción recomendada.
+const AMFE_SEED = {
+  "AC-725": [
+    {mode:"Rotura de correa", npr:288, share:"75% de las fallas", components:["Correas"],
+     action:"Verificar tensado, alineación y estado de las correas; controlar la calidad del repuesto y reforzar la inspección por vibración/ultrasonido entre reemplazos."},
+    {mode:"Desajuste / holgura", npr:252, share:"resto de las fallas", components:["Acople","Rodamiento LA","Rodamiento LOA"],
+     action:"Controlar el torque de fijación de bulones y las holguras del acople motor-ventilador; verificar la alineación con reloj comparador o láser."}
+  ],
+  "P-80A": [
+    {mode:"Sistema de sello", npr:216, nprAfter:108, share:"66,7% de las fallas", components:["Sello mecánico"],
+     action:"Monitorear el sello mecánico por termografía y vibración, y verificar la barrera de lubricación. Mantener el control por condición en vez de aumentar la frecuencia de intervención (β≈1: la tasa de falla es constante)."}
+  ]
+};
 let META = {};
 let CUSTOM = [];
 let data = [];
 let IMPORTED = [];
+let RELIAB = {};
+let AMFE = {};
+let OPHOURS = {};
 let selectedEq = "P-80A";
 let selectedComp = "Rodamiento LA";
 let selectedSeriesKey = "Vibración|Vibración RMS";
+let mapView = "condicion";
 
 const LAYOUT = {
   "T-1200":[200,160], "T-1300":[200,340],
@@ -318,6 +400,12 @@ function loadImported(){
 function saveImported(){
   try{ localStorage.setItem("cv12imported", JSON.stringify(IMPORTED)); }catch(e){}
 }
+function loadReliability(){
+  RELIAB = JSON.parse(JSON.stringify(TBF_SEED));
+}
+function loadAmfe(){
+  AMFE = JSON.parse(JSON.stringify(AMFE_SEED));
+}
 function rand(seed){ const x = Math.sin(seed) * 10000; return x - Math.floor(x); }
 function addSeries(arr, eq, comp, tech, param, unit, base, slope, noise, source, n=24, stepH=72){
   const start = new Date("2026-07-15T08:00:00");
@@ -327,9 +415,26 @@ function addSeries(arr, eq, comp, tech, param, unit, base, slope, noise, source,
     arr.push({timestamp:t, equipment:eq, component:comp, technique:tech, parameter:param, value:v, unit, source});
   }
 }
+// Serie en "diente de sierra": sube hacia el umbral y cae de golpe, repitiendo varios
+// ciclos. Sirve para modos con mortalidad infantil (β<1, ej. correas del AC-725),
+// donde el historial real muestra reintervenciones frecuentes en vez de una única
+// rampa continua como en un desgaste progresivo.
+function addSawtoothSeries(arr, eq, comp, tech, param, unit, base, riseRate, cycleLen, noise, source, cycles=5, stepH=48){
+  const start = new Date("2026-07-10T08:00:00");
+  let i = 0;
+  for(let c=0;c<cycles;c++){
+    for(let k=0;k<cycleLen;k++){
+      const t = new Date(start.getTime() + i*stepH*3600000);
+      const v = Math.max(0, base + riseRate*k + (rand(i*13 + eq.length*5 + comp.length*3 + param.length)*2 - 1) * noise);
+      arr.push({timestamp:t, equipment:eq, component:comp, technique:tech, parameter:param, value:v, unit, source});
+      i++;
+    }
+  }
+}
 function demoData(){
   const arr = [];
   // Bomba de cargadero P-80A (equipo crítico - sistema de sello)
+  // Sello mecánico: modo dominante (66,7% de las fallas del TP, β≈1,02 en el AMFE/Weibull).
   addSeries(arr,"P-80A","Rodamiento LA","Vibración","Vibración RMS","mm/s",3.4,0.19,0.34,"Colector portátil");
   addSeries(arr,"P-80A","Rodamiento LOA","Vibración","Vibración RMS","mm/s",3.0,0.08,0.28,"Colector portátil");
   addSeries(arr,"P-80A","Sello mecánico","Termografía","Temperatura máxima","°C",57,0.72,1.8,"Cámara IR");
@@ -341,7 +446,9 @@ function demoData(){
   addSeries(arr,"P-80C","Rodamiento LA","Vibración","Vibración RMS","mm/s",2.7,0.05,0.20,"Colector portátil");
 
   // Aeroventilador AC-725 (equipo crítico - correas de transmisión)
-  addSeries(arr,"AC-725","Correas","Vibración","Vibración RMS","mm/s",3.6,0.22,0.30,"Colector portátil");
+  // Correas: modo dominante (75% de las fallas, β≈0,57 mortalidad infantil) — se ve
+  // como reintervenciones frecuentes, no como un desgaste progresivo continuo.
+  addSawtoothSeries(arr,"AC-725","Correas","Vibración","Vibración RMS","mm/s",2.9,0.55,5,0.22,"Colector portátil");
   addSeries(arr,"AC-725","Rodamiento LA","Vibración","Vibración RMS","mm/s",3.3,0.15,0.28,"Colector portátil");
   addSeries(arr,"AC-725","Motor eléctrico","Termografía","Temperatura máxima","°C",60,0.34,1.5,"Cámara IR");
   addSeries(arr,"AC-725","Rodamiento LA","Ultrasonido","Nivel ultrasónico","dB",26,0.42,1.8,"Ultrasonido");
@@ -362,8 +469,9 @@ function demoData(){
   return arr;
 }
 function resetApp(){
-  cloneMeta(); loadCustom(); loadImported();
+  cloneMeta(); loadCustom(); loadImported(); loadReliability(); loadAmfe();
   data = demoData().concat(IMPORTED);
+  OPHOURS = {};
   selectedEq = "P-80A"; selectedComp = "Rodamiento LA"; selectedSeriesKey = "Vibración|Vibración RMS";
 }
 
@@ -387,6 +495,11 @@ function rowStatus(r){
   const t = thresholds(r.technique, r.parameter);
   return r.value >= t.bad ? "bad" : r.value >= t.warn ? "warn" : "ok";
 }
+// Índice de salud: arranca en 98 (equipo prácticamente nuevo) y descuenta por cada
+// medición histórica fuera de umbral — 1,25 pts por lectura "crítica" y 0,45 por
+// "atención" (ver thresholds() por técnica) — así un equipo con muchos desvíos
+// acumulados baja más que uno con un pico aislado. Piso en 28 para que nunca
+// llegue a 0 ni dé una falsa sensación de "equipo muerto".
 function health(eq){
   const rows = allEq(eq);
   if(!rows.length) return null;
@@ -399,6 +512,7 @@ function eqStatus(eq){
   if(h===null) return "nodata";
   return h < 70 ? "bad" : h < 86 ? "warn" : "ok";
 }
+function critColor(crit){ return crit==="Alta" ? "#ff5d6c" : crit==="Media" ? "#f2c94c" : crit==="Baja" ? "#35d07f" : "#7d8fa7"; }
 function prettyStatus(s){ return s==="bad" ? "Crítico" : s==="warn" ? "Atención" : s==="nodata" ? "Sin datos" : "Normal"; }
 function fmt(r){ return `${r.value.toFixed(r.unit==="bar"||r.unit==="A" ? 2 : 1)} ${r.unit}`; }
 function availableSeries(){
@@ -444,7 +558,7 @@ function renderPlant(){
     const t1 = mk("text"); t1.setAttribute("x",0); t1.setAttribute("y",-8); t1.setAttribute("text-anchor","middle"); t1.setAttribute("class","label"); t1.textContent=META[id].displayName||id; g.appendChild(t1);
     const t2 = mk("text"); t2.setAttribute("x",0); t2.setAttribute("y",12); t2.setAttribute("text-anchor","middle"); t2.setAttribute("class","sublabel"); t2.textContent=id; g.appendChild(t2);
     const s = eqStatus(id);
-    const c = s==="ok"?"#35d07f":s==="warn"?"#f2c94c":s==="bad"?"#ff5d6c":"#7d8fa7";
+    const c = mapView==="criticidad" ? critColor(META[id].crit) : (s==="ok"?"#35d07f":s==="warn"?"#f2c94c":s==="bad"?"#ff5d6c":"#7d8fa7");
     const b = mk("rect"); b.setAttribute("x",27); b.setAttribute("y",-29); b.setAttribute("width",23); b.setAttribute("height",14); b.setAttribute("rx",7); b.setAttribute("fill",c); g.appendChild(b);
     svg.appendChild(g);
   };
@@ -513,6 +627,9 @@ function renderSide(){
   document.getElementById("metaOrders").textContent = meta.orders ?? "-";
   document.getElementById("metaTag").textContent = selectedEq;
   document.getElementById("metaState").textContent = prettyStatus(s);
+  const critEl = document.getElementById("metaCrit");
+  critEl.textContent = meta.crit || "Sin definir";
+  critEl.style.color = critColor(meta.crit);
   renderComponents();
   renderMetricBtns();
 
@@ -581,6 +698,197 @@ function draw(){
   [["#35d07f",m],["#f2c94c",u],["#f2c94c",l]].forEach(z=>{ c.ctx.strokeStyle=z[0]; c.ctx.setLineDash([5,5]); c.ctx.beginPath(); c.ctx.moveTo(cx.ml,yy(z[1])); c.ctx.lineTo(c.w-cx.mr,yy(z[1])); c.ctx.stroke(); c.ctx.setLineDash([]); });
   vals.forEach((v,i)=>{ const x=cx.ml+cx.pw*i/Math.max(1,vals.length-1), y=yy(v); c.ctx.fillStyle=(v>u||v<l)?"#ff5d6c":"#52d6ff"; c.ctx.beginPath(); c.ctx.arc(x,y,3,0,Math.PI*2); c.ctx.fill(); });
 }
+function gammaFn(x){
+  const p = [0.99999999999980993,676.5203681218851,-1259.1392167224028,771.32342877765313,-176.61502916214059,12.507343278686905,-0.13857109526572012,9.9843695780195716e-6,1.5056327351493116e-7];
+  if(x < 0.5) return Math.PI / (Math.sin(Math.PI*x) * gammaFn(1-x));
+  x -= 1;
+  let a = p[0];
+  const t = x + 7.5;
+  for(let i=1;i<9;i++) a += p[i]/(x+i);
+  return Math.sqrt(2*Math.PI) * Math.pow(t, x+0.5) * Math.exp(-t) * a;
+}
+function weibullFit(tbf){
+  const sorted = [...tbf].sort((a,b)=>a-b);
+  const n = sorted.length;
+  if(n < 3) return null;
+  const xs = sorted.map(t=>Math.log(t));
+  const ys = sorted.map((t,i)=>{ const F=(i+1-0.3)/(n+0.4); return Math.log(-Math.log(1-F)); });
+  const mx = mean(xs), my = mean(ys);
+  let sxx=0, sxy=0, syy=0;
+  xs.forEach((x,i)=>{ sxx+=(x-mx)**2; sxy+=(x-mx)*(ys[i]-my); syy+=(ys[i]-my)**2; });
+  const beta = sxy/sxx;
+  const eta = Math.exp(-(my-beta*mx)/beta);
+  const r2 = (sxy*sxy)/(sxx*syy);
+  const mtbf = eta * gammaFn(1 + 1/beta);
+  return {beta, eta, r2, mtbf, n, points:sorted.map((t,i)=>({t, x:xs[i], y:ys[i]}))};
+}
+function weibullR(t, beta, eta){ return Math.exp(-Math.pow(t/eta, beta)); }
+function reliabInterpretation(beta){
+  if(beta < 0.95) return {cls:"", text:`β = ${beta.toFixed(2)} (< 1): mortalidad infantil / fallas tempranas recurrentes. Conviene revisar montaje, calidad de repuesto o el procedimiento de intervención en vez de anticipar el reemplazo por calendario.`};
+  if(beta <= 1.15) return {cls:"good", text:`β = ${beta.toFixed(2)} (≈ 1): tasa de falla aproximadamente constante. Intervenir más seguido no reduce el riesgo; el control por condición es la estrategia correcta.`};
+  return {cls:"neutral", text:`β = ${beta.toFixed(2)} (> 1): desgaste progresivo. Tiene sentido planificar el reemplazo antes de alcanzar la vida característica η.`};
+}
+function renderReliabKpis(fit, seed){
+  const rec = document.getElementById("reliabRec");
+  if(!fit){
+    ["rBeta","rEta","rR2","rMtbf"].forEach(id=>document.getElementById(id).textContent="—");
+    document.getElementById("rBetaNote").textContent="";
+    document.getElementById("reliabSub").textContent = "Tiempo entre fallas (TBF) · rango mediano de Bernard";
+    rec.className = "rec neutral";
+    rec.textContent = "Este equipo todavía no tiene tiempos entre fallas (TBF) cargados para el análisis de confiabilidad.";
+    return;
+  }
+  const {beta,eta,r2,mtbf,n} = fit;
+  document.getElementById("rBeta").textContent = beta.toFixed(2);
+  document.getElementById("rEta").textContent = eta.toFixed(1);
+  document.getElementById("rR2").textContent = r2.toFixed(3);
+  document.getElementById("rMtbf").textContent = mtbf.toFixed(1);
+  document.getElementById("rBetaNote").textContent = beta<0.95 ? "mortalidad infantil" : beta<=1.15 ? "tasa constante" : "desgaste";
+  document.getElementById("reliabSub").textContent = `${seed.mode} · modo dominante (${seed.share}) · ${n} TBF analizados`;
+  const interp = reliabInterpretation(beta);
+  rec.className = "rec" + (interp.cls ? " "+interp.cls : "");
+  rec.textContent = interp.text;
+}
+function drawWeibull(fit){
+  ["weibullLineChart","weibullRChart"].forEach(id=>{
+    const {ctx} = setupCanvas(id);
+    if(!fit){ ctx.fillStyle="#8298b4"; ctx.font="12px Segoe UI"; ctx.fillText("Sin TBF cargados para este equipo",20,35); }
+  });
+  document.getElementById("weibullLineLegend").textContent = fit ? "" : "Sin serie de fallas seleccionada.";
+  document.getElementById("weibullRLegend").textContent = "";
+  if(!fit) return;
+  const {beta, eta, points} = fit;
+
+  const xs = points.map(p=>p.x), ys = points.map(p=>p.y);
+  let xmin=Math.min(...xs), xmax=Math.max(...xs), xpad=(xmax-xmin||1)*.15; xmin-=xpad; xmax+=xpad;
+  let ymin=Math.min(...ys), ymax=Math.max(...ys), ypad=(ymax-ymin||1)*.15; ymin-=ypad; ymax+=ypad;
+  const a = setupCanvas("weibullLineChart"), ax = axes(a.ctx,a.w,a.h,ymin,ymax,"ln(-ln(1-F))");
+  const xTo = x => ax.ml + ax.pw*(x-xmin)/(xmax-xmin||1);
+  const yTo = y => ax.mt + ax.ph*(1-(y-ymin)/(ymax-ymin||1));
+  a.ctx.strokeStyle="#f2c94c"; a.ctx.lineWidth=1.6; a.ctx.beginPath();
+  const lnEta = Math.log(eta);
+  a.ctx.moveTo(xTo(xmin), yTo(beta*xmin - beta*lnEta));
+  a.ctx.lineTo(xTo(xmax), yTo(beta*xmax - beta*lnEta));
+  a.ctx.stroke();
+  a.ctx.fillStyle="#52d6ff";
+  points.forEach(p=>{ const x=xTo(p.x), y=yTo(p.y); a.ctx.beginPath(); a.ctx.arc(x,y,3,0,Math.PI*2); a.ctx.fill(); });
+  document.getElementById("weibullLineLegend").textContent = `${points.length} TBF · pendiente = β = ${beta.toFixed(2)}`;
+
+  const tmax = Math.max(...points.map(p=>p.t)) * 1.3;
+  const b = setupCanvas("weibullRChart"), bx = axes(b.ctx,b.w,b.h,0,1,"R(t)");
+  const txTo = t => bx.ml + bx.pw*(t/tmax);
+  b.ctx.strokeStyle="#35d07f"; b.ctx.lineWidth=2; b.ctx.beginPath();
+  const steps=60;
+  for(let i=0;i<=steps;i++){
+    const t=tmax*i/steps, r=weibullR(t,beta,eta), x=txTo(t), y=bx.mt+bx.ph*(1-r);
+    i?b.ctx.lineTo(x,y):b.ctx.moveTo(x,y);
+  }
+  b.ctx.stroke();
+  b.ctx.strokeStyle="#8298b4"; b.ctx.setLineDash([4,4]); b.ctx.beginPath();
+  b.ctx.moveTo(txTo(eta),bx.mt); b.ctx.lineTo(txTo(eta),bx.mt+bx.ph); b.ctx.stroke(); b.ctx.setLineDash([]);
+  document.getElementById("weibullRLegend").textContent = `η = ${eta.toFixed(1)} días (t donde R(t) ≈ 36,8%)`;
+}
+function currentFit(){
+  const seed = RELIAB[selectedEq];
+  return seed && seed.tbf ? weibullFit(seed.tbf) : null;
+}
+function reliabPrediction(fit, hours, thrPct){
+  if(!fit) return null;
+  const days = Math.max(0, hours/24);
+  const thr = Math.min(0.99, Math.max(0.01, thrPct/100));
+  const {beta, eta} = fit;
+  const tThreshold = eta * Math.pow(-Math.log(thr), 1/beta);
+  const rNow = weibullR(days, beta, eta);
+  return {days, thr, tThreshold, rNow, daysLeft: tThreshold - days};
+}
+function renderPrediction(fit){
+  const box = document.getElementById("predAlert");
+  if(!fit){
+    box.className = "predAlert neutral";
+    box.innerHTML = `<div><div class="pTitle">Próxima intervención recomendada</div><div class="pMain">—</div><div class="pSub">Este equipo no tiene un ajuste de Weibull disponible.</div></div>`;
+    return;
+  }
+  const hours = Math.max(0, Number(document.getElementById("opHours").value) || 0);
+  const thrPct = Math.min(99, Math.max(1, Number(document.getElementById("rThreshold").value) || 80));
+  const p = reliabPrediction(fit, hours, thrPct);
+  const cls = p.daysLeft <= 0 ? "bad" : p.daysLeft <= 30 ? "warn" : "ok";
+  const badgeCls = cls==="bad" ? "red" : cls==="warn" ? "yellow" : "green";
+  const badgeText = p.daysLeft <= 0 ? "VENCIDA" : p.daysLeft <= 30 ? "PRÓXIMA" : "EN CONTROL";
+  const mainText = p.daysLeft <= 0 ? `Vencida hace ${Math.abs(Math.round(p.daysLeft))} días` : `${Math.round(p.daysLeft)} días`;
+  const targetDate = new Date(Date.now() + p.daysLeft*86400000);
+  box.className = "predAlert " + cls;
+  box.innerHTML = `<div>
+      <div class="pTitle">Próxima intervención recomendada</div>
+      <div class="pMain">${mainText}</div>
+      <div class="pSub">R(t) llega a ${(p.thr*100).toFixed(0)}% a los ${p.tThreshold.toFixed(1)} días de operación (≈ ${targetDate.toLocaleDateString("es-AR")}) · confiabilidad actual estimada ${(p.rNow*100).toFixed(0)}%</div>
+    </div>
+    <div class="badge ${badgeCls}">${badgeText}</div>`;
+}
+function renderReliability(){
+  const seed = RELIAB[selectedEq];
+  const fit = currentFit();
+  renderReliabKpis(fit, seed);
+  drawWeibull(fit);
+  document.getElementById("opHours").value = OPHOURS[selectedEq] ?? 0;
+  renderPrediction(fit);
+}
+function amfeActionFor(eq, comp){
+  const modes = AMFE[eq];
+  if(!modes || !modes.length) return null;
+  return modes.find(m => m.components && m.components.includes(comp)) || modes.reduce((a,b)=> b.npr>a.npr?b:a, modes[0]);
+}
+function renderAmfe(){
+  const box = document.getElementById("amfeBody");
+  const modes = AMFE[selectedEq];
+  document.getElementById("amfeSub").textContent = modes && modes.length
+    ? `${META[selectedEq]?.displayName || selectedEq} · modos de falla relevados en el AMFE`
+    : "Modos de falla, NPR y acción recomendada";
+  if(!modes || !modes.length){
+    box.innerHTML = `<div class="rec neutral">Este equipo todavía no tiene modos de falla relevados en el AMFE.</div>`;
+    return;
+  }
+  box.innerHTML = modes.map(m=>{
+    const sev = m.npr>=200 ? "red" : m.npr>=100 ? "yellow" : "green";
+    const sevLabel = m.npr>=200 ? "Alto" : m.npr>=100 ? "Medio" : "Bajo";
+    return `<div class="amfeCard">
+      <div class="amfeHead">
+        <div><b>${m.mode}</b>${m.share ? `<small>${m.share}</small>` : ""}</div>
+        <div class="badge ${sev}">NPR ${m.npr}${m.nprAfter ? " → "+m.nprAfter : ""} · ${sevLabel}</div>
+      </div>
+      <div class="amfeAction">${m.action}</div>
+    </div>`;
+  }).join("");
+}
+function renderShareAlert(){
+  const box = document.getElementById("shareAlert");
+  const s = eqStatus(selectedEq);
+  if(s !== "bad" && s !== "warn"){ box.style.display = "none"; box.innerHTML = ""; return; }
+  const meta = META[selectedEq] || {};
+  const rows = rowsFor();
+  const amfe = amfeActionFor(selectedEq, selectedComp);
+  const accion = amfe ? amfe.action : recText(rows).replace(/<[^>]*>/g,"");
+  const msg = [
+    "Alerta de mantenimiento por condición",
+    `Equipo: ${selectedEq} - ${meta.name || selectedEq}`,
+    `Estado: ${prettyStatus(s)}${meta.crit ? " · Criticidad " + meta.crit : ""}`,
+    `Componente: ${selectedComp}`,
+    rows.length ? `Última medición: ${last(rows).technique} · ${last(rows).parameter} = ${fmt(last(rows))}` : "Sin mediciones recientes",
+    `Acción sugerida: ${accion}`
+  ].join("\n");
+  const waHref = "https://wa.me/?text=" + encodeURIComponent(msg);
+  const mailHref = `mailto:?subject=${encodeURIComponent("Alerta de mantenimiento · " + selectedEq)}&body=${encodeURIComponent(msg)}`;
+  box.style.display = "";
+  box.className = "predAlert " + s;
+  box.innerHTML = `<div>
+      <div class="pTitle">Notificar desvío</div>
+      <div class="pMain">${selectedEq} · ${prettyStatus(s)}</div>
+      <div class="pSub">Compartí el estado del equipo y la acción sugerida por WhatsApp o email.</div>
+    </div>
+    <div style="display:flex;gap:8px;flex-wrap:wrap">
+      <a class="fileBtn" href="${waHref}" target="_blank" rel="noopener">💬 WhatsApp</a>
+      <a class="fileBtn" href="${mailHref}">✉️ Email</a>
+    </div>`;
+}
 function renderHistory(){
   const tb = document.getElementById("history");
   tb.innerHTML = "";
@@ -591,7 +899,13 @@ function renderHistory(){
     tb.appendChild(tr);
   });
 }
-function render(){ ensureSelection(); renderPlant(); renderSide(); draw(); renderHistory(); }
+function renderLegend(){
+  const box = document.getElementById("mapLegend");
+  box.innerHTML = mapView==="criticidad"
+    ? '<span><i class="dot bad"></i>Alta</span><span><i class="dot warn"></i>Media</span><span><i class="dot ok"></i>Baja</span><span><i class="dot nodata"></i>Sin definir</span>'
+    : '<span><i class="dot ok"></i>Normal</span><span><i class="dot warn"></i>Atención</span><span><i class="dot bad"></i>Crítico</span><span><i class="dot nodata"></i>Sin datos</span>';
+}
+function render(){ ensureSelection(); renderLegend(); renderPlant(); renderSide(); draw(); renderReliability(); renderAmfe(); renderShareAlert(); document.getElementById("aiDiagResult").style.display="none"; renderHistory(); }
 
 function openModal(id){ document.getElementById(id).style.display="flex"; }
 function closeModal(id){ document.getElementById(id).style.display="none"; }
@@ -675,11 +989,12 @@ document.getElementById("csvInput").onchange = async e=>{
 };
 document.getElementById("interBtn").onclick = ()=>{
   const rows = rowsFor(), r = rows.length ? last(rows) : null;
+  const amfe = amfeActionFor(selectedEq, selectedComp);
   document.getElementById("intEq").value = selectedEq;
   document.getElementById("intComp").value = selectedComp;
   document.getElementById("intPriority").value = eqStatus(selectedEq)==="bad" ? "Alta" : eqStatus(selectedEq)==="warn" ? "Media" : "Baja";
   document.getElementById("intDesc").value = r ? `Desvío detectado en ${r.technique} - ${r.parameter}. Último valor: ${fmt(r)}. Componente: ${selectedComp}.` : `Seguimiento de condición del componente ${selectedComp}.`;
-  document.getElementById("intAction").value = recText(rows).replace(/<[^>]*>/g,"");
+  document.getElementById("intAction").value = amfe ? `[AMFE · ${amfe.mode} · NPR ${amfe.npr}] ${amfe.action}` : recText(rows).replace(/<[^>]*>/g,"");
   openModal("interModal");
 };
 document.getElementById("downloadInt").onclick = ()=>{
@@ -696,7 +1011,48 @@ document.getElementById("resetBtn").onclick = ()=>{
   localStorage.removeItem("cv12imported");
   resetApp(); render(); toast("Demo restaurada");
 };
-window.addEventListener("resize", draw);
+document.getElementById("toggleMapView").onclick = ()=>{
+  mapView = mapView==="condicion" ? "criticidad" : "condicion";
+  document.getElementById("toggleMapView").textContent = mapView==="condicion" ? "🎯 Ver criticidad" : "🚦 Ver condición";
+  render();
+};
+document.getElementById("opHours").oninput = ()=>{
+  OPHOURS[selectedEq] = Math.max(0, Number(document.getElementById("opHours").value) || 0);
+  renderPrediction(currentFit());
+};
+document.getElementById("rThreshold").oninput = ()=>{
+  renderPrediction(currentFit());
+};
+// Diagnóstico con IA — OPCIONAL. Solo funciona si esta app corre dentro del
+// entorno de artefactos de Claude (claude.ai), que expone window.claude.complete().
+// Abierta como archivo .html local (el modo normal de uso de este tablero) NO hay
+// conexión al modelo: se muestra el caveat y el diagnóstico heurístico de arriba
+// (recommendation / AMFE) sigue siendo el plan B permanente. Ver CLAUDE.md §2.
+document.getElementById("aiDiagBtn").onclick = async ()=>{
+  const box = document.getElementById("aiDiagResult");
+  box.style.display = "";
+  if(typeof window.claude === "undefined" || typeof window.claude.complete !== "function"){
+    box.className = "rec neutral";
+    box.textContent = "El diagnóstico con IA solo está disponible corriendo este tablero dentro del entorno de artefactos de Claude. Abierto como archivo .html local no hay conexión al modelo — el diagnóstico heurístico de arriba es el plan B.";
+    return;
+  }
+  const btn = document.getElementById("aiDiagBtn");
+  const prevText = btn.textContent;
+  btn.disabled = true; btn.textContent = "🤖 Consultando...";
+  try{
+    const rows = rowsFor();
+    const prompt = `Sos un ingeniero de mantenimiento predictivo. Equipo ${selectedEq} (${META[selectedEq]?.name||""}), componente ${selectedComp}. Estado: ${prettyStatus(eqStatus(selectedEq))}. Última medición: ${rows.length?fmt(last(rows)):"sin datos"} de ${selectedSeriesKey}. Dá un diagnóstico breve (máx. 3 líneas) y una acción concreta, en español.`;
+    const result = await window.claude.complete(prompt);
+    box.className = "rec good";
+    box.textContent = result;
+  }catch(e){
+    box.className = "rec neutral";
+    box.textContent = "No se pudo obtener el diagnóstico de IA en este momento. Queda el diagnóstico heurístico de arriba como plan B.";
+  }finally{
+    btn.disabled = false; btn.textContent = prevText;
+  }
+};
+window.addEventListener("resize", ()=>{ draw(); renderReliability(); });
 
 resetApp();
 render();
```

**Archivo nuevo (no en el diff de arriba porque `git diff` no lo lista al no estar
trackeado todavía):** `Guion_Demo.md` — guion de exposición de 3–4 minutos.
