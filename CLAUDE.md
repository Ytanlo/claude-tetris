# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JavaScript Tetris (HTML5 Canvas + CSS). No dependencies, no build step, no package manager, no tests, no linter. The README and UI text are in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python3 -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

All game logic lives in a single script, `game.js` (loaded by `index.html` with a plain `<script>` tag, strict mode, no modules). It relies on fixed element IDs in `index.html` (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`, `theme-toggle`, `theme-icon`, `theme-label`) — renaming one requires updating both files.

- **State** is a set of module-level `let` globals (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, ...) reset in `init()`. `board` is a 20×10 matrix of color indices (0 = empty, 1–7 index into `COLORS`/`PIECES`).
- **Pieces** are square matrices of color indices (`PIECES`); rotation is matrix rotation (`rotateCW`) with simple horizontal wall kicks `[0,-1,1,-2,2]` in `tryRotate` (no SRS, no bag randomizer — `randomPiece` is uniform random).
- **Game loop**: `loop(ts)` runs on `requestAnimationFrame`, accumulates `dropAccum` against `dropInterval` for gravity, and redraws every frame. Pause/game over stop it via `cancelAnimationFrame(animId)`; unpausing restarts it with a fresh `lastTime`.
- **Piece lifecycle**: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next`, rolls a new one, and calls `endGame()` if the new piece collides immediately. `clearLines()` also owns scoring/level/speed (`LINE_SCORES[n] * level`; level = lines/10 + 1; `dropInterval = max(100, 1000 - (level-1)*90)`). Soft drop adds 1 pt/cell, hard drop 2 pts/cell.
- **Rendering** is split across `draw()` (board, ghost via `ghostY()`, current piece) and `drawNext()` (preview canvas); both use `drawBlock`. Canvas size (300×600) must stay consistent with `COLS`/`ROWS`/`BLOCK` (10/20/30).
- **Input**: one `keydown` listener (arrows, `X` rotate, `Space` hard drop, `P` pause, `T` theme). Restart is the overlay button calling `init()`.
- **Theming**: colors are CSS variables in `style.css`, switched by `data-theme` (`dark` default / `light`) on `<html>`. An inline script in `index.html` applies the saved theme (`localStorage` key `tetris-theme`) before first paint; `applyTheme()` in `game.js` updates the button, caches the canvas grid color from `--grid`, and persists. The `T` key and `#theme-toggle` call `toggleTheme()`.
