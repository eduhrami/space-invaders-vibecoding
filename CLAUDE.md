# Space Invaders temático — instrucciones para continuar

Este proyecto tiene dos partes que conviven en la misma conversación de origen:

1. **El juego**: `index.html`, un Space Invaders de un solo archivo construido de forma incremental (vibecoding).
2. **El taller**: un manual publicado como Artifact que usa el desarrollo de este mismo juego como caso de estudio para enseñar a crear juegos retro con IA.

Este archivo es el punto de partida para retomar cualquiera de las dos cosas en una sesión nueva.

## Repositorio y hosting

- GitHub: `https://github.com/eduhrami/space-invaders-vibecoding` (público, remoto `origin`, rama `master`).
- GitHub Pages activado sirviendo la raíz de `master` — cualquier `git push` a `master` actualiza ambas URLs en un par de minutos:
  - Juego: `https://eduhrami.github.io/space-invaders-vibecoding/`
  - Manual del taller: `https://eduhrami.github.io/space-invaders-vibecoding/taller/manual.html`

## Cómo probar el juego

- Todo vive en `index.html` (HTML + CSS + JS inline). No hay build, ni dependencias, ni servidor.
- Abrir en el navegador de Windows desde WSL: `wslview index.html &`
- Antes de dar por buena una edición, validar sintaxis sin necesidad de navegador:
  ```
  python3 -c "
  import re
  html = open('index.html').read()
  m = re.search(r'<script>(.*)</script>', html, re.S)
  open('/tmp/check.js','w').write(m.group(1))
  "
  node --check /tmp/check.js && echo OK
  ```
- Después de cada cambio visible, reabrir con `wslview` y confirmar antes de seguir.
- Git ya está inicializado (rama `master`). Un commit por feature terminada, no por archivo tocado.

## Arquitectura del juego

- **Loop**: `loop()` → `update()` + `draw()` vía `requestAnimationFrame`. Canvas de 600×600.
- **Temas**: `enemyTheme` es `'insects'` o `'monsters'`, elegido en el menú inicial. Todo el arte de enemigos vive en pares de arrays paralelos (`insectDrawers`/`monsterDrawers`, `insectBossDrawers`/`monsterBossDrawers`, etc.) para que agregar un tema nuevo sea replicar el patrón, no reescribir lógica.
- **Enemigos regulares**: grid de `aliens[]` con `row`/`col`; cada fila usa un dibujante distinto. Tienen `hp`, `burning` (fuego) y `frozen` (hielo).
- **Items sorpresa**: registro genérico `ITEM_TYPES` (línea ~207). Cada entrada define `label`, `color`, `icon`, `weight` (opcional, default 1 — más bajo = más raro) y `onPickup(effects, def)`. La selección es ponderada (`pickRandomItemType()`). Para agregar un item nuevo: sumar una entrada acá, nada más.
  - **Cuándo caen**: cada fila tiene un único enemigo "portador" asignado al azar al armar la oleada (`buildAliens()`, campo `alien.itemDrop`), invisible hasta que muere. No es por cantidad de bajas — es siempre ese enemigo puntual el que suelta el item, sin importar cómo muera (bala normal, fuego, hielo roto, bomba).
  - `fireBullet`: perfora 2 enemigos por disparo (`FIRE_PIERCE_COUNT`), quema con parpadeo rojo en vez de matar al instante, se apila (2 items = doble disparo, tope `FIRE_MAX_STACKS = 8`).
  - `iceBullet`: congela al enemigo y a sus vecinos de fila (azul, no disparan); un segundo impacto sobre uno congelado lo rompe.
  - `shield`: +2 golpes de escudo (`player.shieldHits`), se ve como anillo brillante alrededor de la nave.
  - `extraLife`: raro (`weight: 1.4`, calculado para igualar lo que antes sumaban el item de reintentar + vida extra juntos — ver commit `40dcf5c`), reaparece en el mismo lugar con 1 vida, sin reconstruir la oleada/boss.
  - `bomb`: tecla **B**, no gasta el arma equipada. Misil lento con estela de humo; al chocar destruye un bloque de 3×3 enemigos (`detonateBomb`) o resta 9 hp al boss. Muestra el cartel "¡BOMBA LEGENDARIA!" al disparar.
  - `machineGun`: modificador de cadencia, no un tipo de bala propio — baja el enfriamiento de disparo a 1/5 (`MACHINEGUN_COOLDOWN = NORMAL_COOLDOWN / 5`) sin importar qué arma esté equipada. Se combina con fuego/hielo (ametralladora de fuego, de hielo) en vez de competir con ellos.
