# BREACH — Project Handoff

Paste this into a new chat to continue the project.

---

I'm continuing work on **BREACH** — a Contra-style 2D run-and-gun web game built as a single HTML file with pure Canvas 2D (no Phaser) and base64-embedded NES Contra sprites.

## Game identity
- **Name:** BREACH
- **Story:** Year 2089. Crimson Dawn syndicate built Project Starkfire — superweapon that kills every power grid on Earth. Three operatives deployed to Archipelago Sigma. No backup.
- **Characters:** Kane (Commander), Viper (Ghost), Ironside (Wall) — player selects at character select screen
- **Logo:** NES pixel art letterforms, cream (#e8e2d4) with black shadow, red strikethrough (#d94236), center-aligned

## Where the files are
- **Production build:** GitHub `https://github.com/sumanprdas-alt/BREACH-GAME` → `index.html` (commit `5da0d90`)
  - `currentStage=1` — plays Stage 1. Stage 2 code is present but gated, never executes.
- **Stage 2 WIP:** `breach-s2-wip.html` (local/outputs only, NOT in repo)
  - `currentStage=2` — jumps straight to Stage 2 for testing.
  - Identical codebase to production, only the `currentStage` variable differs.
- **PAT:** (stored separately — not in repo)
- **Stage 2 design doc:** `stage-2-design.md` (in project files)
- **Stage 2 rationale doc:** `stage-2-rationale.md` (in outputs) — explains WHY every number was chosen
- **Character sheets:** Kane=`hamza_sheet_v2.png`, Viper=`john_wick_sheet.png`, Ironside=`tiger_sheet_v2.png` in `/mnt/user-data/uploads/`
- **NES sprite sheets:** Defense_Wall, Dogra boss, Enemies_Obstacles, Final_Gate, Turrets, Weapon_Pickups in `/mnt/project/`

## Two-file workflow (IMPORTANT)
Both files contain the SAME codebase — both stages, all enemies, all logic. The ONLY difference is one variable:

```javascript
// Production (index.html in repo):
let currentStage = 1;

// Stage 2 WIP (breach-s2-wip.html, local only):
let currentStage = 2; // WIP — Stage 2 testing. Set to 1 for production.
```

**How to work:**
1. Always edit the WIP file (`breach-s2-wip.html`) during Stage 2 development
2. Test Stage 2 features with `currentStage=2`
3. When ready to commit: copy WIP → `index.html`, change `currentStage` to `1`, commit
4. The game flows naturally: player beats Stage 1 → `currentStage` flips to `2` → Stage 2 briefing → corridor gameplay
5. All stage-gating is in place — Stage 1 code only runs when `currentStage===1`, Stage 2 only when `currentStage===2`

**NEVER commit with `currentStage=2`** — production must always start at Stage 1.

## Engine (LOCKED)
- Pure Canvas 2D + requestAnimationFrame — NO Phaser
- Single HTML file, no build step, no external dependencies
- All assets embedded as base64, all audio synthesized via Web Audio API
- Viewport: 512×288 at 2× CSS scale (1024×576). GROUND_Y = 216

## Controls (LOCKED)
| Key | Action |
|-----|--------|
| ← → | Move / Aim |
| UP | Jump + Aim Up (dual function) |
| SPACE / X | Shoot (both keys work) |
| DOWN | Prone |
| DOWN + UP | Drop Through Platform |
| C | Grenade |
| H | Field Manual (help overlay, pauses game) |
| ENTER | Advance / Confirm |

## Characters (3 playable, LOCKED)
All share same frame map on 924×340 sheet, 14×5 grid, 66×68 cells, CHAR_SCALE=1.3.

| # | Codename | Stats |
|---|----------|-------|
| 1 | Kane | POW:4 SPD:3 ARM:4 RNG:3 |
| 2 | Viper | POW:3 SPD:5 ARM:2 RNG:4 |
| 3 | Ironside | POW:5 SPD:2 ARM:5 RNG:3 |

Frame map: idle(5), run(1,3), aimUp(12), prone(31), flip(28-30 somersault at 80ms).
Prone: frame 31, drawY+=18 offset, hitbox {y+28,h:12}, flipX=bill.facing<0 (same as standing).

## Difficulty (REBALANCED — tuned for 60% novice / 30% amateur / 10% expert)

| Property | Easy | Normal | Hard |
|----------|------|--------|------|
| HP | 9 | 6 | 3 |
| Lives | 5 | 4 | 2 |
| Shot interval (ms) | 4500 | 3000 | 1800 |
| Invulnerability (ms) | 4000 | 2800 | 1500 |
| Bullet speed multiplier | 0.35 | 0.55 | 0.9 |
| Enemy speed multiplier | 0.5 | 0.75 | 1.1 |
| Enemy density | 0.35 | 0.6 | 1.0 |
| Crawler speed | 0.6 | 0.9 | 1.2 |
| Laser off bonus (ms) | 800 | 400 | 0 |
| Steam cycle bonus (ms) | 1200 | 500 | 0 |
| Blackout crawlers | 3 | 5 | 7 |

All difficulty properties are WIRED IN and functional. Earlier builds had `crawlerSpeed`, `laserOffBonus` defined but never used — this is now fixed.

## Scene Flow (LOCKED)
```
Splash → Difficulty → Character Select → Story Prelude → Stage Briefing → Play → GameOver/StageClear
```

## Stage 1: JUNGLE BREACH (BUILT & LOCKED)
- 6400px level, FORTRESS_X=6100, 3 zones (jungle/riverbank/fortress)
- 21 multi-tier platforms (fixed for reachability with JUMP_POWER=-8.5)
- 34 turrets (21 ground + 13 on platforms) + 2 tanks = 36 fixed enemies
- Wave interval: 2200ms, up to 3 mobile enemies simultaneously
- All enemies SOLID — horizontal push-back + contact damage
- Viewport shooting fix: ALL enemies check viewport before firing
- Fortress destruction: 3s sequence — 12 explosions, 4 falling chunks, 15 debris, 2.5s shake
- Level exit: walk through destroyed fortress off right edge
- BGM: War Drums (~125 BPM)

## Stage 2: THE CORRIDOR (IN PROGRESS — ~55% complete)

### What's built:
- **Full 7200px level playable end-to-end** across all 4 zones
- S2_W=7200, S2_FX=6900, CEIL_Y=30
- 4 zones: Entry Tunnel (0-1800), Lab Sector (1800-3600), Reactor Core (3600-5400), Sentinel Chamber (5400-7200)

**Background & visuals:**
- Enclosed corridor with ceiling, floor, pipes, wall panels, warning stripes
- **Zone-dependent palette** — steel-blue (tunnel) → purple (lab) → orange/red (reactor) → dark red (sentinel)
- **Reactor Core:** glowing orange conduit pipes on walls (pulsing animation)
- **Sentinel Chamber:** emergency red light strips on ceiling (flashing)
- Metal grate platforms (gray steel, distinct from Stage 1 green)

**Hazards (all difficulty-scaled):**
- **Conveyor belts** — 12 total across all zones, push player L/R at 0.8px/frame
- **Laser gates** — 11 total, cycle on/off, damage on contact. Off-time extended by `laserOffBonus` on easier modes
- **Steam vents** — 8 in Reactor Core, telegraph shake (500ms) → burst damage (1000ms) → cooldown. Cycle extended by `steamCycleBonus` on easier modes
- **Blackout event** — one-time setpiece at x=3200 (Lab/Reactor boundary). 2.5s near-darkness, enemy eye-glows, WARNING text flash, alarm SFX, screen shake. Spawns `blackoutCrawlers` (3/5/7 by difficulty)

**Enemies:**
- **Ceiling turrets** — 22 total, NES turret flipped, shoot downward
- **Ground turrets + tanks** — 24 turrets + 5 tanks across all zones
- **Crawlers** — 15 spawn points, groups of 2-6 (scaled by `enemyDensity`). Speed uses `crawlerSpeed` (0.6/0.9/1.2 by difficulty). 600ms landing pause then chase.
- **Energy orbs** — 15 total, sine wave float, 4 shrapnel bullets on death
- **Differentiated scoring** — Crawler:50, Orb:150, Ceiling Turret:150, others:100
- **Stage-dependent spawning** — Stage 1 enemies gated out of Stage 2
- **Behind-spawning** — 10% chance enemies come from behind

**Sound:**
- `steamHiss()` — filtered noise burst for vent activation
- `blackoutAlarm()` — descending sweep + rising alarm for blackout event

### What's NOT built yet (remaining ~45%):
- Alien Warrior enemy (3HP, charges+leaps, mini-boss class, appears 2-3 times in Reactor/Sentinel)
- Shield Trooper enemy (2HP, frontal shield, must hit from behind/above/grenades)
- The Sentinel boss (3-phase walking mech: cannon barrage → stomp+drones → core exposed, 20HP core)
- Stage 2 BGM: Industrial Pulse (~140 BPM)
- Stage 2 stage-clear screen (currently shows "JUNGLE BREACH COMPLETE" — needs "THE CORRIDOR COMPLETE")
- End-of-level trigger for Stage 2 (needs Sentinel boss defeat → `fortressOpen=true` flow)

## Sound (LOCKED — Hollywood Action v3)
- All Web Audio API. Zero external files.
- Weapon-specific: shoot() for basic/rapid, machineGun() for M, spreadShot() for S, fireball() for F
- Stage 2 additions: steamHiss(), blackoutAlarm()
- BGM: War Drums — kick/snare/hi-hat + saw bass riff, 0.12s/step
- Master volume: 0.35

## Player Movement (LOCKED)
- BILL_SPEED = 2.0, JUMP_POWER = -8.5 (max jump ~60px)
- Variable jump: releasing UP cuts jump short
- Gravity: 0.6, MAX_FALL: 8

## Exception handling (CRITICAL)
- NEVER hardcode negative coordinates
- All coords clamped with Math.max(0, ...)
- Validate sprite bounds before every ctx.drawImage()
- try/catch around asset loading and rendering
- Death/respawn: gameTime-based only, NEVER setTimeout
- Array removals: iterate backwards

## Key technical constants
- `currentStage` — 1 (production) or 2 (WIP testing)
- `CEIL_Y` = 30 (Stage 2 ceiling collision)
- `S2_PLATS[]`, `S2_ZONES[]`, `S2_CONVEYORS[]`, `S2_LASERS[]`, `S2_CEIL_TURRETS[]`, `S2_FIXED[]`, `S2_CRAWLER_SPAWNS[]`, `S2_ORBS[]`, `S2_STEAM_VENTS[]`
- `s2CrawlerTriggered[]` — tracks which crawler zones have been activated
- `blackoutTriggered`, `blackoutStart` — blackout event state
- Crawler/orb sprites embedded as base64 (`crawler`, `orb` keys in ASSETS_B64)

## Conventions
- Single HTML file, no build step
- Pure Canvas 2D, no composite operations
- Show visual mockups BEFORE coding
- Keep responses brief
- **Two-file workflow:** edit WIP → test → copy to index.html with `currentStage=1` → commit
- **NEVER commit with `currentStage=2`**
- Read `stage-2-rationale.md` before making number changes — every value has documented reasoning
