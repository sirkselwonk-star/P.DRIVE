# REKT Racer

```
██████╗ ███████╗██╗  ██╗████████╗    ██████╗  █████╗  ██████╗███████╗██████╗
██╔══██╗██╔════╝██║ ██╔╝╚══██╔══╝    ██╔══██╗██╔══██╗██╔════╝██╔════╝██╔══██╗
██████╔╝█████╗  █████╔╝    ██║       ██████╔╝███████║██║     █████╗  ██████╔╝
██╔══██╗██╔══╝  ██╔═██╗    ██║       ██╔══██╗██╔══██║██║     ██╔══╝  ██╔══██╗
██║  ██║███████╗██║  ██╗   ██║       ██║  ██║██║  ██║╚██████╗███████╗██║  ██║
╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝   ╚═╝       ╚═╝  ╚═╝╚═╝  ╚═╝ ╚═════╝╚══════╝╚═╝  ╚═╝
```

> **A retro pseudo-3D racing game with a crypto/blockchain aesthetic — delivered as a single, self-contained HTML file.**

---

## Overview

REKT Racer is an OutRun-style pseudo-3D racer built entirely with vanilla HTML5, CSS, and JavaScript — no frameworks, no build tools, no package managers. Drop the file in a browser and you're racing in seconds.

Speed through 10 visually distinct levels — each one themed after a crypto market moment — collecting coins, unlocking car skins, and pushing your car to the limit before you get **REKT**.

---

## Features

- **Zero dependencies** — one `index.html` file (~45 KB), runs offline via `file://`
- **Pseudo-3D road rendering** on an HTML5 Canvas (OutRun-style perspective projection)
- **Synthesized audio** — engine roar, coin pings, and explosion crunch via the Web Audio API (no audio files)
- **10 crypto-themed level palettes** cycling through visual environments
- **20 unlockable car skins** earned by collecting coins on the road
- **Mobile-friendly** with a virtual touch joystick and gas/brake buttons
- **Responsive** — fills the viewport on any screen size

---

## Level Themes

| Level | Theme | Vibe |
|-------|-------|------|
| 1 | GENESIS BLOCK | Pitch black, gold stripes, starfield |
| 2 | BULL RUN | Deep amber tones, orange lane markers |
| 3 | ALTCOIN | Midnight blue, neon cyan |
| 4 | FLASH CRASH | Blood red, emergency orange |
| 5 | SYNTHWAVE | Purple gradient, hot magenta |
| 6 | CRYPTO GREEN | Matrix terminal green |
| 7 | BEAR MARKET | Dark violet, eerie purple |
| 8 | LIQUIDATION | Scorched orange-black |
| 9 | ICE WALLET | Cold steel blue |
| 10 | MOON SHOT | Void black, pure white, stars |

Levels beyond 10 recycle the palette cycle, with speed caps increasing indefinitely.

---

## Car Skins

Earn **20 coins** to unlock the next skin. 20 skins total:

`GOLD STANDARD` · `MIDNIGHT BLUE` · `CRIMSON DEVIL` · `VENOM GREEN` · `ULTRAVIOLET` · `ARCTIC WHITE` · `INFERNO ORANGE` · `NEON CYAN` · `SAND STORM` · `HOT PINK` · `GALAXY SILVER` · `ACID LIME` · `LAVA FLOW` · `DEEP TEAL` · `CANDY MAGENTA` · `NUCLEAR YELLOW` · `MOLTEN CORE` · `EMERALD RUSH` · `ROYAL PURPLE` · `STEALTH CHROME`

---

## Controls

### Keyboard

| Key | Action |
|-----|--------|
| `↑` Arrow Up | Accelerate |
| `↓` Arrow Down | Brake |
| `←` Arrow Left | Steer left |
| `→` Arrow Right | Steer right |

### Touch (Mobile)

| Control | Action |
|---------|--------|
| Left-side joystick (X-axis) | Steer |
| `GAS` button (right side) | Accelerate |
| `BRK` button (right side) | Brake |

The touch controls appear automatically on phones and tablets (or any viewport ≤ 820px).

---

## Gameplay

- **Steer** your car and stay on the road — drive off the edge and you're **REKT**
- **Collect coins** by driving over them on the road surface
- **Complete a level** by covering 12,000 metres before you crash
- **Speed caps** increase each level (from 420 px/s on Level 1 up to 3100 px/s on Level 10+)
- After each level you can **upgrade** your car skin if you've collected enough coins

---

## Getting Started

### Play Instantly

Download `index.html` and open it in your browser — that's it.

```bash
# Or serve it locally
python3 -m http.server 8080
```

Then open `http://localhost:8080` in Chrome, Firefox, or Safari.

### Browser Support

| Browser | Minimum Version |
|---------|----------------|
| Chrome | 51+ |
| Firefox | 54+ |
| Safari | 10+ |

---

## Architecture

The entire application is a single `index.html` file (~1,238 lines):

```
index.html
├── <style>         CSS — layout, overlay, animations, responsive sizing
├── <body>          Two <canvas> elements + overlay modal + touch controls
└── <script>        All game logic — no modules, runs natively in the browser
```

### Key Technical Details

| Aspect | Detail |
|--------|--------|
| Road rendering | Perspective-projected strip segments (200 strips per frame) |
| Road generation | Procedural sinusoidal curvature, built once per level |
| Audio | Two oscillators + BiquadFilter modulated by speed ratio |
| Particles | Coin sparkles + explosion debris, iterated and filtered per frame |
| State machine | `menu` → `playing` → `dead` / `win` → `playing` |
| Distance tracking | `distanceTravelled` increments by `speed × dt / 1000` metres per frame |

---

## Deployment

No server-side logic required. Copy `index.html` to any static host:

- GitHub Pages
- Netlify
- Cloudflare Pages
- AWS S3 + CloudFront
- Any CDN

The only outbound network requests are two **Google Fonts** (`Orbitron` + `Share Tech Mono`) loaded via CSS `@import`. The game runs fully offline if those fonts are unavailable — system fallbacks apply.

---

## Project Structure

```
P.DRIVE/
├── index.html      # The entire game
└── README.md       # This file
```

---

*Don't get REKT.*
