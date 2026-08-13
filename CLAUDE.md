# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla Tetris — HTML5 Canvas + CSS + JavaScript. No dependencies, no build step, no package.json.

## Commands

There is no build/lint/test tooling. To run the game locally:

```bash
open index.html              # macOS, direct file open
python3 -m http.server 8000  # or any static server, then open http://localhost:8000
```

There are no automated tests, linter, or type checker in this repo — "done" here means manually verifying the change in the browser.

## Architecture

Three files, no modules/bundler: `index.html` (DOM + two canvases), `style.css` (dark/retro theme), `game.js` (all logic, ~300 lines, single global scope, `'use strict'`).

Key mechanics in `game.js`:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying the piece type that occupies it.
- **Pieces**: `PIECES` are square matrices (index 0 unused/`null`). Rotation is done via `rotateCW` (transpose + reverse), not by storing pre-rotated shapes.
- **Collision** (`collide`): checks board bounds and overlap against already-locked cells.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` until a non-colliding position is found, else the rotation is discarded.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates `dt` in `dropAccum` and advances the piece once `dropAccum >= dropInterval`.
- **Line clear** (`clearLines`): scans bottom-up, splices full rows out and unshifts empty rows at the top; re-checks the same row index (`r++`) since rows shift down.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
- **Level/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.
- All game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars) lives in module-level `let` bindings reset by `init()` — there is no state container/class.

If `COLS`, `ROWS`, or `BLOCK` change, the `<canvas id="board">` `width`/`height` in `index.html` must be updated to match (`COLS × BLOCK`, `ROWS × BLOCK`).
