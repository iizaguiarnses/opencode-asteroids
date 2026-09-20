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
- El estado del juego está en globals a nivel módulo en `game.js` (nave, balas, asteroides, partículas, power-ups, puntaje, vidas, nivel).
- El loop principal con `requestAnimationFrame` está al final de `game.js`; `dt` está limitado a 0.05s.
- La entrada usa `keys[]` (mantenido) + `justPressed[]` (un solo disparo, limpiado automáticamente vía `pressed()`), así que espacio/enter dispara solo una vez por keydown.

## Power-up "Velocidad"

- **Drop de asteroides**: 8% de probabilidad al destruir un asteroide (bala vs asteroide). Aparece como un ícono de relámpago cian (clase `PowerUp` en `game.js`).
- **Recogida**: al tocarlo con la nave, se activa el boost de 5 segundos.
- **Boost de nave**: `THRUST` (260 px/s²) se duplica a 520 px/s² durante el boost. `DRAG` (0.987) se mantiene, por lo que la velocidad terminal también se duplica. Rotación (3.5 rad/s) no cambia.
- **Timer**: si se recolecta otro power-up mientras el boost está activo, el timer se reinicia a 5s sin apilar tiempo.
- **Cancelación**: al morir (naves vs asteroide), el boost se cancela (`ship.speedBoostTimer = 0`). Al reaparecer la nave con invencibilidad, el boost está inactivo.
- **HUD**: barra de progreso cian (#0ff) con label "VEL" aparece debajo del SCORE (14, 36) mientras el boost dura.
- **Indicador visual**: la llama del propulsor de la nave cambia de naranja a azul brillante durante el boost.

## Asteroide especial "Estrella fugaz"

- **Drop de asteroides**: 8% de probabilidad al destruir un asteroide (bala vs asteroide). Aparece como un asteroide más rápido de lo normal (clase `ShootingStar` en `game.js`).
- **Movimiento**: velocidad duplicada respecto a un asteroide normal del mismo tamaño.
- **Desaparición por tiempo**: dura un tiempo aleatorio entre 1 y 8 segundos, luego desaparece automáticamente (no por colisión).
- **Visual**: color naranja (#ff6600) con contorno rojo (#ff0000) y llamas rojas (trail naranja-rojo). Al destruirse genera partículas naranjas.
- **No se divide**: a diferencia de los asteroides normales, no se parte en fragmentos al ser destruido.
- **Spawn adicional**: el 8% de spawn ocurre independientemente del asteroide destruido (incluye otro ShootingStar).
