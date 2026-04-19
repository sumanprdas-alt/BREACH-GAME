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

## Characters (3 playable — selectable at title screen)

All share the same frame map, animation logic, and gameplay stats.

| # | Codename | Sheet File | Size | Grid | Cell | Description |
|---|----------|-----------|------|------|------|-------------|
| 1 | **Kane** | `hamza_sheet_v2` | 924×340 | 14×5 | 66×68 | The Commander. Cold, precise. *"We go in together. We come out together."* |
| 2 | **Viper** | `john_wick_sheet` | 924×340 | 14×5 | 66×68 | The Ghost. *"I don't need a plan. I need a door."* |
| 3 | **Ironside** | `tiger_sheet_v2` | 924×340 | 14×5 | 66×68 | The Wall. *"Point me at the biggest thing. I'll make it smaller."* |

### Character stats (shared)
- 3 HP (Normal), 5 HP (Easy), 7 HP (Training)
- Loses current weapon when hit
- 3 lives, respawns at checkpoint with 2.5s invulnerability
- Starts every life with 3 grenades
- On respawn: ALL mobile enemies removed, enemy bullets cleared, wave timer reset

### Frame map (LOCKED — shared by all)
```
idle: 5, run1: 1, run2: 3, aimUp: 12
prone: 31  (head LEFT in raw sprite)
flip: [28, 29, 30]
```

### Prone system (LOCKED)
- Frame 31 with drawY += 18 offset. Hitbox shrinks to {y+28, h:12}.
- Head ALWAYS faces aiming direction. Facing RIGHT = flip sprite. Facing LEFT = raw.

## Weapons

| Letter | Name | Behavior |
|--------|------|----------|
| — | Basic | Standard red bullet, 160ms cadence |
| M | Machine Gun | Fat red bullets, 90ms cadence |
| S | Spread Shot | 5-way fan, 180ms cadence |
| R | Rapid Fire | Fast thin bullets, 80ms cadence |
| F | Fire | Big orange fireball, 180ms cadence |

## Enemies (LOCKED)

| Role | Sprite | Behavior |
|------|--------|----------|
| Soldier | Grunt sheet (36×40) | Walks at 1.2, shoots every ~2.2s |
| Charger | Grunt + red headband | Rushes at 3.0 |
| Sniper | Rifle soldiers (20×36) | Stationary, shoots every ~1.1s |
| Turret | Wall turrets IN rock (35×32) | Pop-up within 200px, HP=2 |
| Tank | Procedural (80×56) | Stationary, 3 HP, fires every 2.5s |
| Mini-boss | Alien warriors (55×70) | Fortress zone |
| Surprise | Pink crawlers (28×24) | Ambush from ground |
| Boss | Dogra (49×42) | Boss encounter |

## Stages (5 total)

### Stage 1: JUNGLE BREACH — *"Get boots on the ground."*
Dense jungle, patrol squads, watchtower turrets. 6400px, 3 zones.
**Boss: The Gate** — fortified wall, 4 cannons + 15HP core.

### Stage 2: THE CORRIDOR — *"We're inside. Stay sharp."*
Indoor base. Tight corridors, security systems, armored soldiers.
**Boss: Sentinel** — walking mech.

### Stage 3: THE CASCADE — *"The island is alive."*
Vertical ascent up waterfall cliff. Collapsing platforms, rushing water.
**Boss: Hydra Battery** — multi-cannon artillery nest.

### Stage 4: THE FURNACE — *"This is where they build it."*
Weapons facility. Molten conduits, rotating machinery.
**Boss: The Forge Master** — shielded command platform.

### Stage 5: THE CORE — *"Shut it down. Whatever it takes."*
Starkfire chamber. Alien-organic architecture. Walls pulse, floor breathes.
**Boss: Starkfire** — sentient biomechanical horror.

## Art Direction (LOCKED)

**Engine:** Pure Canvas 2D + requestAnimationFrame. Viewport: 512×288 at 2× CSS scale. GROUND_Y=216.

**Rocks:** ONE large 256×128 texture. THREE layers. 8-10 vertex irregular polygons + pixel roughness. Yellow-green palette: specular(225,205,50)→base(45,42,5). Zero gaps.

**Grass:** 10-12px thick, 3 bands: tip #88D048, body #38A038, shadow #184818. Serrated top + draping bottom.

**Background:** Dark crosshatch. base(2,3,0), stripes(22,28,9).

**Fern fringe:** Top of screen only. Must NOT overlap play area.

## Logo (LOCKED)

- **BREACH** in bold angular military stencil, steel (#d8d0c0), gold edge (#fcd848)
- Red fracture line (#e03828) through B, R, C, H only. E and A clean.
- Tagline: "THREE OPERATIVES. NO BACKUP." / "BREAK THROUGH EVERYTHING"
- Year: 2089

## Difficulty

| Mode | HP | Invuln | Enemy Count | Bullet Speed |
|------|-----|---------|--------------|---------------|
| Training | ∞ | ∞ | 0 | n/a |
| Easy | 7 | 4s | ×0.3 | ×0.45 |
| Normal | 5 | 3s | ×0.5 | ×0.6 |
| Hard | 3 | 1.8s | ×0.7 | ×0.85 |

## Audio (LOCKED — Hollywood Action v3)

All Web Audio API. Zero external files. Multi-layered action movie style.
- **BGM:** War Drums — kick/snare/hi-hat + sawtooth bass riff, 0.12s/step (~125 BPM)
- **Locked SFX:** Jump (150→400Hz sweep), Death (NES staircase 400→80Hz), Stage Clear (C5→C6 arpeggio)
- **Sound test:** `/mnt/user-data/outputs/sound-test-v3.html`

## Scenes flow

Boot → Title (character select + difficulty) → Prelude (mission briefing) → Play → GameOver/StageClear
