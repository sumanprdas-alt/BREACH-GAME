# BREACH — Project Handoff

Paste this into a new chat to continue the project.

---

I'm continuing work on **BREACH** — a Contra-style 2D run-and-gun web game built as a single HTML file with pure Canvas 2D (no Phaser) and base64-embedded NES Contra sprites. The game is at `/mnt/user-data/outputs/breach.html` and also committed to GitHub at `https://github.com/sumanprdas-alt/BREACH-GAME` as `index.html`.

## Game identity
- **Name:** BREACH
- **Story:** Year 2089. Crimson Dawn syndicate built Project Starkfire — superweapon that kills every power grid on Earth. Three operatives deployed to Archipelago Sigma. No backup.
- **Characters:** Kane (Commander), Viper (Ghost), Ironside (Wall) — player selects at character select screen
- **Logo:** NES pixel art letterforms, cream (#e8e2d4) with black shadow, red strikethrough (#d94236), center-aligned

## Engine
Pure Canvas 2D + requestAnimationFrame. No Phaser, no physics engine.
Viewport: 512×288 at 2× CSS scale. GROUND_Y=216.

## What's built
- **5-screen title flow:** Splash → Difficulty Select → Character Select (with stat bars) → Story Prelude (Star Wars scroll, auto-advance) → Stage 1 Briefing (typewriter) → Play
- **3 playable characters** (all 924×340, 14×5 grid, 66×68 cells, shared frame map)
  - Kane: POW:4 SPD:3 ARM:4 RNG:3
  - Viper: POW:3 SPD:5 ARM:2 RNG:4
  - Ironside: POW:5 SPD:2 ARM:5 RNG:3
- Movement: speed 1.6, jump -7.2, gravity 0.6, coyote 90ms, jump buffer 120ms
- Prone: frame 31, head toward facing, legs opposite. Hitbox {y+28, h:12}
- Weapons: basic, M(machineGun sound), S(spreadShot sound), R(shoot sound), F(fireball sound)
- Grenades: C key, 1.2s fuse, 60px blast, stock of 3
- Drop-through platforms: DOWN+SPACE
- Stage 1: 6400px, 3 zones, 21 multi-tier platforms, 31 fixed turrets/tanks (many on platforms)
- Wave spawner: 2200ms interval, up to 3 mobile enemies simultaneously
- **Solid enemies:** Player cannot walk through turrets or enemies — horizontal push-back collision + contact damage for ALL enemies including fixed turrets
- **Platform turrets:** Turrets placed on platforms at varied heights, start already visible, shoot diagonally toward player using Math.atan2
- Death: gameTime-based timer, clear all enemies on respawn, 2.5s invuln
- **Massive fortress destruction:** 12 cascading explosions with sfx, wall splits into 4 rotating/falling chunks, 15 debris particles, 2.5s screen shake, orange flash effects, 3s total sequence
- **Level exit:** After fortress crumbles (3s), player walks through gap and off right edge (+100px past LEVEL_W) to trigger Stage Clear
- **Audio:** Hollywood v3 SFX (weapon-specific sounds for each weapon type), War Drums BGM (starts on title screen), master volume 0.35
- **Browser notice:** "OPTIMISED FOR WEB BROWSERS · TRUE TO RETRO" at bottom of splash screen

## 5 Stages
1. Jungle Breach (built) → The Gate boss
2. The Corridor (planned) → Sentinel mech boss
3. The Cascade (planned) → Hydra Battery boss
4. The Furnace (planned) → Forge Master boss
5. The Core (planned) → Starkfire boss

## Key files
- Game: `index.html` (373KB, single file, zero dependencies)
- GitHub: `https://github.com/sumanprdas-alt/BREACH-GAME`
- Game spec: `game-spec.md`
- Story bible: `story-bible.md`
- Sound test: `/mnt/user-data/outputs/sound-test-v3.html`
- Character sheets: Kane (`hamza_sheet_v2`), Viper (`john_wick_sheet`), Ironside (`tiger_sheet_v2`) in `/mnt/user-data/uploads/`

## Exception handling (CRITICAL)
- NEVER hardcode negative coordinates. Compute: `y = groundY - spriteHeight`
- All coords clamped: `Math.max(0, Math.min(value, BOUNDARY))`
- Validate sprite bounds before every `ctx.drawImage()`
- try/catch around all asset loading and rendering
- Death/respawn: gameTime-based only, NEVER setTimeout
- Array removals: iterate backwards

## Conventions
- Single HTML file, no build step
- Pure Canvas 2D, no composite operations
- Show visual mockups BEFORE coding
- Keep responses brief
