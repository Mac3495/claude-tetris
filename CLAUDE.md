# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open directly or via a local server:

```bash
open index.html
# or
python3 -m http.server 8000
```

## Architecture

Three files, no dependencies:

- `index.html` — DOM structure: `<canvas id="board">` (300×600px), aside panel with score/lines/level/next-piece preview (`<canvas id="next-canvas">`), and a shared overlay div for pause and game-over states.
- `style.css` — dark/retro arcade theme, flexbox layout.
- `game.js` — all game logic (~305 lines, `'use strict'`, no modules).

### game.js key concepts

**State**: global mutable vars — `board` (ROWS×COLS matrix, 0=empty or color index 1–7), `current`/`next` piece objects `{type, shape, x, y}`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`.

**Game loop**: `requestAnimationFrame(loop)` accumulates delta time in `dropAccum`; fires gravity drop when `dropAccum >= dropInterval`. `animId` is used to cancel via `cancelAnimationFrame` on pause/game-over/restart.

**Piece rotation**: `rotateCW(shape)` — transpose + column-reverse. `tryRotate()` applies wall kicks `[0, -1, 1, -2, 2]` column offsets before giving up.

**Locking flow**: `lockPiece()` → `merge()` (writes piece into board) → `clearLines()` (splice + unshift, updates score/level/speed) → `spawn()` (promotes next→current, detects game-over if collision at spawn).

**Speed formula**: `dropInterval = Math.max(100, 1000 − (level − 1) × 90)` ms; level increases every 10 lines.

**Rendering**: `draw()` clears canvas, draws grid, board cells, ghost piece (`globalAlpha=0.2` at `ghostY()`), then current piece. `drawNext()` renders the preview on `next-canvas`.

### Tunable constants (top of game.js)

| Constant | Default | Note |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Also update canvas `width`/`height` in HTML (`COLS×BLOCK` × `ROWS×BLOCK`) |
| `BLOCK` | 30 | Pixel size per cell |
| `COLORS` | 7 colors | Index matches piece type (1–7) |
| `LINE_SCORES` | `[0,100,300,500,800]` | Multiplied by level |
