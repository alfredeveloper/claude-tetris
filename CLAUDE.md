# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build step, no linter, no test suite. `README.md` is the user-facing documentation (in Spanish); UI strings (`PAUSA`, `Reiniciar`, `Puntuación`) are in Spanish too — keep new user-facing text in Spanish.

## Running

Open `index.html` directly, or serve the directory statically and visit `http://localhost:8000`:

```bash
python3 -m http.server 8000
```

Verification is manual in the browser.

## Architecture

`index.html` provides the DOM (board canvas, side panel, overlay); `game.js` looks elements up by id at load time and holds all logic; `style.css` is presentation only. `game.js` is a plain classic script (`'use strict'`, no modules) with all game state in module-level `let` variables that `init()` resets — `init()` is also the restart handler.

Key conventions that span functions:

- **Cell values double as piece type and color index.** `PIECES[t]` shapes contain the value `t` (not `1`), `merge()` copies those values into `board`, and `drawBlock` looks them up in `COLORS[t]`. Index `0` / `null` means empty in all three arrays, so they must stay aligned.
- **Rotation** is a generic matrix rotation (`rotateCW`) plus a simple kick list `[0, -1, 1, -2, 2]` in `tryRotate` — not SRS. Shapes are square matrices padded with zeros so rotation pivots correctly.
- **Game loop**: `loop()` runs on `requestAnimationFrame`, accumulating `dropAccum` until `dropInterval`. Pausing and game over cancel the frame via `animId`; resuming resets `lastTime` so the elapsed pause isn't counted as a drop.
- **Piece lifecycle**: `lockPiece()` → `merge()` → `clearLines()` (updates lines/score/level/`dropInterval`) → `spawn()`, which promotes `next` to `current` and calls `endGame()` if the new piece collides immediately.
- **Canvas size is hard-coded in `index.html`** (`300×600` board = `COLS×BLOCK` by `ROWS×BLOCK`; `120×120` preview drawn at 30px for a 4×4 area). Changing `COLS`, `ROWS`, or `BLOCK` requires updating the canvas attributes to match.
