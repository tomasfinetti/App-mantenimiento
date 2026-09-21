# CLAUDE.md — Tablero de Mantenimiento (Grupo 06)

Este archivo lo lee Claude Code automáticamente al abrir la carpeta. Contiene el
**contexto y las reglas permanentes** del proyecto sobre la app
`Tablero_Mantenimiento_v13_G06.html`. Respetalas en todo lo que desarrolles.

Las órdenes puntuales (qué bloque hacer, cuándo guardar y subir a GitHub) las da el
usuario por chat; no van en este archivo.

---

## 1. Qué es la app

Tablero de **monitoreo por condición** que acompaña el TP de Mantenimiento (planta
YPF GLP-ELP, sectores **Fraccionamiento** y **Despacho**). Es la materialización de la
propuesta central del informe: migrar de mantenimiento por calendario a mantenimiento
por condición. Muestra un mapa de planta, salud por equipo, gráficos de tendencia /
histograma / carta de control, historial e importación/exportación a SAP PM.

**Equipos críticos del TP** (mantener SIEMPRE la coherencia con el informe):
- **AC-725** — aeroventilador de propano. Modo de falla dominante: **correas de
  transmisión** (75 % de sus fallas). Weibull β ≈ 0,57 (mortalidad infantil /
  reintervenciones). NPR AMFE: rotura de correa **288**, desajuste **252**.
- **P-80A** — bomba de cargadero. Modo dominante: **sistema de sello** (66,7 %).
  Weibull β ≈ 1,02 (tasa de fallas constante) → la acción correcta es control por
  condición, NO intervenir más seguido. η ≈ 94 días. NPR AMFE: **216 → 108**.

## 2. Restricciones técnicas (no romper)

- **Un solo archivo `.html` autocontenido.** Sin librerías externas: los gráficos se
  dibujan a mano sobre `<canvas>`. NO agregar Chart.js ni ningún CDN — así funciona
  offline y para exponer desde una laptop.
- **Coherencia con el TP.** Los equipos, sectores y niveles de criticidad tienen que
  coincidir con el informe (R02). No inventar equipos.
- **Persistencia:** `localStorage` (`cv12custom` = máquinas agregadas,
  `cv12imported` = mediciones importadas). Todo por-navegador.
- **Función de IA (Bloque 6.2):** una llamada a la API de Claude SOLO funciona
  corriendo la app dentro del entorno de artefactos de Claude; abriendo el `.html`
  localmente NO funciona. Dejar siempre el diagnóstico heurístico como plan B.

## 3. Estado actual — Bloque 1 HECHO ✅

Ya aplicado en la v13:
- Equipos alineados al TP: torres **T-1200/T-1300**, acumuladores **V-270/V-280**,
  intercambiadores **E-505/E-515**, aeroenfriadores **AC-725/720/745**, bombas
  **P-80A/B/C**. Eliminados los equipos fuera de alcance (T-731/732/733, TK-60x,
  HE-301, DEH-302, LD-701, sector Acondicionamiento).
- Criticidad por niveles de la matriz (**Alta / Media / Baja**), con campo `crit` en
  cada equipo de `BASE_META`.
- Bug de persistencia corregido: las mediciones importadas ahora persisten.
- Parser CSV endurecido (maneja comillas) vía `parseCsvLine()`.

## 4. Roadmap por bloques (lo que falta)

**Bloque 2 — Matriz de criticidad** *(conecta el punto 4 del TP)*
- Cargar los equipos con sus 6 puntajes de la matriz → nivel automático.
- Colorear los nodos del mapa por criticidad (alternable con el semáforo de condición).
- Mostrar puntaje y nivel al seleccionar un equipo.

**Bloque 3 — Weibull embebido** *(el diferenciador)*
- Panel/pestaña "Confiabilidad" por equipo.
- Cargar los TBF del AC-725 y la P-80A (del historial del TP) o importarlos.
- Calcular β, η, R² y MTBF (rango mediano de Bernard + mínimos cuadrados).
- Dibujar la recta linealizada y la curva R(t) con el motor de canvas existente
  (reutilizar el patrón de `draw()` / los canvas de tendencia).
- Interpretación automática según β.

**Bloque 4 — Predicción de intervención** *(la función más vendedora)*
- Ingresar/estimar horas de operación acumuladas.
- Calcular el t al que R(t) cae por debajo de un umbral (ej. 80 %) con η y β.
- Mostrar "días hasta la próxima intervención" como alerta destacada.

**Bloque 5 — AMFE/NPR + orden de trabajo**
- Panel con los modos críticos y sus NPR (288/252/216) + acciones del AMFE.
- Auto-completar el generador de intervención (ya existe: botón `interBtn` /
  `downloadInt`) con la acción del AMFE del equipo seleccionado.

**Bloque 6 — Diferenciadores extra**
- Alerta WhatsApp/mail: link `wa.me/?text=…` o `mailto:` al cruzar umbral (standalone).
- (Opcional) Diagnóstico con IA — con el caveat del punto 2.

**Bloque 7 — Pulido y demo**
- Sembrar P-80A y AC-725 con datos parecidos al historial real.
- Documentar/ponderar el índice de salud (`health()` en el código).
- Guion de 3–4 min: mapa → P-80A → condición → Weibull → predicción → generar OT.

## 5. Mapa del código (para orientarse rápido)

- `BASE_META` — definición de equipos (id, sector, `crit`, componentes).
- `LAYOUT` — posiciones [x,y] de cada equipo en el mapa SVG.
- `demoData()` / `addSeries()` — datos simulados de mediciones.
- `resetApp()` — arma el estado (mete demo + importados).
- `thresholds()` / `rowStatus()` / `eqStatus()` / `health()` — lógica de semáforo y salud.
- `drawMap()` — dibuja el mapa (labels, pipes, nodos).
- `draw()` — dibuja los 3 canvas (tendencia, histograma, carta de control). **Este es
  el motor a reutilizar para el Weibull.**
- Handler `csvInput.onchange` — importación; `interBtn`/`downloadInt` — generación de OT.

## 6. Control de versiones y trabajo en equipo (git + GitHub)

El proyecto se versiona con **git** (una "máquina del tiempo" del código) y se comparte
con el grupo por **GitHub**. Todo sin usar la terminal.

**Los commits los hace Claude Code**, hablándole en español:
- Al arrancar: *"Iniciá el control de versiones con git en esta carpeta y guardá una
  primera versión."*
- Después de cada bloque: *"Guardá una versión (commit) con estos cambios y subila a
  GitHub (push), con el nombre 'Bloque N listo'."*
- Si algo se rompe: *"Volvé a la última versión guardada."*

**Repositorio compartido (una sola vez, lo hace una persona):**
1. Crear cuenta en github.com e instalar **GitHub Desktop** (app con botones, sin terminal).
2. Publicar la carpeta del proyecto como repositorio **PRIVADO** (es un TP, no va público).
3. Invitar al grupo: Settings → Collaborators, con el usuario GitHub de cada uno.

**Los compañeros:**
- Instalan GitHub Desktop y usan **"Clone"** para bajar el proyecto.
- **Antes** de trabajar: botón **"Pull"** (traer la última versión).
- **Al terminar:** Claude Code hace commit + push, o suben con el botón **"Push"**.

**Reglas para no chocar (es un solo archivo `.html`):**
- Un bloque por persona por vez; nunca dos editando el mismo archivo a la vez.
- Siempre **Pull antes de empezar** y **Push al terminar**.
- Repartir los bloques del checklist por turnos (2, 3, 4…), no en paralelo sobre lo mismo.
- Red de seguridad extra: duplicar el `.html` antes de un bloque grande.
