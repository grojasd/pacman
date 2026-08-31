# AGENTS.md

## Resumen
- Clon de Pac-Man en JS / HTML / CSS vanilla. No hay `package.json`, ni build, ni tests, ni comandos de lint/typecheck.
- Para verificar cambios: abrir `src/index.html` en un navegador (o servir `src/` con `npx serve src` o similar).

## Arquitectura
- Los scripts se cargan como etiquetas `<script>` simples en `src/index.html:19-22`, en orden de dependencia: `maze.js` → `game.js` → `render.js` → `main.js`. No hay ES modules; los archivos se comunican mediante globales de `window.*` (`MAZE`, `createGame`, `draw`, etc.). Mantén este patrón al añadir archivos — exporta las funciones nuevas a `window`.
- Tablero basado en celdas de 28x31, con el origen arriba-izquierda. Casillas: `1` pared, `2` dot, `3` puerta de la casa de fantasmas, `0` vacío. La puerta (`3`) bloquea a Pac-Man pero no a los fantasmas (`isWall` en game.js).
- `MAZE` (maze.js:50) es la matriz maestra inmutable; `createGame` (game.js:18) la copia en profundidad a `game.grid` para poder comer dots y reiniciar partidas. El renderer siempre dibuja `game.grid`, nunca `MAZE`.
- Los actores se mueven en incrementos sub-celda por frame (`aligned()` hace el snap en game.js), y la fila 14 es el túnel que envuelve con `wrapTunnel`.

## Flujo de trabajo (spec-driven)
- Este proyecto existe para practicar desarrollo spec-driven. Usa las skills `/spec` y `/spec-impl` (`.agents/skills/`, fijadas en `skills-lock.json`).
- Las specs viven en `specs/` — la carpeta aún no existe; `/spec` la crea y genera `specs/.spec-config.yml` (`AutoCreateBranch: true`).
- `/spec-impl` solo implementa specs cuyo estado **significa "Approved"** (en cualquier idioma); crea/cambia a una rama `spec-NN-slug` y nunca hace commits por su cuenta.

## Convenciones
- El idioma del repo es el español: README, comentarios de código y textos del juego están en español ("GANASTE"/"PERDISTE"). Respeta ese idioma en textos dirigidos al usuario y en prosa. La skill `/spec` también responde en el idioma del prompt inicial.
- Estilo de código: comillas simples, punto y coma, y espacios dentro de paréntesis y llaves — `( x )`, `{ x: 1 }`, `MAP.map( ( row ) => ... )`. No hay formateador configurado, así que imita los archivos existentes a mano.