# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. No dependencies, no build tooling, no package.json — just static files served or opened directly.

## Running

There is no build/lint/test process. To run the game:

```bash
# Open directly
start index.html      # Windows

# Or serve locally (recommended, avoids any local-file quirks)
python3 -m http.server 8000
npx serve .
```

Then visit `http://localhost:8000` if using a server. Since there's no test suite, verify changes by playing the game in a browser (movement, rotation, line clears, game over, pause).

## Architecture

Three files, no modules/bundler — `index.html` loads `game.js` directly as a classic script, everything lives in global scope.

- **`index.html`** — DOM shell: the `#board` canvas (300×600, i.e. `COLS×BLOCK` by `ROWS×BLOCK`), the `#next-canvas` preview (120×120), HUD spans (`#score`, `#lines`, `#level`), and the `#overlay` used for both PAUSE and GAME OVER states.
- **`style.css`** — dark/retro arcade look; the `.overlay.hidden` class toggles overlay visibility.
- **`game.js`** — all game logic, single file, top-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) rather than a class/module.

### Core model

- **Board**: `ROWS × COLS` matrix (`createBoard`), each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: `PIECES` array of square matrices (index 0 unused/null so piece type == color index). Rotation is done by transpose+reverse in `rotateCW`, not by pre-defined rotation states.
- **Collision** (`collide`): checks board bounds and existing locked cells for a given shape/offset.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until one doesn't collide.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears full rows (shifting down, unshifting empty rows at top), then spawns the next piece. `spawn` triggers `endGame()` if the new piece immediately collides.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2` in `draw()`.
- **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time (`dropAccum`) and advances the piece one row once `dropInterval` is exceeded.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row. Level increments every 10 cleared lines; `dropInterval = max(100, 1000 - (level-1)*90)`.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS×BLOCK` and `ROWS×BLOCK`).

### Input

All keyboard handling is one `keydown` listener at the bottom of `game.js` (arrows to move/rotate/soft-drop, Space for hard drop, `P` to pause). `restart-btn` click calls `init()` to fully reset state.
