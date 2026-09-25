# CLAUDE.md — Reglas de trabajo del proyecto

Este archivo lo lee Claude Code automáticamente al abrir la carpeta. Define únicamente
CÓMO se trabaja en este proyecto. El qué construir y el estado actual los da el usuario
por chat, en cada pedido.

## Cómo trabajar

- Mostrá el diff antes de aplicar cualquier cambio, y esperá la confirmación.
- De a un cambio por vez; no encarar varias cosas juntas.
- No rompas lo que ya funciona: no cambies IDs de elementos, datos ni cálculos ya
  verificados sin avisarlo explícitamente.
- Ante cualquier ambigüedad, preguntá antes de asumir.

## Reglas técnicas (no romper)

- Un solo archivo .html autocontenido. Sin librerías ni CDNs: los gráficos se dibujan a
  mano sobre <canvas>. Debe correr offline, abriendo el archivo.
- Persistencia: localStorage (por navegador). Sin backend.
- No tocar el motor de cálculo ya verificado (weibullFit, gammaFn, weibullR) salvo pedido
  explícito.
- La función de IA solo corre dentro del entorno de artefactos de Claude; abierta como
  archivo local debe mostrar el fallback heurístico. Mantener siempre ese plan B.

## Control de versiones (git + GitHub)

- Los commits y push los hace Claude Code cuando el usuario lo pide, sin usar la terminal.
- Un cambio por persona por vez; siempre Pull antes de empezar y Push al terminar.
- Nunca dos personas editando el archivo a la vez.
