# BREACH — Project Rules

## Conversation style
- Keep responses brief. Diagnose → fix → summary.
- Show visual mockups BEFORE coding visual changes.
- When user gives a long wishlist, triage before coding.

## Build & delivery
- **Single HTML file**, no build step. Opens from `file://`.
- Latest file: `/mnt/user-data/outputs/breach.html`
- **Pure Canvas 2D** — no Phaser, no physics engine.
- Assets embedded as base64.
- Always iterate on previous version. Never rebuild from scratch.

## Engine architecture
- `requestAnimationFrame` loop with `ctx.drawImage()`
- Collision: `if(feetY >= groundY) { y = groundY - height }`
- Viewport: 512×288 internal, 2× CSS scale. GROUND_Y = 216.
- NO `source-atop` or composite operations.

## Sprite conventions

### Character sheets (3 playable)

| Character | Sheet | Size | Grid | Cell |
|-----------|-------|------|------|------|
| Kane | `hamza_sheet_v2` | 924×340 | 14×5 | 66×68 |
| Viper | `john_wick_sheet` | 924×340 | 14×5 | 66×68 |
| Ironside | `tiger_sheet_v2` | 924×340 | 14×5 | 66×68 |

- Frame N = row `Math.floor(N / 14)`, col `N % 14`
- All use 66×68 cells. Code must use character config for cell dimensions.

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
- Grunt sheet (36×40, 2 frames): clean transparency. Soldier, charger, sniper roles.
- Charger: red headband overlay. Sniper: yellow headband.
- All enemies face player via `ctx.scale(-1,1)`.
- Turrets sit IN rock wall, below grass line. Pop-up animation clipped to above-ground.

## Visual design (LOCKED)

### Rocks
- ONE large 256×128 texture. Three layers: body fill → background scatter → staggered polygons.
- 8-10 vertex polygons + pixel roughness (40% edge scatter, 8% body noise).
- Palette: specular(225,205,50) → base(45,42,5). Zero gaps.

### Grass
- 10-12px thick on platform tops ONLY. 3 bands: #88D048, #38A038, #184818.
- Serrated top (4-7px) + draping bottom (3-6px). No gradients.

### Background void
- Dark crosshatch: base(2,3,0), stripes(22,28,9).

### Fern fringe
- Top of screen ONLY. Must NOT overlap play area or characters.

### Logo
- BREACH in angular stencil, steel (#d8d0c0), gold edge (#fcd848).
- Red line (#e03828) through B, R, C, H. NOT through E or A.

## Exception handling (CRITICAL)

### Coordinate safety
- NEVER hardcode negative coordinates. Always: `y = groundY - spriteHeight`
- All math clamped: `Math.max(0, Math.min(value, BOUNDARY))`
- Validate before `ctx.drawImage()`: x≥0, y≥0, w>0, h>0, source within sheet

### Asset loading
- try/catch around ALL loading and rendering calls
- If image fails, use fallback colored rectangle
- Check `.complete` and `.naturalWidth > 0` before drawing

### Game logic
- Death/respawn: gameTime-based only, NEVER setTimeout
- Array removals: iterate backwards
- Collision: validate both objects exist and alive
- Protect against division by zero

## Sound (LOCKED — Hollywood v3)

- All Web Audio API synthesis. Zero external files.
- Core primitives: tone, sweep, noise, filtered noise, kick, distortion (waveshaper)
- Every SFX: 2-5 simultaneous layers
- Locked: Jump (150→400Hz), Death (NES staircase), Stage Clear (C5→C6 arpeggio)
- BGM: War Drums — kick/snare/hi-hat + bass riff, 0.12s/step
- Sound test: `/mnt/user-data/outputs/sound-test-v3.html`

## Do / Don't

**DO:**
- Show mockups before coding visuals
- Use gameTime-based timers
- Keep colors in locked palette families
- Read cell height from character config

**DON'T:**
- Use Phaser or physics engines
- Use `source-atop` composites
- Use setTimeout for game logic
- Use brown in rock palette
- Put grass on walls/cliffs
- Make rocks from small tiles
- Hardcode negative coordinates
- Rebuild from scratch
