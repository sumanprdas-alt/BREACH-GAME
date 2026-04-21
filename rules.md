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

## Two-file workflow (Stage 2 development)
- **`index.html`** — production build, `currentStage=1`. Committed to repo.
- **`breach-s2-wip.html`** — Stage 2 testing, `currentStage=2`. Local only, NOT in repo.
- Both files are identical except the `currentStage` variable.
- Always edit the WIP file during development. Copy to `index.html` and set `currentStage=1` before committing.
- **NEVER commit with `currentStage=2`.**
- Read `stage-2-rationale.md` before changing any numbers — every value has documented reasoning.

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

## Difficulty (REBALANCED — 60% novice / 30% amateur / 10% expert)

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

**All properties are wired in.** `crawlerSpeed` scales crawler chase speed. `laserOffBonus` adds extra off-time to laser gates. `steamCycleBonus` adds extra cooldown to steam vents. `blackoutCrawlers` controls how many crawlers spawn during the blackout event. `enemyDensity` controls what fraction of fixed enemies and crawler swarm counts actually spawn.

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

## Stage 1 specifics (LOCKED)
- 6400px, 3 zones, 21 platforms, 34 turrets + 2 tanks
- Bridge + helicopter set-piece
- Fortress boss (4 cannons + 15HP core)
- Massive destruction sequence on boss defeat
- BGM: War Drums (~125 BPM)

## Stage 2 specifics (IN PROGRESS — ~55%)
- 7200px, 4 zones, CEIL_Y=30
- Enclosed corridor: ceiling + floor + wall panels
- Zone-dependent background palette (blue → purple → orange → red)
- Metal grate platforms (gray, not green)
- Hazards: 12 conveyor belts, 11 laser gates, 8 steam vents, 1 blackout event
- Enemies: 22 ceiling turrets, 24 ground turrets + 5 tanks, 15 crawler spawn points, 15 energy orbs
- All hazards/enemies difficulty-scaled via DIFFICULTY properties
- Stage-dependent enemy spawning (Stage 1 enemies gated out)
- Behind-spawning at 10% rate
- Differentiated scoring per enemy type
- SFX: steamHiss(), blackoutAlarm()
- Environmental props gated to Stage 1 only

### NOT yet built:
- Alien Warrior enemy (3HP, charge+leap)
- Shield Trooper enemy (2HP, frontal shield)
- The Sentinel boss (3-phase mech)
- Stage 2 BGM: Industrial Pulse (~140 BPM)
- Stage 2 stage-clear screen text
- End-of-level trigger via Sentinel boss defeat

## Player movement (LOCKED)
- BILL_SPEED = 2.0, JUMP_POWER = -8.5 (max jump ~60px)
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
- Scale ALL hazards/enemies via DIFFICULTY properties
- Read `stage-2-rationale.md` before changing numbers
- Edit WIP file, copy to index.html with `currentStage=1` before committing

**DON'T:**
- Use Phaser or physics engines
- Use `source-atop` composites
- Use setTimeout for game logic
- Spawn Stage 1 enemies in Stage 2 or vice versa
- Let enemies fire from off-screen
- Make platforms unreachable (check jump height vs platform Y)
- Rebuild from scratch
- Hardcode difficulty values — always use `diff.propertyName`
- Commit with `currentStage=2`
