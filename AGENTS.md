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
- El estado del juego está en globals a nivel módulo en `game.js` (nave, balas, asteroides, partículas, power-ups, shieldBursts, puntaje, vidas, nivel).
- El loop principal con `requestAnimationFrame` está al final de `game.js`; `dt` está limitado a 0.05s.
- La entrada usa `keys[]` (mantenido) + `justPressed[]` (un solo disparo, limpiado automáticamente vía `pressed()`), así que espacio/enter dispara solo una vez por keydown.
- Los drops se tiran al destruir un asteroide con bala: cuatro tiradas **independientes** de `Math.random() < 0.08` en el bloque de colisión bala-vs-asteroide de `update()` (power-up velocidad, power-up triple, power-up escudo y estrella fugaz).
- Los power-ups comparten la clase `PowerUp` con un campo `type` (`'speed'` | `'triple'` | `'shield'`); las barras del HUD comparten el helper `drawPowerBar(timer, y, color, label)`.

## Power-up "Velocidad"

- **Drop de asteroides**: 8% de probabilidad al destruir un asteroide (bala vs asteroide), tirada independiente de las del triple, del escudo y de la estrella fugaz. Aparece como un ícono de relámpago cian (clase `PowerUp` con `type = 'speed'` en `game.js`).
- **Recogida**: al tocarlo con la nave, se activa el boost de 5 segundos.
- **Boost de nave**: `THRUST` (260 px/s²) se duplica a 520 px/s² durante el boost. `DRAG` (0.987) se mantiene, por lo que la velocidad terminal también se duplica. Rotación (3.5 rad/s) no cambia.
- **Timer**: si se recolecta otro power-up mientras el boost está activo, el timer se reinicia a 5s sin apilar tiempo.
- **Cancelación**: al morir (naves vs asteroide), el boost se cancela (`ship.speedBoostTimer = 0`). Al reaparecer la nave con invencibilidad, el boost está inactivo.
- **HUD**: barra de progreso cian (#0ff) con label "VEL" en (14, 36), dibujada por `drawPowerBar()`, mientras el boost dura.
- **Indicador visual**: la llama del propulsor de la nave cambia de naranja a azul brillante durante el boost.

## Power-up "Triple" (T)

- **Drop de asteroides**: 8% de probabilidad al destruir un asteroide (bala vs asteroide), **independiente** de los drops de velocidad, escudo y estrella fugaz (24% combinado de aparición de power-ups). Aparece como una letra "T" magenta (clase `PowerUp` con `type = 'triple'` en `game.js`).
- **Recogida**: al tocarlo con la nave, se activa el disparo triple de 5 segundos (`ship.tripleShotTimer = 5`).
- **Disparo**: `Ship.tryShoot()` lanza 3 balas desde la nariz en lugar de 1, con dispersión de ±4° respecto al ángulo de la nave (bala central + una a cada lado). El cooldown (0.2s) y la cadencia no cambian.
- **Timer**: si se recolecta otro "T" mientras está activo, el timer se reinicia a 5s sin apilar tiempo.
- **Coexistencia**: puede estar activo a la vez que el boost de velocidad y con el escudo; los dos timers se gestionan por separado (`tripleShotTimer` / `speedBoostTimer`) con sus propias barras.
- **Cancelación**: al morir, `tripleShotTimer` se cancela. Al reaparecer la nave (`Ship.reset()`), está inactivo.
- **HUD**: barra de progreso magenta (#f0f) con label "TRIPLE" en (14, 48), debajo de la barra de velocidad, dibujada por `drawPowerBar()`, mientras dura.

## Power-up "Escudo"

- **Drop de asteroides**: 8% de probabilidad al destruir un asteroide (bala vs asteroide), independiente de los drops de velocidad y triple (clase `PowerUp` en `game.js`, `type = 'shield'`).
- **Icono**: hexágono púrpura (#aa44ff) con pulso de escala, distinto del relámpago cian de velocidad y de la "T" magenta del triple.
- **Recogida**: al tocarlo con la nave, se activa el escudo (`ship.shieldActive = true`). Es de uso único: no se apila ni se reinicia si ya estaba activo (el power-up se consume sin efecto extra).
- **Efecto**: el escudo interfiere físicamente contra asteroides y estrellas fugaces (proyectiles enemigos actuales). Al chocar, el cuerpo se destruye al tocar el escudo (como si lo absorbiera) y el escudo desaparece (termina su efecto). Genera 25 partículas púrpuras + un anillo expansivo `ShieldBurst` (anillo púrpura que se expande de 28 a 60px en 0.35s con flash blanco interior).
- **Radio de escudo**: 28px alrededor de la nave (vs 12px del cuerpo), usando el mismo coeficiente de colisión 0.82 que la colisión normal.
- **No bloquea power-ups**: los power-ups amistosos pasan a través del escudo (no se absorben).
- **Sin barra de HUD**: a diferencia de velocidad y triple, el escudo no tiene barra de tiempo. El anillo púrpura pulsante alrededor de la nave (con glow exterior y núcleo blanco tenue) es el único indicador.
- **Prioridad sobre invencibilidad**: si la nave tiene escudo activo, este absorbe la colisión antes que el check de invencibilidad, incluso si la nave está invencible.
- **Cancelación**: al morir (`killShip()`: `ship.shieldActive = false`), al reaparecer (`ship.reset()` lo limpia), y al cambiar de nivel (`nextLevel()`).

## Asteroide especial "Estrella fugaz"

- **Drop de asteroides**: 8% de probabilidad al destruir un asteroide (bala vs asteroide), tirada independiente de las tres de power-up. Aparece como un asteroide más rápido de lo normal (clase `ShootingStar` en `game.js`). Se genera independientemente de los power-ups (velocidad 8% + triple 8% + escudo 8%).
- **Movimiento**: velocidad duplicada respecto a un asteroide normal del mismo tamaño.
- **Desaparición por tiempo**: dura un tiempo aleatorio entre 1 y 8 segundos, luego desaparece automáticamente (no por colisión).
- **Visual**: color naranja (#ff6600) con contorno rojo (#ff0000) y llamas rojas (trail naranja-rojo). Al destruirse genera partículas naranjas.
- **No se divide**: a diferencia de los asteroides normales, no se parte en fragmentos al ser destruido.
- **Interacción con escudo**: al ser subclase de `Asteroid` y compartir el mismo array, el escudo activo la absorbe igual que a un asteroide normal (genera `ShieldBurst` púrpura en su lugar de partículas naranjas).
- **Spawn adicional**: el 8% de spawn ocurre independientemente del asteroide destruido (incluye otro ShootingStar).

## Sistema de skins

- **Rotación**: tecla `Q` durante el juego rota al siguiente skin disponible. Funciona en estados `playing` y `dead`, no en `gameover`.
- **Disponibilidad**: las 6 skins están disponibles desde el inicio, sin progresión ni desbloqueo.
- **Definición** (array `SKINS` en `game.js`): cada skin define `id`, `name`, `verts` (polígono), `nose` (posición del cañón), `flame` (geometría del propulsor), `stroke`/`fill` (colores), y opcionalmente `accent`/`accentLines`/`window` para detalles visuales.
- **Skins actuales**: `classic` (blanco), `cyan-lance` (cian), `naranja-wing` (naranja), `verde-delta` (verde con acento), `rojo-cross` (rojo), `dorado-rocket` (dorado con ventana).
- **Persistencia**: `localStorage` guarda `asteroids.skin.current` (índice del skin actual) y `asteroids.skin.unlocked` (array de índices desbloqueados, siempre `[0..5]`). Se carga al iniciar con `loadSkin()`; si el valor guardado es inválido, se usa el índice 0.
- **HUD**: al cambiar de skin aparece un flash centrado con `SKIN: <NOMBRE>` (22px bold) y `Q: CAMBIAR` (12px), con fade-out de 1.5 segundos (alpha proporcional al timer).
- **Nave y balas**: `Ship.draw()` usa `verts`, `fill`, `stroke` y `flame` del skin. `tryShoot()` usa `SKINS[currentSkinIndex].nose` en lugar de un valor fijo. La llama del propulsor usa `flameColor` normal y `boostFlame` durante el boost.
- **Íconos de vidas**: `drawLifeIcon()` dibuja la silueta del skin actual en miniatura (escala 0.45x) con su color de `stroke`.
