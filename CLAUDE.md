# CLAUDE.md — REKT Racer

## Project Overview

**REKT Racer** is a self-contained, single-file HTML5 browser racing game with a crypto/blockchain aesthetic. The entire application lives in `index.html` (~1238 lines, ~45 KB). There are no build tools, no package managers, no external dependencies beyond two Google Fonts loaded at runtime.

The game is a retro-style pseudo-3D racer (à la OutRun) rendered on an HTML5 Canvas, with synthesized audio via the Web Audio API and a mobile-friendly touch joystick.

---

## Repository Structure

```
P.DRIVE/
├── index.html      # The entire application — HTML + CSS + JS
└── CLAUDE.md       # This file
```

That's it. No `package.json`, no `Makefile`, no build artifacts.

---

## Running the Game

Open `index.html` in any modern browser (Chrome 51+, Firefox 54+, Safari 10+). No server required for local play — `file://` protocol works fine. For serving over a network, any static file server will do:

```bash
python3 -m http.server 8080
# or
npx serve .
```

---

## Architecture: `index.html` Layout

The file is divided into three contiguous sections:

### 1. HTML Structure (lines 1–287)

DOM elements used by the game:

| Element | Purpose |
|---|---|
| `#dashCanvas` | Top dashboard HUD (speed, level, coins, distance dials) |
| `#gameCanvas` | Main game viewport — all road/car rendering |
| `#wrapper` | Positions canvases relative to each other |
| `#overlay` | Full-screen modal for menu / death / win / upgrade screens |
| `#mobile-controls` | Touch joystick UI (hidden on pointer:fine devices) |
| `#controls-hint` | Keyboard hint bar (hidden on touch/small screens) |

### 2. CSS (lines 9–287, inside `<style>`)

- Primary accent color: `#FFD700` (gold)
- Background: `#000` (pure black)
- Fonts: `Orbitron` (display/UI), `Share Tech Mono` (monospace labels)
- Responsive font sizing uses CSS `clamp()`
- The overlay simulates CRT scanlines with a `repeating-linear-gradient`
- Canvas elements share a gold border with `box-shadow` glow

### 3. JavaScript Game Engine (lines 288–1238, inside `<script>`)

All game logic is a single flat script. No modules, no classes.

---

## Key Constants (lines 293–351)

```js
// Canvas sizing
const DASH_H = 48;                          // Dashboard height in px
const W = window.innerWidth;
const H = window.innerHeight - DASH_H;

// Road geometry
const ROAD_HALF  = 1.9;   // half-width of drivable road in world units
const DRAW_DIST  = 200;   // segments rendered ahead
const HORIZON    = 0.58;  // fraction of canvas height where horizon sits
const SCROLL_DIV = 22;    // controls apparent speed of scenery (lower = faster)

// Progression
const levelDistance = 12000;               // metres per level
const COINS_PER_UPGRADE = 20;              // coins required to unlock next car skin
const LEVEL_SPEEDS = [420, 600, 800, 1050, 1350, 1600, 1900, 2250, 2650, 3100];
// index = level-1; value = maxSpeed in pixels/second
```

---

## Game State Machine

Controlled by the `state` variable (string):

| Value | Description |
|---|---|
| `'menu'` | Title screen shown; game not running |
| `'playing'` | Active gameplay loop |
| `'dead'` | Player fell off road; crash screen |
| `'win'` | Level completed; transition screen |

State transitions:
- `menu` → `playing`: `startGame()`
- `playing` → `dead`: `showDead()` (player x exits `±ROAD_HALF`)
- `playing` → `win`: `showWin()` (distanceTravelled ≥ levelDistance)
- `dead`/`win` → `playing`: `startGame()` (restart/next level)

---

## Key Data Structures

### `player` object (line 354)
```js
let player = { x: 0, speed: 0, falling: false, fallY: 0, fallVY: 0 };
// x: lateral position in world units (clamped to ±ROAD_HALF when on road)
// speed: current forward speed in pixels/second
// falling: true while playing the off-road fall animation
// fallY/fallVY: vertical position/velocity during fall
```

### `segments` array (line 357)
```js
segments = [{ curve: <float>, coin: <bool> }, ...];
// ~120,000+ entries per level; built by buildRoad()
// curve: road curvature (positive = right-bending, negative = left-bending)
// coin: whether a collectible coin sits at this segment
```

### `CAR_SKINS` array (lines 320–341)
20 entries, each:
```js
{ body, roofA, roofB, stripe, hub, name }
// All color values are CSS hex strings
```

### `artStyles` array (lines 375–386)
10 visual themes (one per level, cycling):
```js
{ name, sky: [top, mid, bottom], road, verge: [light, dark], stripe, fog, stars: bool }
```

---

## Rendering Pipeline (`draw()`)

Called every animation frame in this order:

1. `drawSky(style)` — gradient background, optional procedural starfield
2. `drawRoad(style)` — perspective-projected road strips with verge, lane stripes, coins
3. `drawCar()` — player vehicle sprite at bottom-center
4. `drawFall()` — crash animation overlay (only when `player.falling`)
5. `drawParticles()` — coin pickup sparkles and explosion debris
6. Overlay rendering: handled separately via DOM (`#overlay`)
7. `drawDashboard()` — dials on `dashCanvas` (separate canvas element)

