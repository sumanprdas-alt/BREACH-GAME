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

## Two-file workflow (CRITICAL)

Both files contain the SAME codebase. The ONLY difference is one variable:

| File | `currentStage` | Purpose | Location |
|------|---------------|---------|----------|
| `index.html` | `1` | Production build. Plays Stage 1. Stage 2 code present but gated. | GitHub repo |
| `breach-s2-wip.html` | `2` | Stage 2 testing. Jumps straight to corridor. | Local / outputs only |

**Workflow:**
1. Always edit `breach-s2-wip.html` during Stage 2 development
2. Test with `currentStage=2`
3. To commit: copy WIP → `index.html`, set `currentStage=1`, push
4. **NEVER commit with `currentStage=2`** — production always starts at Stage 1

**Baseline commit:** `5da0d90` — Stage 1 production + Stage 2 gated code (Phase A+B + difficulty rebalance)

## Engine architecture
- `requestAnimationFrame` loop with `ctx.drawImage()`
- Collision: `if(feetY >= groundY) { y = groundY - height }`
- Viewport: 512×288 internal, 2× CSS scale. GROUND_Y = 216.
- NO `source-atop` or composite operations.
- Screen shake: `screenShake` variable, decremented each frame.

## Scene flow (LOCKED)
```
Splash → Difficulty Select → Character Select → Story Prelude → Stage Briefing → Play → GameOver/StageClear
```

## Sprite conventions

### Character sheets (3 playable)
924×340, 14×5 grid, 66×68 cells. Frame map: idle(5), run(1,3), aimUp(12), prone(31), flip(28-30).

### Prone rules (LOCKED)
- Frame 31, drawY+=18, hitbox {y+28,h:12}
- flipX = bill.facing < 0 (same as standing)

### Enemy sprites
- Grunt sheet (36×40, 2 frames). Charger=red headband. Sniper=yellow headband.
- NES turret sprites (35×32). Ground AND platform AND ceiling (flipped for Stage 2).
- Crawler sprite: embedded base64, pink alien facehugger.
- Orb sprite: embedded base64, blue plasma sphere.

## Enemy collision (LOCKED)
- ALL enemies are SOLID — push-back + contact damage
- ALL enemies check viewport before firing (no off-screen shots)
- Turrets only solid after revealed/popped up

## Difficulty (REBALANCED — tuned for 60% novice / 30% amateur / 10% expert)

| Property | Easy | Normal | Hard |
|----------|------|--------|------|
| HP | 9 | 6 | 3 |
| Lives | 5 | 4 | 2 |
| Shot interval (ms) | 4500 | 3000 | 1800 |
| Invulnerability (ms) | 4000 | 2800 | 1500 |
| Bullet speed mul | 0.35 | 0.55 | 0.9 |
| Enemy speed mul | 0.5 | 0.75 | 1.1 |
| Enemy density | 0.35 | 0.6 | 1.0 |
| Crawler speed | 0.6 | 0.9 | 1.2 |
| Laser off bonus (ms) | 800 | 400 | 0 |
| Steam cycle bonus (ms) | 1200 | 500 | 0 |
| Blackout crawlers | 3 | 5 | 7 |

**ALL properties are wired in and functional.** Earlier builds had `crawlerSpeed` and `laserOffBonus` defined but never used — now fixed. Read `stage-2-rationale.md` before changing any values.

## Stage 1 specifics (LOCKED)
- 6400px, 3 zones, 21 platforms, 34 turrets + 2 tanks
- Bridge + helicopter set-piece
- Fortress boss (4 cannons + 15HP core)
- Massive destruction sequence on boss defeat
- BGM: War Drums (~125 BPM)

## Stage 2 specifics (IN PROGRESS — ~55% complete)
- 7200px, 4 zones, CEIL_Y=30
- Enclosed corridor: ceiling + floor + wall panels
- **Zone-dependent palette** — blue-gray → purple → orange/red → dark red
- Metal grate platforms (gray, not green)
- **Hazards (all difficulty-scaled):** conveyor belts (12), laser gates (11), steam vents (8, Reactor only), blackout event (one-time at x=3200)
- **Enemies:** ceiling turrets (22), ground turrets (24) + tanks (5), crawlers (15 spawn points), energy orbs (15)
- Stage-dependent enemy spawning (Stage 1 enemies gated out)
- Behind-spawning at 10% rate
- Differentiated scoring per enemy type
- Environmental props gated to Stage 1 only
- Reactor Core: glowing conduit pipes. Sentinel Chamber: emergency light strips.
- SFX: `steamHiss()`, `blackoutAlarm()`

### NOT built yet (~45% remaining):
- Alien Warrior enemy (3HP, charges+leaps)
- Shield Trooper enemy (2HP, frontal shield)
- The Sentinel boss (3-phase mech)
- Stage 2 BGM: Industrial Pulse (~140 BPM)
- Stage 2 stage-clear screen text
- End-of-level trigger (needs Sentinel boss defeat)

## Player movement (LOCKED)
- BILL_SPEED = 1.6, JUMP_POWER = -8.5 (max jump ~60px)
- Variable jump: releasing UP cuts jump short
- Gravity: 0.6, MAX_FALL: 8
- Somersault: frames 28→29→30 at 80ms/frame

## Sound (LOCKED — Hollywood v3)
- All Web Audio API. Zero external files.
- Weapon-specific sounds per weapon type.
- Stage 2 additions: steamHiss(), blackoutAlarm()
- BGM starts on splash PRESS START.
- Master volume: 0.35

## Exception handling (CRITICAL)
- NEVER hardcode negative coordinates
- All coordinate math clamped with Math.max(0, ...)
- Validate sprite bounds before every ctx.drawImage()
- try/catch around all asset loading and rendering
- Death/respawn: gameTime-based only, NEVER setTimeout
- Array removals: iterate backwards

## Do / Don't

**DO:**
- Show mockups before coding visuals
- Use gameTime-based timers
- Gate stage-specific code with `if(currentStage===N)`
- Test platform reachability (max 50px vertical gap between connected platforms)
- Scale all hazards/enemies by difficulty properties (density, speed, timing bonuses)
- Edit `breach-s2-wip.html` during development, commit `index.html` with `currentStage=1`
- Read `stage-2-rationale.md` before changing any placement numbers

**DON'T:**
- Use Phaser or physics engines
- Use `source-atop` composites
- Use setTimeout for game logic
- Spawn Stage 1 enemies in Stage 2 or vice versa
- Let enemies fire from off-screen
- Make platforms unreachable (check jump height vs platform Y)
- Rebuild from scratch
- Commit with `currentStage=2`
- Hardcode difficulty-dependent values (always use `DIFFICULTY[diffKey]` properties)
