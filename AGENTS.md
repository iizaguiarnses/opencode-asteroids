# AGENTS.md

Juego de Canvas HTML5 de un solo archivo (sin build, sin dependencias).

## Ejecución / verificación

Abre `index.html` directamente en el navegador, o sirve el repositorio:

```bash
npx serve .
```

Luego visita la URL localhost mostrada (por defecto `http://localhost:3000`).

No hay ningún paso de build, lint, typecheck ni test. Toda la lógica está en `game.js`, que es ES6+ puro cargado directamente por la página — sin bundler, sin transpilación.

## Convenciones

- El canvas está fijado en 800×600 (el elemento canvas en `index.html`). El mundo se envuelve de forma toroidal.
- El estado del juego está en globals a nivel módulo en `game.js` (nave, balas, asteroides, partículas, puntaje, vidas, nivel).
- El loop principal con `requestAnimationFrame` está al final de `game.js`; `dt` está limitado a 0.05s.
- La entrada usa `keys[]` (mantenido) + `justPressed[]` (un solo disparo, limpiado automáticamente vía `pressed()`), así que espacio/enter dispara solo una vez por keydown.
