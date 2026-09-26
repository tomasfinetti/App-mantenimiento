# Guion de demo — Tablero de Mantenimiento (3–4 min)

Guion para exponer el tablero `index.html` en la defensa del TP.
Recorre: mapa → P-80A → condición → Weibull → predicción → generar OT.

## 0:00 – 0:30 · Mapa general
- Abrir el tablero. Mostrar el mapa de planta: Fraccionamiento (torres, acumuladores,
  intercambiadores, aeroenfriadores) y Despacho (bombas de cargadero).
- Tocar el botón "Ver criticidad": los nodos cambian de semáforo de condición a
  criticidad (Alta/Media/Baja), conectando el mapa con la matriz de criticidad del TP.
- Volver a "Ver condición".

## 0:30 – 1:00 · Seleccionar P-80A
- Clic en el nodo P-80A (bomba de cargadero, Despacho).
- Señalar la ficha: salud 61/100, badge CRÍTICO, criticidad Alta, 16 órdenes
  relevadas en SAP.

## 1:00 – 1:40 · Condición
- Mostrar el componente Sello mecánico (modo de falla dominante, 66,7% de las
  fallas) y la técnica Termografía.
- Señalar la tendencia temporal ascendente, el histograma y la carta de control
  (media ± 2σ) — ahí se ve la migración de mantenimiento por calendario a
  mantenimiento por condición.

## 1:40 – 2:40 · Confiabilidad (Weibull)
- Bajar al panel Confiabilidad (Weibull).
- Leer los KPI: β≈1,0 (tasa de falla constante), η≈94 días, R² alto, MTBF.
- Mostrar la recta linealizada y la curva R(t).
- Leer la interpretación automática: con β≈1, intervenir más seguido no reduce el
  riesgo — el control por condición es la estrategia correcta. Es el argumento
  central del informe.

## 2:40 – 3:15 · Predicción de intervención
- Cargar unas horas de operación acumuladas (ej. 500) en el campo correspondiente.
- Mostrar cómo la alerta "Próxima intervención recomendada" recalcula los días
  restantes hasta que R(t) cae del umbral configurable (80% por defecto).

## 3:15 – 3:45 · AMFE y generar orden de trabajo
- Bajar al panel AMFE / Modos críticos: NPR 216 → 108 del sello mecánico y la
  acción recomendada.
- Clic en "Generar intervención": mostrar que "Acción recomendada" ya viene
  autocompletado con la acción del AMFE. Descargar el CSV (formato listo para
  SAP PM / Maximo).

## 3:45 – 4:00 · Cierre
- (Si da el tiempo) Mostrar la alerta de WhatsApp / email que aparece cuando el
  equipo cruza el umbral de condición.
- Cerrar remarcando la propuesta del TP: pasar de mantenimiento por calendario a
  mantenimiento por condición, con AC-725 (correas, β≈0,57, mortalidad infantil) y
  P-80A (sello, β≈1,02, tasa constante) como los dos casos que lo demuestran.

## Por si preguntan
- ¿Por qué β≈1 en P-80A implica no intervenir más seguido? Porque la tasa de falla
  es aproximadamente constante en el tiempo (no hay desgaste progresivo): achicar
  el intervalo de mantenimiento no baja el riesgo, hay que vigilar la condición
  real del sello.
- ¿Por qué el AC-725 tiene β<1? Mortalidad infantil: la mayoría de las roturas de
  correa ocurren poco después de una reintervención (montaje, tensado o repuesto),
  no por desgaste — el AMFE lo confirma con el 75% de las fallas en ese modo.
- ¿De dónde salen los TBF y el AMFE? Son datos de arranque calibrados para
  reproducir los β/η/NPR del informe (ver comentarios en el código); se pueden
  reemplazar por el historial real completo sin tocar el motor de cálculo.