- **Bosses**: aparecen al limpiar la oleada de cada nivel (`spawnBoss()`, línea ~415). `TOTAL_LEVELS = 20`.
  - Niveles normales → boss **pequeño** (3×3), rotando por `insectBossDrawers`/`monsterBossDrawers` (15 diseños cada uno, `smallBossIndex()` garantiza que ninguno se repite en una partida completa de 20 niveles).
  - Múltiplos de 4 (4, 8, 12, 16) → boss **mayor** (4×4), uno único por aparición (`insectMajorBossDrawers`/`monsterMajorBossDrawers`, 4 diseños cada uno).
  - Nivel 20 → **Rata Furiosa** (`drawRat`), 5×5, `FINAL_BOSS_HP = 45`, la misma para ambos temas.
  - Cada boss tiene nombre (arrays `*BossNames` paralelos a los `*Drawers`) mostrado arriba de su barra de vida, con anuncio "¡BOSS!" al aparecer.
  - Por ciclo de 4 niveles suben velocidad, frecuencia de disparo y daño (desde el ciclo 3 sus balas quitan 2 vidas). Descienden 16px en cada rebote, igual que la oleada normal.
  - Sueltan un item sorpresa cada `BOSS_ITEM_DROP_INTERVAL = 5` impactos recibidos (cualquier tipo de disparo cuenta).

## Convenciones a mantener

- Un solo archivo, sin frameworks ni pasos de build — es parte del pitch del taller ("se abre con doble clic").
- Comentarios solo para explicar el *porqué* de una decisión no obvia (por qué un número es ese y no otro), nunca el *qué* hace el código.
- Antes de emprender una tarea de arte/contenido grande (por ejemplo, diseñar 20 personajes de boss nuevos), confirmar el alcance con el usuario en vez de asumir — así se hizo con `AskUserQuestion` para la cadencia de bosses y la lista de personajes.
- Los prompts de este proyecto tienden a ser pequeños y concretos, uno por turno. Seguir ese patrón al proponer cambios: un feature por commit, probado en navegador antes de seguir.

## El taller (manual publicado como Artifact)

- URL publicada: `https://claude.ai/code/artifact/52d61da3-e8fd-4a21-8dfe-4b9ba5d47734` ("Vibecoding Arcade" — manual de taller en español neutro latino).
- Copia fuente versionada: **`taller/manual.html`** en este repo — es el mismo HTML que está publicado (con las fuentes ya incrustadas en base64), así que se puede abrir directo con doble clic o servir desde GitHub Pages sin depender del Artifact.
- Contenido: qué es vibecoding, tabla de 14 pasos con el historial real de este juego como caso de estudio (con los pedidos casi textuales, no resumidos, y el detalle concreto de qué se construyó en cada uno), la regla de oro ("un prompt, un cambio visible"), guion de taller de ~90 min, menú de juegos recomendados (Pong/Viborita/Rompe-bloques en nivel 1; Space Invaders/Flappy en nivel 2; Pac-Man/Tetris/2048 en nivel 3 avanzado, con advertencias técnicas específicas de cada uno), banco de prompts por categoría, buenas prácticas para ahorrar tokens, y retos para después.
- Diseño: identidad "manual de arcade" — modo claro como papel de manual envejecido, modo oscuro como pantalla CRT. Tipografías incrustadas como `@font-face` en base64 (Agency FB para titulares tipo marquesina, Corbel para cuerpo de texto), tomadas de `/mnt/c/Windows/Fonts/`.

