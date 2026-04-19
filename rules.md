# BREACH — Project Rules

## Conversation style
- Keep responses brief. Diagnose → fix → summary.
- Show visual mockups BEFORE coding visual changes.
- When user gives a long wishlist, triage before coding.

## Build & delivery
- **Single HTML file** (`index.html`), no build step. Opens from `file://`.
- GitHub repo: `https://github.com/sumanprdas-alt/BREACH-GAME`
- **Pure Canvas 2D** — no Phaser, no physics engine.
- Assets embedded as base64. Audio synthesized via Web Audio API.
- Always iterate on previous version. Never rebuild from scratch.

## Engine architecture
- `requestAnimationFrame` loop with `ctx.drawImage()`
- Collision: `if(feetY >= groundY) { y = groundY - height }`
- Viewport: 512×288 internal, 2× CSS scale. GROUND_Y = 216.
- NO `source-atop` or composite operations.
- Screen shake: `screenShake` variable, decremented each frame, applied as `ctx.translate()` in render.

## Scene flow (LOCKED)
```
Splash → Difficulty Select → Character Select → Story Prelude → Stage Briefing → Play → GameOver/StageClear
```
- All pre-game screens are phases of the `title` scene (`titlePhase` variable)
- Splash: branding only, PRESS START
- Difficulty: EASY/NORMAL/HARD arrow select
- Character Select: 3 panels with stat bars (POWER/SPEED/ARMOR/RANGE), LEFT/RIGHT, ESC to go back
- Story Prelude: Star Wars crawl (scrolls bottom→top), auto-advances after blank pause
- Stage Briefing: typewriter text, shows operative + difficulty, ENTER to deploy
- BGM starts on title screen (War Drums)

## Sprite conventions

### Character sheets (3 playable)

| Character | Sheet | Size | Grid | Cell |
|-----------|-------|------|------|------|
| Kane | `hamza_sheet_v2` | 924×340 | 14×5 | 66×68 |
| Viper | `john_wick_sheet` | 924×340 | 14×5 | 66×68 |
| Ironside | `tiger_sheet_v2` | 924×340 | 14×5 | 66×68 |

- Frame N = row `Math.floor(N / 14)`, col `N % 14`
- All use 66×68 cells (code uses bill sheet for sprite preview currently)

### Frame map (shared)
```
idle: 5, run1: 1, run2: 3, aimUp: 12
prone: 31 (head LEFT in raw sprite)
flip: [28, 29, 30]
```

### Prone rules
- Frame 31 with drawY += 18 offset
- Facing RIGHT = flip sprite. Facing LEFT = raw.
- Hitbox: {y+28, h:12}. NO physics body changes.

### Enemy sprites
- Grunt sheet (36×40, 2 frames): clean transparency. Soldier, charger roles.
- Charger: red headband overlay. Sniper: yellow headband.
- NES turret sprites (35×32): from `nes_turret` asset, placed on ground AND platforms
- All enemies face player via `ctx.scale(-1,1)`.

### Enemy collision (LOCKED)
- ALL enemies (including fixed turrets/tanks) are SOLID — player cannot walk through
- Horizontal push-back collision when player overlaps vertically
- Touch damage on contact for all enemy types (fixed and mobile)
- Turrets only become solid after fully revealed/popped up

## Terrain (LOCKED)
- 21 multi-tier platforms organized into 20 sections
- 2-3 tier stacked platforms for vertical traversal (jumping up/down between levels)
- Close spacing (80-120px gaps), wider platforms (72-110px)
- Double-tier bridges where high and low platforms overlap

## Enemy spawning
- 31 fixed turrets/tanks across the level
- ~50% of turrets placed ON platforms at varied heights (start visible, no pop-up)
- Ground turrets pop up when player within 200px
- Platform turrets aim at player with Math.atan2 (shoot diagonally up/down)
- Wave interval: 2200ms, up to 3 mobile enemies simultaneously

## Fortress destruction (LOCKED)
- 3-second destruction sequence after boss core destroyed
- 12 cascading explosion SFX calls (200ms spacing + random jitter)
- 10 visual explosion bursts at 3.5× scale with orange flash
- Wall splits into 4 chunks that rotate and fall with physics after 1.5s
- 15 debris particles with gravity
- 2.5s screen shake (6px intensity)
- `fortressOpen = true` after 3s — wall disappears, player can walk through
- Player walks off right edge (+100px past LEVEL_W) to trigger Stage Clear
- Camera follows player past level boundary when fortress is open

## Logo (LOCKED)
- NES pixel art letterforms (10×12 grid per letter, 3px scale)
- Cream (#e8e2d4) letters with black drop shadow
- Red strikethrough (#d94236) at crossbar row (row 6)
- Center-aligned using `drawNESLogo(cx, cy, scale)`

## Sound (LOCKED — Hollywood v3)
- All Web Audio API synthesis. Zero external files.
- Core primitives: t(), sweep(), n(), nf(), kick(), dist(), d()
- Every SFX: 2-5 simultaneous layers
- **Weapon-specific sounds:** shoot() for basic/rapid, machineGun() for M, spreadShot() for S, fireball() for F
- Grenade throw: t(440) + n()
- Locked: Jump (150→400Hz), Death (NES staircase), Stage Clear (C5→C6 arpeggio)
- BGM: War Drums — kick/snare/hi-hat + bass riff, 0.12s/step, starts on title screen
- Master volume: 0.35

## Exception handling (CRITICAL)
- NEVER hardcode negative coordinates. Always: `y = groundY - spriteHeight`
- All coordinate math clamped with `Math.max(0, ...)`
- Validate sprite bounds before every `ctx.drawImage()`
- try/catch around all asset loading and rendering
- Death/respawn: gameTime-based only, NEVER setTimeout
- Array removals: iterate backwards
- Protect against division by zero

## Do / Don't

**DO:**
- Show mockups before coding visuals
- Use gameTime-based timers
- Keep colors in locked palette families
- Read cell height from character config
- Start BGM on title screen
- Reset titlePhase to 'splash' when returning to title

**DON'T:**
- Use Phaser or physics engines
- Use `source-atop` composites
- Use setTimeout for game logic
- Use brown in rock palette
- Put grass on walls/cliffs
- Make rocks from small tiles
- Hardcode negative coordinates
- Rebuild from scratch