---

## Audio System (lines 573–576+)

Uses Web Audio API with synthesized sounds — no audio files:

- `initAudio()` — creates `AudioContext`, two oscillators + BiquadFilter for engine sound
- `updateEngineAudio(speed, maxSpeed)` — modulates oscillator frequencies and filter cutoff in real time based on current speed ratio
- `playCoinSound()` — short ascending tone on coin pickup
- `playExplosionSound()` — noise burst on crash

Audio context is created on first user interaction (browser autoplay policy requirement).

---

## Input Handling

### Keyboard
```js
const keys = {};
window.addEventListener('keydown', e => { keys[e.code] = true; e.preventDefault(); });
window.addEventListener('keyup',   e => { keys[e.code] = false; });
// Codes used: ArrowUp, ArrowDown, ArrowLeft, ArrowRight
```

### Touch Joystick (lines 395+)
- Left half of screen: joystick (steer + optional gas via Y axis)
- Right half: `#btn-gas` / `#btn-brake` buttons
- Shown automatically when `pointer: coarse` media query matches or viewport width ≤ 820px
- Orientation change triggers `location.reload()` to resize canvases cleanly

---

## Naming Conventions

| Pattern | Usage |
|---|---|
| `UPPER_SNAKE_CASE` | Constants (`ROAD_HALF`, `DRAW_DIST`, `LEVEL_SPEEDS`) |
| `camelCase` | Variables and functions (`roadOffset`, `distanceTravelled`) |
| `draw*` prefix | Rendering functions (`drawRoad`, `drawCar`, `drawDashboard`) |
| `show*` prefix | Game state transition functions (`showDead`, `showWin`, `showLevelUp`) |
| `spawn*` prefix | Particle emitter functions (`spawnParticles`, `spawnExplosion`) |
| `update*` prefix | Per-frame logic functions (`updateEngineAudio`, `updateExplosion`) |
| Hex color literals | All colors defined inline as `'#RRGGBB'` strings |

---

## Procedural Road Generation (`buildRoad()`, line 359)

Road is generated once per level into `segments[]`. Curvature cycles through 4 sinusoidal patterns in 35-segment phases:

```js
const phase = Math.floor(i / 35) % 4;
const curve = [
   Math.sin(i * 0.07) *  0.010,   // phase 0: gentle right
  -Math.sin(i * 0.05) *  0.012,   // phase 1: gentle left (wider)
   Math.sin(i * 0.09) *  0.008,   // phase 2: tight right
  -Math.sin(i * 0.06) *  0.014,   // phase 3: tight left
][phase];
```

Coins placed at every 17th segment after the first 20 (warm-up buffer).

---

## Progression System

- **Levels**: 10+ (cycles visual themes after level 10)
- **Level speed cap**: `LEVEL_SPEEDS[level - 1]`, capped at index 9 (3100 px/s)
- **Car skins**: 20 total; unlocked one per `COINS_PER_UPGRADE` (20) coins collected
- **Coin spawn rate**: every 17th road segment ≈ roughly every 5–6 metres of road

---

## Development Notes for AI Assistants

1. **Single-file constraint**: All changes happen in `index.html`. Do not split into separate `.js` or `.css` files unless explicitly asked — the single-file format is intentional for portability.

2. **No build step**: There is no transpilation or bundling. Any JavaScript must run natively in modern browsers. Avoid ES2022+ syntax that lacks broad browser support without checking `caniuse.com`.

3. **No tests**: There is no test harness. Manual play-testing in a browser is the only verification method.

4. **Canvas coordinate system**: `(0, 0)` is top-left of each canvas. `W×H` for `gameCanvas`, `DW×DH` (same width, 48px tall) for `dashCanvas`.

5. **Road offset**: `roadOffset` is a float index into `segments[]`, advancing each frame by `player.speed / SCROLL_DIV`. It represents the leading edge of the player's visible road.

6. **Segment indexing**: `Math.floor(roadOffset + i)` gives the segment index for the `i`-th strip ahead of the player. Bounds-check before accessing `segments[]`.

7. **Distance tracking**: `distanceTravelled` increments each frame by `player.speed * dt / 1000` (metres). Displayed as `km` on the dashboard dial.

8. **Particle arrays**: `particles` (coin sparkles) and `explosionParticles` each hold objects with `{x, y, vx, vy, life, maxLife, color}`. They are iterated and filtered every frame.

9. **Art style lookup**: `getStyle()` returns `artStyles[(level-1) % artStyles.length]`, so levels beyond 10 recycle themes.

10. **Touch vs. desktop detection**: `isTouchOrSmall` is computed once at startup and does not react to changes (except orientation change, which reloads the page).

---

## Deployment

Copy `index.html` to any static hosting service (GitHub Pages, Netlify, Cloudflare Pages, S3, etc.). No server-side logic is needed. The only external network requests are the two Google Fonts imports in the CSS `@import`.
