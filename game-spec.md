# BREACH — Game Design Document

A Contra-style 2D run-and-gun platformer set in 2089.

## Pitch

**"Three operatives. No backup. Break through everything between you and the end of the world."**

## Story

**Year: 2089.** A paramilitary syndicate called **Crimson Dawn** has seized Archipelago Sigma in the South Pacific. Hidden beneath its volcanic islands: **Project Starkfire** — a superweapon that sends a single cascading plasma pulse to kill every power grid on Earth. No missiles, no warheads — just one pulse, and civilization goes dark. Eight billion people plunged into permanent blackout. The weapon is 72 hours from activation. Three operatives are deployed. No backup. No extraction plan.

## Design pillars

1. **Nostalgic** — NES Contra (1987) feel. Pixel art, chiptune sounds, 60fps run-and-gun.
2. **Fair** — hard but never cheap. Always a telegraph before a threat.
3. **Snappy** — controls respond instantly. Frame-perfect feedback.

## Controls

| Key | Action |
|-----|--------|
| ← → | Move / aim horizontally |
| ↑ | Aim up (with X) |
| ↓ | Prone (lie flat, dodge bullets) |
| ↓ + SPACE | Drop through platform |
| SPACE | Jump |
| X | Shoot |
| C | Throw grenade |
| ENTER | Advance screens / confirm |

## Scene Flow (LOCKED)

5-screen title flow → gameplay → end screens:

1. **Splash** — BREACH logo (NES pixel art), taglines, year 2089, "PRESS START", browser-only notice at bottom
2. **Difficulty Select** — EASY / NORMAL / HARD with arrow selection
3. **Character Select** — 3 panels side-by-side with stat bars, sprite preview, description, and quote. LEFT/RIGHT to select, ENTER to confirm, ESC to go back
4. **Story Prelude** — Star Wars style scrolling text. Full Starkfire narrative scrolls from bottom to top. After all text scrolls off, blank pause then auto-advance (or Enter to skip)
5. **Stage Briefing** — "STAGE 1 OF 5: JUNGLE BREACH" with typewriter mission briefing, shows selected operative + difficulty. ENTER to deploy

Then: **Play** → **Game Over** (with retry/title) or **Stage Clear** (with score + next stage tease)

## Characters (3 playable — selectable at character select screen)

All share the same frame map, animation logic, and gameplay stats.

| # | Codename | Sheet File | Size | Grid | Cell | Stats |
|---|----------|-----------|------|------|------|-------|
| 1 | **Kane** | `hamza_sheet_v2` | 924×340 | 14×5 | 66×68 | POW:4 SPD:3 ARM:4 RNG:3 |
| 2 | **Viper** | `john_wick_sheet` | 924×340 | 14×5 | 66×68 | POW:3 SPD:5 ARM:2 RNG:4 |
| 3 | **Ironside** | `tiger_sheet_v2` | 924×340 | 14×5 | 66×68 | POW:5 SPD:2 ARM:5 RNG:3 |

### Character quotes
- **Kane:** *"We go in together. We come out together."*
- **Viper:** *"I don't need a plan. I need a door."*
- **Ironside:** *"Point me at the biggest thing. I'll make it smaller."*

### Frame map (LOCKED — shared by all)
```
idle: 5, run1: 1, run2: 3, aimUp: 12
prone: 31 (head LEFT in raw sprite)
flip: [28, 29, 30]
```

### Prone system (LOCKED)
- Frame 31 with drawY += 18 offset. Hitbox shrinks to {y+28, h:12}.
- Head ALWAYS faces aiming direction. Facing RIGHT = flip sprite. Facing LEFT = raw.

## Weapons

| Letter | Name | Sound | Behavior |
|--------|------|-------|----------|
| — | Basic | Assault rifle crack | Standard red bullet, 160ms cadence |
| M | Machine Gun | M60 rattle | Fat red bullets, 90ms cadence |
| S | Spread Shot | Shotgun blast | 5-way fan, 180ms cadence |
| R | Rapid Fire | Assault rifle crack | Fast thin bullets, 80ms cadence |
| F | Fire | Napalm launch | Big orange fireball, 180ms cadence |

## Enemies (LOCKED)

| Role | Sprite | Behavior |
|------|--------|----------|
| Soldier | Grunt sheet (36×40) | Walks at 1.2, shoots every ~2.2s |
| Charger | Grunt + red headband | Rushes at 3.0 |
| Turret | NES wall turrets (35×32) | Pop-up on ground OR sit on platforms, HP=2, shoot diagonally toward player |
| Tank | Procedural (80×56) | Stationary, 3 HP, fires every 2.5s |

### Enemy density
- Wave interval: 2200ms (was 3500ms)
- Up to 3 mobile enemies on screen simultaneously
- 31 fixed turrets/tanks across the level (many on platforms at varied heights)
- All enemies are SOLID — player cannot walk through them, takes contact damage

## Stages (5 total)

### Stage 1: JUNGLE BREACH — *"Get boots on the ground."* (BUILT)
6400px level, 3 zones. 21 multi-tier platforms with connected traversal (stacked 2-3 tiers, staircases, bridges). Turrets on ground AND platforms shooting diagonally.
**Boss: The Gate** — fortified wall with 4 stacked cannons + central core (15 HP).
**Fortress destruction:** Massive 3-second sequence — 12 cascading explosions with sound, wall splits into 4 rotating/falling chunks, 15 debris particles, 2.5s screen shake, orange flash effects.
**Exit:** After fortress crumbles, player walks through the gap and off the right edge to trigger Stage Clear.

### Stage 2: THE CORRIDOR (planned) → Sentinel mech boss
### Stage 3: THE CASCADE (planned) → Hydra Battery boss
### Stage 4: THE FURNACE (planned) → Forge Master boss
### Stage 5: THE CORE (planned) → Starkfire boss

## Art Direction (LOCKED)

**Engine:** Pure Canvas 2D + requestAnimationFrame. No Phaser, no physics engine.
Viewport: 512×288 at 2× CSS scale (1024×576 on screen). GROUND_Y = 216.

**Logo:** NES pixel art letterforms (10×12 grid per letter, 3px scale). Cream letters (#e8e2d4) with black drop shadow, red strikethrough (#d94236) at crossbar row. Center-aligned.

## Difficulty

| Mode | HP | Invuln | Enemy Count | Bullet Speed |
|------|-----|---------|--------------|---------------|
| Training | ∞ | ∞ | 0 | n/a |
| Easy | 7 | 4s | ×0.3 | ×0.45 |
| Normal | 5 | 3s | ×0.5 | ×0.6 |
| Hard | 3 | 1.8s | ×0.7 | ×0.85 |

## Audio (LOCKED — Hollywood Action v3)

All Web Audio API. Zero external files. Multi-layered action movie style.
- **BGM:** War Drums — kick/snare/hi-hat + sawtooth bass riff, 0.12s/step (~125 BPM). Starts on title screen.
- **Weapon SFX:** Each weapon has unique sound — shoot (rifle crack), machineGun (M60 rattle), spreadShot (shotgun blast), fireball (napalm whoosh)
- **Locked SFX:** Jump (150→400Hz sweep), Death (NES staircase 400→80Hz), Stage Clear (C5→C6 arpeggio)
- **Master volume:** 0.35

## Architecture (LOCKED)

- Pure Canvas 2D + requestAnimationFrame — NO Phaser
- Single HTML file (`index.html`), no build step, no external dependencies
- All assets embedded as base64
- All audio synthesized procedurally
- Optimised for web browsers · True to retro
