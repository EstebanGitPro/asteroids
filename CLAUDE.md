# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Asteroids** is a faithful HTML5 Canvas implementation of the classic arcade game, built with pure vanilla JavaScript and no dependencies or build tools.

- **Single file**: All game logic in `game.js` (~424 lines)
- **No bundler/framework**: Runs directly in the browser from `index.html`
- **60 FPS target**: Uses `requestAnimationFrame` for smooth gameplay
- **Canvas 2D**: All rendering via native HTML5 Canvas API

## Running the Game

**Simplest way** — double-click `index.html` to open in your default browser.

**With a local server:**
```bash
npx serve .
```
Then visit `http://localhost:3000`.

## Codebase Structure

### Entity Architecture

The game uses a consistent entity pattern with three phases per frame:

```
constructor(...)     → Initialize state
update(dt)           → Physics, input, state changes (dt = deltaTime in seconds)
draw()               → Render to canvas
```

Each entity stores its own `dead` flag; dead entities are filtered out after update.

**Entity types** (in game.js):
- `Ship` — Player-controlled vessel with invincibility cooldown and shoot cooldown
- `Bullet` — Projectiles with 1.1s lifetime (time-to-live)
- `Asteroid` — Randomly-generated irregular polygons; split into two smaller ones when destroyed
- `Particle` — Explosion debris with fade-out alpha

### Game State

Global state (lines 238–242):
- `ship`, `bullets`, `asteroids`, `particles` — Entity arrays
- `score`, `lives`, `level` — Game counters
- `state` — One of `'playing'`, `'dead'` (waiting 2s for respawn), `'gameover'`

State transitions:
- `'playing'` → `'dead'` when ship hit by asteroid (unless invincible)
- `'dead'` → `'playing'` after 2s timer
- `'playing'` → `'gameover'` when lives reach 0
- `'gameover'` → `'playing'` on Space keypress

### Update Loop

`update(dt)` runs game logic:
1. Handle state transitions (`gameover`, `dead`, `playing`)
2. Process input (Arrow keys, Space)
3. Update all entities
4. Collision detection (bullet vs asteroid, ship vs asteroid)
5. Check win condition (no asteroids left → next level)

### Rendering

`draw()` follows this order (back-to-front):
1. Clear canvas black
2. Draw particles (explosions)
3. Draw asteroids
4. Draw bullets
5. Draw ship (with blinking during invincibility)
6. Draw HUD (score, level, lives)
7. Draw overlay (GAME OVER message)

## Important Constants & Tuning

Located in respective class constructors:

**Ship (line 122–136)**
- `ROT = 3.5` — rotation speed (rad/s)
- `THRUST = 260` — acceleration (px/s²)
- `DRAG = 0.987` — velocity damping per frame
- `invincible = 3` — respawn invincibility duration (seconds)
- `shootCooldown = 0.2` — delay between shots (seconds)

**Asteroid (line 61–63)**
- `RADII[size]` — radius by size (1, 2, 3)
- `SPEEDS[size]` — base velocity by size
- `POINTS[size]` — score awarded by size
- `rotSpeed` — random rotation speed (rad/s)
- Polygon generation: 8–13 random vertices per asteroid (line 81)

**Bullet (line 33–42)**
- `SPEED = 520` — velocity magnitude (px/s)
- `ttl = 1.1` — lifetime before disappearing (seconds)

**Spawn mechanics (line 244–254)**
- `SAFE_DIST = 130` — minimum distance from ship center when spawning asteroids
- `spawnAsteroids(4)` at game start; `spawnAsteroids(3 + level)` per level

## Making Common Changes

### Adjust difficulty

- Increase initial asteroid count in `initGame()` (line 265)
- Increase spawn count per level in `nextLevel()` (line 273)
- Tweak `SPEEDS[size]` for asteroid velocity (line 62)

### Change visual appearance

- Ship silhouette: edit `ctx.moveTo/lineTo` calls (lines 184–189)
- Asteroid polygon vertices: adjust `randInt(8, 13)` range (line 81)
- Thruster flame: modify random check or color (lines 193–200)
- HUD font/color: edit `drawHUD()` (lines 371–383)

### Adjust shooting/collision

- Bullet speed: change `SPEED = 520` (line 37)
- Shot cooldown: change `shootCooldown = 0.2` (line 164)
- Ship hit radius: change `ship.radius + a.radius * 0.82` (line 342)
- Bullet vs asteroid detection: change distance comparison (line 327)

### Add features

- Lives per extra level: modify `spawnAsteroids()` count formula
- Sound effects: use Web Audio API (no external library needed)
- Power-ups: add new entity type with constructor/update/draw, spawn randomly
- Score multipliers: add conditional logic to `POINTS[a.size]` calculation

## Input Handling

Two-stage key tracking (lines 9–24):
- `keys[code]` — current frame state (true = held down)
- `justPressed[code]` — one-time per-press flag (consumed by `pressed()` call)

```javascript
if (keys['ArrowLeft']) { /* held */ }      // repeats every frame while held
if (pressed('Space')) { /* one-time */ }   // fires once per press
```

Key codes used: `'ArrowLeft'`, `'ArrowRight'`, `'ArrowUp'`, `'Space'`.

## Coordinate & Physics Notes

- **Canvas origin** (0,0) = top-left; x → right, y → down
- **Wrapping**: `wrap(v, max)` handles toroidal space (ship/bullets/asteroids wrap at edges)
- **Distance**: `dist(a, b)` = Euclidean distance between two entities
- **Angle**: 0 = right, π/2 = down, π = left, 3π/2 = up; `Math.atan2(dy, dx)` for angle from delta

## Comments & Code Style

- Code is sparsely commented (intentional — implementation is self-documenting)
- Class names are CamelCase; instance/global variables are camelCase
- Game parameters are SCREAMING_SNAKE_CASE
- Sections are marked with `// ── Label ─` for navigation

## Performance Considerations

- Entity arrays are filtered to remove dead objects each frame (lines 320–337)
- Collision detection is O(n²) for bullets vs asteroids (acceptable for small counts)
- No image loading or external assets — pure canvas drawing
- Canvas context state saved/restored per draw call (`ctx.save()`/`ctx.restore()`) to avoid state bleeding

## Testing the Game

Manual testing checklist:
1. **Controls** — arrow keys rotate, up thrusts, space shoots
2. **Collisions** — bullets destroy asteroids; asteroids kill ship
3. **Spawning** — asteroids split into 2 smaller ones; new level spawns more
4. **Lives/Score** — lives decrement on hit; score increases correctly
5. **Invincibility** — ship flickers and is immune after respawn
6. **Wrapping** — objects disappear at edge and reappear opposite side
7. **Game Over** — space key restarts after game over

## Tools & Workflow

- **No build step required** — edit, save, refresh browser
- **No linter/formatter** — maintain existing style (see Code Style above)
- **Browser DevTools** — console for debugging; check for rendering performance in Performance tab
- **Source file** is small enough to read entirely in one sitting (424 lines)