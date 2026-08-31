# SPEC 01 — Cuatro fantasmas con conductas distintas

> **Estado:** Aprobado
> **Depende de:** ninguna
> **Fecha:** 2026-08-30
> **Objetivo:** Añadir cuatro fantasmas al juego, cada uno con una conducta de persecución propia (perseguidor, predictor, flanqueador y errático) y liberación cronometrada desde la casa de fantasmas.

## Alcance

**Incluye:**
- Cuatro fantasmas simultáneos con colores propios: rojo, rosa, cian y naranja.
- Cuatro conductas: `chaser` (persigue a Pac-Man), `predictor` (apunta delante de Pac-Man), `flanker` (rodea usando la posición del perseguidor), `erratic` (aleatorio).
- Liberación cronometrada: los 4 arrancan dentro de la casa; el perseguidor sale al instante y el resto salen uno a uno cada ~2 s.
- Misma velocidad de movimiento (0.1 celda/frame) para los 4.

**Fuera de alcance (futuras specs):**
- Power pills y modo "fantasma aturdido" (comedible) — feature independiente.
- Velocidad distinta por fantasma.
- Liberación basada en dots comidos.

## Modelo de datos

```js
// maze.js — 4 inicios, todos dentro de la casa (orden = orden de salida)
const GHOST_STARTS = [
  { x: 13, y: 13, kind: 'chaser' },    // rojo, activo desde el inicio
  { x: 14, y: 13, kind: 'predictor' }, // rosa
  { x: 13, y: 14, kind: 'flanker' },   // cian
  { x: 14, y: 15, kind: 'erratic' },   // naranja
];
```

```js
// game.js — estado nuevo por fantasma
{
  x, y, dir, speed, kind,
  state: 'pen' | 'leaving' | 'active',
}
// 'pen'     -> espera inmóvil en la casa
// 'leaving' -> sube por su columna (13 o 14) hasta cruzar la puerta
// 'active'  -> conduce con decideGhost normal

// game.js — temporizador de lanzamiento
const GHOST_RELEASE_DELAY = 720; // frames (~12 s a 60 fps)
game.ghostReleaseTimer = GHOST_RELEASE_DELAY;
```

Las celdas (13,13), (14,13), (13,14) y (14,15) son interiores de la casa (espacios en filas 13-15, columnas 13/14) y quedan alineadas con los tiles de puerta (3) de la fila 12. La puerta ya es transitable para fantasmas (`isWall` en game.js).

## Plan de implementación

1. **`src/js/maze.js`:** sustituir `GHOST_STARTS` por los 4 inicios con los kinds nuevos. La partida sigue funcionando (cualquier kind distinto de `'hunter'` ya cae en la rama aleatoria de `decideGhost`).
2. **`src/js/game.js` — estados de salida:** añadir `state`, `ghostReleaseTimer` y el helper `initGhosts()` (perseguidor `'active'`, resto `'pen'`). En `moveGhost`: `'pen'` no avanza; `'leaving'` fuerza `dir = 'up'` hasta cruzar la puerta (línea 11) y pasa a `'active'`. En `update`: decrementar el timer y, al llegar a 0, liberar al primer fantasma en estado `'pen'` y reiniciar el timer. `resetPositions` vuelve a llamar a `initGhosts()` para re-liberar tras perder una vida.
3. **`src/js/game.js` — conductas por kind:** renombrar `'hunter'` -> `'chaser'` y `'random'` -> `'erratic'`; añadir `ghostTarget( game, g )`:
   - `chaser` -> posición redondeada de Pac-Man.
   - `predictor` -> Pac-Man + 2 celdas en su dirección (clamped al tablero).
   - `flanker` -> celda espejo de Pac-Man respecto al perseguidor: `2 * pacman - ghost[0]` (clamped).
   - `erratic` -> sin objetivo (elección aleatoria actual).
   `decideGhost` elige en cada cruce la dirección que minimiza la distancia Manhattan al objetivo (reusa la lógica actual); `erratic` sigue aleatorio.
4. **`src/js/render.js` — colores:** reordenar `GHOST_COLORS` a `[ '#ff0000', '#ffb8ff', '#00ffff', '#ffb852' ]` (rojo/rosa/cian/naranja, por índice = orden de `GHOST_STARTS`).

Cada paso queda jugable y verificable en el navegador.

## Criterios de aceptación

- [ ] Al iniciar partida hay exactamente 4 fantasmas visibles dentro de la casa.
- [ ] El perseguidor (rojo) sale al instante; el resto salen de uno en uno con ~12 s de separación.
- [ ] Un fantasma antes de su turno permanece inmóvil en la casa.
- [ ] Al salir, el fantasma sube y cruza la puerta sin atascarse en los tiles `3`.
- [ ] El perseguidor elige en cada cruce el camino que minimiza la distancia a Pac-Man (persecución agresiva).
- [ ] El predictor apunta a la celda 2 posiciones delante de Pac-Man según su dirección.
- [ ] El flanqueador apunta a la celda espejo de Pac-Man respecto al perseguidor.
- [ ] El errático elige dirección aleatoria entre las válidas en cada cruce.
- [ ] Los 4 avanzan a la misma velocidad.
- [ ] Al perder una vida, los 4 vuelven a la casa y se re-liberan como al inicio.
- [ ] No hay errores en la consola y la partida se puede ganar y perder.

## Decisiones

- **Sí:** cuarteto clásico (chaser/predictor/flanker/erratic). Cubre "uno persigue agresivamente" con lógica conocida.
- **No:** 4 variantes a medida. El cuarteto clásico ya cumple el requisito.
- **Sí:** liberación cronometrada (~12 s; el primero inmediato). Fiel al original sin añadir contador de dots.
- **No:** liberación por dots comidos. Más fiel pero más compleja; este MVP usa tiempo.
- **No:** power pills / frightened. Feature independiente -> otra spec.
- **Sí:** misma velocidad (0.1) para todos. Sin pills, un fantasma más rápido volvería la partida injusta.
- **Sí:** puntuación Manhattan por objetivo (reusa la lógica `'hunter'` actual).
- **No:** BFS / distancia euclidiana. Innecesario a esta escala.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Dificultad alta sin power pills con 4 persecutores | Misma velocidad, salidas espaciadas 12 s, `erratic` aleatorio reduce la presión. |
| Fantasma atascado en la puerta al salir | Columnas 13/14 paralelas sobre tiles puerta transitable; los fantasmas no colisionan entre sí. |
| `resetPositions` reubica un fantasma encima de Pac-Man | Comportamiento ya existente; la colisión se detecta el frame siguiente (aceptado para MVP). |

## Qué **no** incluye esta spec

- Power pills y fantasmas aturdidos/comedibles.
- Velocidades o ritmos distintos por fantasma.
- Cambios en el laberinto ni en el HUD.

Cada una de esas, si llega, va en su propia spec.