### Cómo seguir editando el manual

El HTML de `taller/manual.html` ya trae las fuentes incrustadas (~950 KB) — no hace falta regenerarlo desde cero para cambios de texto/CSS, se puede editar directo. Solo hay que volver a incrustar las fuentes si se reescribe el archivo entero desde una plantilla sin ellas (ver el script de incrustado en el historial de esta conversación, usa `/mnt/c/Windows/Fonts/AGENCYB.TTF`, `AGENCYR.TTF`, `corbel.ttf`, `corbelb.ttf`). Después de editar:

1. Actualizar `taller/manual.html` en el repo y comitear.
2. Republicar con la herramienta `Artifact` pasando `url` con la misma URL de arriba, para que se actualice en el mismo lugar en vez de crear una página nueva.

Si en algún momento se perdiera la copia local y hiciera falta partir del HTML ya publicado, se puede traer con `WebFetch` sobre la URL de arriba.

### Pendientes que el usuario dejó abiertos sobre el taller

- Ajustar la duración del guion (hoy pensado para ~90 min, recortable a 60).
- Posibilidad de sumar más juegos al menú.
- Adaptar el contenido a un rango de edad específico.

## Ideas mencionadas pero no implementadas (backlog)

- Modo difícil seleccionable desde el menú.
- Modo de dos jugadores compartiendo teclado.
- Más items sorpresa siguiendo el patrón de `ITEM_TYPES` (el sistema ya está preparado para esto).
- Efectos de sonido puntuales (disparo, explosión, golpe) — hoy solo hay música de fondo, ver sección de audio abajo.

## Audio (música de fondo generada por código)

- Sin archivos externos: todo con Web Audio API (`AudioContext`, osciladores). Se arranca dentro de `startGame(theme)` (línea ~590), porque los navegadores exigen un gesto del usuario para iniciar audio.
- `startNatureMusic()` (insectos): melodía pentatónica suave (`NATURE_SCALE`/`NATURE_MELODY`) sobre un drone grave, con un "grillo" agudo aleatorio de vez en cuando (`playChirp`).
- `startHorrorMusic()` (monstruos): drone grave con vibrato lento (LFO en la frecuencia) más una segunda voz a distancia de tritono que entra y sale, y un "stinger" ocasional (`playStinger`).
- `stopMusic()` frena y limpia los osciladores sostenidos (`musicNodes`) antes de arrancar la otra pista — se llama automáticamente al elegir tema, incluida la revancha después de game over/victoria.
- Tecla **M** alterna silencio (`toggleMusicMute()`), afecta un único `GainNode` maestro (`musicGain`).

## Historial de commits (más antiguo primero)

1. `c5b53f2` — Juego base con insectos como enemigos.
2. `f044390` — Menú principal: elegir historia de insectos o monstruos.
3. `6603ce5` — Sistema de items sorpresa extensible + balas de fuego.
4. `222050c` — Balas de hielo + rediseño del drop de fuego.
5. `ee2054d` — Pelea de boss al limpiar cada oleada.
6. `e2bf64e` — Escudo, reintento (luego eliminado), bosses por nivel, rata furiosa.
7. `ad054b0` — Campaña de 20 niveles con bosses pequeños/mayores/final.
8. `079e0c0` — Apilamiento de fuego + bosses que descienden.
9. `29e9eae` — 20 diseños nuevos de boss pequeño (sin repetir ninguno).
10. `6664f54` — Drop de item cada 5 disparos al boss.
11. `71f1b49` — Item de vida extra (raro).
12. `40dcf5c` — Nombres de boss visibles + eliminación del item de reintento.
13. `879baa2` — Item de bomba (tecla B, área 3×3).
14. `2f08407` — Bomba con estética de misil, humo y explosión.
