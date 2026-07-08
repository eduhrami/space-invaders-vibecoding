# Space Invaders Vibecoding

Un Space Invaders temático (insectos vs. monstruos, power-ups, 20 niveles y 39 jefes distintos) escrito por completo mediante **vibecoding**: ni una línea de `index.html` se tipeó a mano, todo salió de describirle el juego, pedido por pedido, a un modelo de IA dentro de Claude Code.

Este repositorio es a la vez el juego y el caso de estudio de un taller que enseña a hacer lo mismo.

## Jugarlo / verlo

- 🕹️ Juego: **https://eduhrami.github.io/space-invaders-vibecoding/**
- 📖 Manual del taller: **https://eduhrami.github.io/space-invaders-vibecoding/taller/manual.html**

O simplemente abrir `index.html` con doble clic — no necesita servidor.

## Ficha técnica

| | |
|---|---|
| **Herramienta** | [Claude Code](https://claude.com/claude-code) (CLI) |
| **Modelo** | Claude Sonnet 5 (`claude-sonnet-5`) |
| **Metodología** | Vibecoding — el código se generó completo a partir de instrucciones en lenguaje natural, en español, iterando de a un cambio pequeño por vez |
| **Stack** | HTML5 + CSS + JavaScript vanilla, sin frameworks ni dependencias |
| **Render** | `<canvas>` 2D, loop propio vía `requestAnimationFrame` |
| **Build** | Ninguno — un solo archivo autocontenido, se abre directo en el navegador |
| **Tamaño** | `index.html`: ~2500 líneas / ~75 KB |
| **Control de versiones** | Git, con un commit por feature terminada (17 commits al momento de este README) |
| **Hosting** | GitHub Pages, sirviendo la raíz de `master` |

## Estructura del repositorio

```
.
├── index.html        # el juego completo (HTML + CSS + JS inline)
├── taller/
│   └── manual.html   # manual del taller de vibecoding (mismo HTML que el Artifact publicado)
├── CLAUDE.md          # notas de arquitectura y continuación para retomar el proyecto
└── README.md          # este archivo
```

## Qué tiene el juego

- Dos historias intercambiables desde el menú (insectos / monstruos), cada una con su propio set de enemigos dibujados en canvas.
- Sistema de power-ups extensible (`ITEM_TYPES`): balas de fuego (perforan y queman), hielo (congela y rompe), escudo, vida extra, y una bomba tipo misil con estela de humo y explosión.
- 20 niveles: bosses pequeños que nunca se repiten (15 diseños por historia), bosses mayores únicos cada 4 niveles, y un jefe final compartido — la Rata Furiosa.

El detalle completo de arquitectura está en [`CLAUDE.md`](./CLAUDE.md).

## Sobre el taller

`taller/manual.html` es un manual pensado para enseñar esta misma metodología a otras personas (incluyendo niños): qué es vibecoding, el historial real de pedidos que hicieron crecer este juego paso a paso, un guion de taller de ~90 minutos, y un menú de otros juegos retro simples (Pong, Tetris, Pac-Man, etc.) con los que practicar.
