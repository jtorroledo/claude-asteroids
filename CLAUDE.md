# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A clone of the classic arcade game **Asteroids**, implemented in pure HTML5 Canvas with vanilla JavaScript (ES6+) — no frameworks, no bundler, no dependencies, no build step, no test suite. The entire game logic lives in a single file, `game.js` (~420 lines).

## Running the game

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

Then visit `http://localhost:3000`. There is no build, lint, or test command — changes to `game.js` take effect on page reload.

## Architecture

Everything is in `game.js`, structured top-to-bottom as:

1. **Input** — `keys`/`justPressed` maps populated by `keydown`/`keyup` listeners; `pressed(code)` consumes a one-shot press (used for firing and restart).
2. **Utils** — `wrap` (toroidal edge wrapping), `dist`, `rand`, `randInt`.
3. **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has `update(dt)` and `draw()`. Entities mark themselves `dead = true` rather than removing themselves; arrays are filtered afterward.
4. **Global mutable game state** — `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` (`'playing' | 'dead' | 'gameover'`), `deadTimer`. Reset via `initGame()`, advanced via `nextLevel()`.
5. **`update(dt)`** — branches on `state` first. Otherwise: reads input, updates all entities, filters dead ones, then runs collision detection (bullet↔asteroid, ship↔asteroid) using simple circle distance checks against each entity's `radius`. Asteroid splitting (`Asteroid.split()`) and level completion (`asteroids.length === 0` → `nextLevel()`) happen here.
6. **`draw()`** / HUD — clears canvas, draws entities back-to-front (particles, asteroids, bullets, ship), then HUD (score/level/lives) and any overlay (game over).
7. **Main loop** — `requestAnimationFrame` loop computing `dt` (clamped to 50ms) and calling `update(dt)` then `draw()`.

Key gameplay constants (asteroid sizes/speeds/points, ship rotation/thrust/drag, bullet speed/TTL, invincibility duration) are defined near the top of each relevant section — adjust these directly rather than adding configuration layers.

The game world is toroidal: all positions wrap via `wrap(v, max)` against canvas width `W` (800) and height `H` (600), defined at the top of `game.js`.

When making changes, keep everything in `game.js` unless there's a strong reason to split files — the project's design intentionally favors a single flat file with no dependencies.
