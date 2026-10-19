# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Asteroids clone in plain HTML5 Canvas + vanilla JavaScript (ES6+). No frameworks, no bundler, no dependencies, no build step, no tests, no linter. Code comments, UI strings, and the README are in Spanish — keep new comments and on-screen text in Spanish to match.

## Running

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .   # then visit http://localhost:3000
```

## Architecture

All game logic lives in a single file, `game.js`, loaded by `index.html` (which only provides an 800×600 `<canvas id="canvas">`). `W`/`H` in `game.js` are hardcoded to match the canvas size — change both together.

- **Entities** (`Bullet`, `Asteroid`, `Ship`, `Particle`) are classes that share an informal interface: `update(dt)`, `draw()`, and a `dead` flag. The main `update()` calls `update(dt)` on each collection, then removes dead entities with `.filter(e => !e.dead)`. All drawing uses the module-level `ctx`.
- **Toroidal space**: positions wrap via `wrap()` (except particles, which don't wrap).
- **Asteroid sizes** are integers 3 (large) → 1 (small), used as indices into the `RADII`, `SPEEDS`, and `POINTS` arrays (index 0 is unused). `split()` returns two asteroids of `size - 1`.
- **Game state** is held in module-level `let` globals (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`). `state` is `'playing' | 'dead' | 'gameover'`; `update()` branches on it. `initGame()` resets everything; `nextLevel()` triggers when `asteroids` is empty and spawns `3 + level` large asteroids away from the center (`SAFE_DIST`).
- **Input**: `keys[code]` holds held-key state (used for rotation/thrust); `pressed(code)` is a consume-once edge trigger backed by `justPressed` (used for shooting and restarting). Use `pressed()` for any new one-shot action.
- **Loop**: `requestAnimationFrame` loop with `dt` in seconds, clamped to 0.05s. All speeds/accelerations are expressed per second and multiplied by `dt`.
- **Collisions** are circle-vs-circle via `dist()`; ship collision uses `a.radius * 0.82` for leniency and is skipped while `ship.invincible > 0` (respawn invincibility, which also drives the blink in `Ship.draw()`).

## Note on README vs. code

The README mentions power-ups and special asteroid types (e.g. a "shooting star"). Only the **triple-shot** power-up is implemented (`PowerUp` class, `tripleTimer`, `powerUpSpawned` — drops exactly once per level when an asteroid is destroyed — randomly with `POWERUP_CHANCE`, or forced once ≤2 asteroids remain; `nextLevel()` resets `powerUpSpawned`, lasts `TRIPLE_DURATION` seconds). The **Nova Bomb** is a second `PowerUp` type (`type: 'nova'`): rare drop (`NOVA_CHANCE`, at most once per level via `novaSpawned`, only while `novaBombs === 0`), stored as one charge in `novaBombs`, fired with `pressed('KeyB')` → `detonateNova()` destroys every asteroid without splitting (awarding points), which immediately triggers `nextLevel()`. The rest are not implemented yet.
