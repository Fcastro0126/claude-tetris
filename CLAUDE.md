# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-page Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no `package.json`.

## Running the game

There is no build/lint/test tooling. To run:

```bash
open index.html            # macOS, just opens the file directly
# or serve it locally:
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

Then open `http://localhost:8000` if using a local server. Changes to `game.js`, `style.css`, or `index.html` take effect on browser reload — no compilation needed.

## Architecture

Three files, all loaded directly by `index.html`:

- **`index.html`** — DOM structure: the main `<canvas id="board">` (300×600, i.e. `COLS × BLOCK` by `ROWS × BLOCK`), the side panel (score/lines/level/next-piece preview), and the pause/game-over overlay.
- **`style.css`** — dark/retro arcade visual theme.
- **`game.js`** — all game logic, structured around a `requestAnimationFrame` loop. Everything lives in module-level state variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) initialized in `init()`.

### Key mechanics in `game.js`

- **Board model**: a `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: defined as square matrices in `PIECES`. Rotation (`rotateCW`) is done by transposing + reversing rows — no piece-specific rotation tables.
- **Collision** (`collide`): checks board bounds and overlap with locked cells.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found, else the rotation is discarded.
- **Game loop** (`loop`): accumulates elapsed time each frame; when it exceeds `dropInterval`, the piece drops a row (or locks if it can't).
- **Line clearing** (`clearLines`): scans bottom-to-top, splices out full rows and unshifts empty ones at the top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by current `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Leveling**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.
- **Game over**: triggered in `spawn()` when a freshly spawned piece already collides with the board.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).

## Controls

| Key | Action |
|---|---|
| `←` / `→` | Move piece horizontally |
| `↑` or `X` | Rotate clockwise |
| `↓` | Soft drop |
| `Space` | Hard drop |
| `P` | Pause/resume |
