# BREACH — Project Handoff

Paste this into a new chat to continue the project.

---

I'm continuing work on **BREACH** — a Contra-style 2D run-and-gun web game built as a single HTML file with pure Canvas 2D (no Phaser) and base64-embedded NES Contra sprites. The latest version is at `/mnt/user-data/outputs/contra-strike.html` (being renamed to `breach.html`).

## Game identity
- **Name:** BREACH
- **Story:** Year 2089. Crimson Dawn syndicate has built Project Starkfire — a superweapon that kills every power grid on Earth with a single plasma pulse. Three operatives deployed to Archipelago Sigma. No backup.
- **Characters:** Kane (Commander), Viper (Ghost), Ironside (Wall) — player selects at title screen
- **Logo:** Bold angular stencil letters, steel/gold, red fracture line through B,R,C,H only (not E,A)

## Engine
Pure Canvas 2D + requestAnimationFrame. No Phaser, no physics engine.
Viewport: 512×288 at 2× CSS scale. GROUND_Y=216.

## What's built
- 3 playable characters (all 924×340, 14×5 grid, 66×68 cells, shared frame map)
- Movement: speed 1.6, jump -7.2, gravity 0.6, coyote 90ms, jump buffer 120ms
- Prone: frame 31, head toward facing, legs opposite. Hitbox {y+28, h:12}.
- Weapons: basic, M, S, R, F
- Grenades: C key, 1.2s fuse, 60px blast, stock of 3
- Drop-through platforms: DOWN+SPACE
- Stage 1: 6400px, 3 zones, 16 pop-up turrets, 2 tanks, fortress boss
- Enemy roster: soldier, charger, sniper (grunt_sheet), turrets, tanks, aliens, crawlers, dogra
- Death: gameTime-based timer, clear all enemies on respawn, 2.5s invuln
- Audio: Hollywood v3 SFX (multi-layered), War Drums BGM, all Web Audio synthesis

## Visual overhaul LOCKED — needs coding into game
- Rocks: 256×128 texture, 3-layer fill, rough polygons + pixel noise. Yellow-green palette.
- Grass: 10-12px thick, 3 bands, serrated top + draping bottom.
- Background: dark crosshatch void.
- Fern fringe: top of screen only, short.

## 5 Stages
1. Jungle Breach (built) → The Gate boss
2. The Corridor (planned) → Sentinel mech boss
3. The Cascade (planned) → Hydra Battery boss
4. The Furnace (planned) → Forge Master boss
5. The Core (planned) → Starkfire boss

## Key files
- Game: `/mnt/user-data/outputs/contra-strike.html` → renaming to `breach.html`
- Game spec: `/mnt/user-data/outputs/game-spec.md`
- Story bible: `/mnt/user-data/outputs/story-bible.md`
- Sound test: `/mnt/user-data/outputs/sound-test-v3.html`
- Character sheets:
  - Kane: `/mnt/user-data/uploads/hamza_sheet_v2.png`
  - Viper: `/mnt/user-data/uploads/john_wick_sheet.png`
  - Ironside: `/mnt/user-data/uploads/tiger_sheet_v2.png`
- Enemy/turret sheets in `/mnt/user-data/uploads/`
- Assets: `/home/claude/assets_b64.json`, `/home/claude/new_assets_b64.json`

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
