# AGENTS.md

Guidance for coding agents working in this repository.

## Project Overview

**GA_Racing** ("NeedForJS") is a top-down arcade racing game written in vanilla
JavaScript and DOM rendering (no canvas / no game engine), packaged as an
Electron desktop app. The player controls a car with arrow keys, dodges enemy
cars scrolling toward them, and accumulates score until a collision. It was
built as a learning project (see the "Homework N" commits). UI text is in
Russian — keep it that way unless asked otherwise.

## Commands

```bash
npm install        # install dependencies (Electron 10)
npm start          # launch the game via Electron
```

There is no build, test, or lint script yet. Linting rules live in
`.jshintrc` (jshint, ES6, 2-space indent, camelcase) — run
`npx jshint script.js main.js` if you have jshint available.

## Architecture

| File         | Role                                                                 |
|--------------|----------------------------------------------------------------------|
| `main.js`    | Electron main process. Creates a 500×800 `BrowserWindow` and loads `index.html`. |
| `index.html` | Single page: `.game` → `.score`, `.start` (overlay), `.gameArea`. Loads `script.js` (no module system). |
| `script.js`  | All game logic in the renderer, plain globals. Core structures: `keys` (arrow-key state), `setting` (game state: start/score/speed/traffic + player x/y). Game loop is `requestAnimationFrame(playGame)`. |
| `style.css`  | Reset + game styling. `.car`/`.enemy` are absolutely-positioned 50×100px divs with sprite backgrounds; `.gameArea` is a 300px-wide scrolling road. |
| `image/`     | Sprites: `player.png` + 8 enemy variants. Note: `script.js` currently hardcodes `enemy2.png`; the other enemy sprites are unused. |

Game flow: click `.start` → `startGame()` builds road lines and enemies →
`playGame()` loop moves road (`moveRoad`), enemies, and the player car, and
checks AABB collision via `getBoundingClientRect` in `moveEnemy`. Collision
stops the loop and shows the start overlay again.

## Conventions

- Vanilla ES6 JavaScript, no frameworks, no bundler, no imports in the
  renderer — keep it dependency-free.
- 2-space indentation, camelCase, double quotes (see `.jshintrc`).
- Game tuning values live in the `setting` object (`speed`, `traffic`);
  element sizes (50×100px car, 100px line spacing) are tied to the CSS —
  change them together.

## Known Issues / Gotchas

- **Electron 10 (Oct 2020) is very old** and has no native Apple Silicon
  build — it may fail to launch on modern macOS without Rosetta.
- `package.json` lists npm packages `path` and `url` as devDependencies —
  these shadow Node built-ins and should be removed. `main.js` also has an
  unused `fileURLToPath` import.
- `index.html` has no `<!DOCTYPE html>` (quirks mode).
- `startRun`/`stopRun` call `preventDefault()` on **every** keydown, which
  blocks devtools/F5 shortcuts inside Electron.
- Movement and score are per-frame, not time-based — game speed depends on
  the display refresh rate.
