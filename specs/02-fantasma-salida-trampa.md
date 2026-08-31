# SPEC 02 — Fantasmas quedan atrapados al salir de la casa

> **Estado:** Aprobado
> **Depende de:** SPEC 01
> **Fecha:** 2026-08-31
> **Objetivo:** Corregir el bug que atrapa fantasmas dentro de la casa de fantasmas tras activarse, impidiéndoles salir a perseguir a Pac-Man.

## Alcance

**Incluye:**
- Bloquear la re-entrada de fantasmas activos a través de los tiles de puerta (valor 3) cuando se mueven hacia abajo.
- Verificar que la salida inicial (estado `'leaving'`) funciona correctamente desde cualquier posición alineada.

**Fuera de alcance (futuras specs):**
- Cambios en la conducta de persecución (`decideGhost`).
- Power pills o modo fantasma aturdido.
- Cambios en la velocidad o la dificultad.

## Modelo de datos

Esta spec no introduce datos nuevos. Reutiliza el modelo de SPEC 01: estados `'pen'`, `'leaving'`, `'active'` en `game.js`, y las funciones `isWall` / `canMove`.

## Causa del bug

Cuando un fantasma alcanza y=11 y pasa a `'active'`, `decideGhost` elige `'down'` porque Pac-Man siempre está por debajo de la casa (y=23). El fantasma re-entra por la puerta (tiles valor 3 en fila 12, cols 13-14). Una vez dentro, `decideGhost` siempre prefiere `'down'` (distancia Manhattan menor), creando un ciclo inf salvable de 6 celdas dentro de la casa.

**Secuencia exacta:**
1. Timer expira → fantasma pasa a `'leaving'` → sube por la columna → cruza puerta → llega y=11 → pasa a `'active'`.
2. En y=11, `decideGhost` elige `'down'` (hacia Pac-Man) → fantasma desciende a y=12 (puerta) → y=13 (interior).
3. Dentro, `decideGhost` siempre prefiere `'down'` → fantasma oscila entre (13,14)→(13,15)→(12,15)→(11,15)→(11,14)→(12,14)→(13,14) → nunca sale.

## Plan de implementación

1. **`src/js/game.js` — función auxiliar `isDoor`:** añadir helper que dado `(grid, x, y)` devuelva `true` si `grid[y][x] === 3`. Colocarla junto a `isWall` (línea 67).
2. **`src/js/game.js` — modificar `canMove`:** en la función `canMove` (línea 77), añadir condición: si `actor === 'ghost'`, `dir === 'down'` e `isDoor( grid, tx, ty )`, devolver `false`. Esto bloquea la re-entrada por puerta sin afectar la salida (dir `'up'`).
3. **Verificación manual:** abrir `index.html`, iniciar partida, observar que los fantasmas 2-4 salen de la casa al cumplirse el timer y permanecen fuera persiguiendo a Pac-Man sin re-entrar.

Cada paso deja el sistema funcional y verificable en el navegador.

## Criterios de aceptación

- [ ] Al iniciar partida, los 4 fantasmas salen de la casa cuando toca su turno.
- [ ] Un fantasma que ya salió no re-entra a la casa por la puerta (tiles valor 3).
- [ ] Un fantasma en estado `'leaving'` sube y cruza la puerta sin atascarse.
- [ ] Tras perder una vida, los fantasmas se re-liberan correctamente (mismo comportamiento que al inicio).
- [ ] No hay errores en la consola.

## Decisiones

- **Sí:** bloquear re-entrada vía `canMove` (condición en `canMove`). Es la solución más localizada — un cambio en una función existente, sin alterar estados ni añadir flags.
- **No:** añadir flag temporal `'justExited'` tras activarse. Más complejo, requiere timer/frame counter, y el bloqueo en `canMove` ya cubre el caso.
- **No:** hacer que `decideGhost` detecte la casa y fuerce `'up'`. Cambiaría la lógica de persecución y afectaría conductas no relacionadas.
- **No:** convertir fantasmas activos dentro de la casa a `'leaving'`. Altera el modelo de estados y puede crear conflictos con `resetPositions`.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Un fantasma activo que empieza dentro de la casa (tras `resetPositions`) necesita salir | `decideGhost` ya elige `'up'` desde la mayoría de celdas interiores (distancia Manhattan favorece subir hacia la puerta cuando no hay opción `'down'` mejor). El bloqueo no afecta movimientos `'up'`. |
| Cambio en `canMove` afecta otros usos de la función | `canMove` solo se usa en `movePacman` y `moveGhost`. La condición nueva solo aplica a `actor === 'ghost'` y `dir === 'down'` con tile puerta — efecto mínimo. |

## Qué **no** incluye esta spec

- Cambios en la conducta de persecución de los fantasmas.
- Power pills o fantasmas aturdidos/comedibles.
- Velocidades o ritmos distintos por fantasma.
- Cambios en el laberinto ni en el HUD.

Cada una de esas, si llega, va en su propia spec.
