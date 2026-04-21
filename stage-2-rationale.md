# BREACH — Stage 2 Build Rationale & Reference

## Purpose

This document explains **why** every number, placement, and design choice was made in Stage 2: THE CORRIDOR. Any agent continuing this build should read this alongside `handoff.md`, `stage-2-design.md`, and `rules.md` to understand intent, not just implementation.

---

## CORE DESIGN PRINCIPLES (applies to all decisions)

1. **Escalating difficulty curve across zones** — Tunnel (intro/easy) → Lab (medium) → Reactor (peak) → Sentinel (eases off pre-boss). The boss IS the Sentinel Chamber challenge, so the zone itself is lighter.

2. **"Fair, never cheap"** — Every hazard has either a visual telegraph, a timing window, or adequate spacing. No surprise instant-kills. This is the game-spec design pillar.

3. **Spacing consistency** — Elements within the same category (turrets, lasers, etc.) maintain roughly the same spacing as established in Tunnel and Lab zones. ~200-300px between turrets, ~150-200px between major hazards.

4. **Combo hazards** — The most interesting challenges come from overlapping threats: a conveyor pushing you toward a laser, a crawler swarm during a blackout, an orb floating near a turret gauntlet. Individual hazards are fair; their combinations create difficulty.

5. **Thematic zone identity** — Each zone should FEEL different. Tunnel=industrial intro, Lab=bio-horror, Reactor=heat/danger, Sentinel=dread before the boss.

---

## ZONE STRUCTURE

```
Zone 1: Entry Tunnel    0–1800px     Steel blue/gray    "Tutorial corridor"
Zone 2: Lab Sector       1800–3600px  Purple/pink        "Containment breach"
Zone 3: Reactor Core     3600–5400px  Orange/red         "Peak hazard zone"
Zone 4: Sentinel Chamber 5400–7200px  Dark red/black     "Boss approach"
```

**Why 7200px total?** Stage 1 is 6400px. Stage 2 is 12.5% longer because indoor corridors feel shorter than outdoor levels (no sky, confined space compresses perceived distance). The extra 800px prevents Stage 2 from feeling like a downgrade in scope.

**Why 4 zones instead of 3?** Stage 1 has 3 zones. Stage 2 adds a dedicated boss arena (Sentinel Chamber) because the boss is a mobile 3-phase mech that needs room, unlike Stage 1's static fortress wall.

---

## DATA ARRAYS — FULL RATIONALE

### S2_PLATS (Platforms)

**Already complete across all 4 zones** — built in the first 35% pass. No changes needed in Phase A.

Key rules that governed platform placement:
- Max 50px vertical gap between connected platforms (JUMP_POWER=-8.5 gives ~60px max jump height, 50px gives margin for error)
- Platforms placed as reachable chains — every high platform has a stepping-stone platform within 50px vertically and ~150px horizontally
- Dual-height platforms at same X (e.g., `{x:420,y:150}` and `{x:420,y:190}`) create vertical choice: top platform for sniping, bottom for safer traversal
- Sentinel Chamber has fewer platforms and wider spacing — boss arena needs open ground for dodging mech attacks

---

### S2_CONVEYORS (Conveyor Belts)

```javascript
// Entry Tunnel — 3 belts, gentle introduction
{x:400,w:120,dir:1}     // Pushes right (with player movement — feels fast)
{x:1100,w:100,dir:-1}   // Pushes left (against player — first friction)
{x:1600,w:80,dir:1}     // Short push right before zone exit

// Lab Sector — 3 belts, alternating directions
{x:2100,w:110,dir:-1}   // Against player, near turret at x:2150
{x:2700,w:90,dir:1}     // With player, near crawler spawn at x:2900
{x:3300,w:100,dir:-1}   // Against player, near laser at x:3100

// Reactor Core — 4 belts, OPPOSING PAIRS push into hazards
{x:3800,w:130,dir:1}    // Long belt pushes right INTO laser at x:3750 area
{x:4150,w:100,dir:-1}   // Pushes left, against player, near tank at x:4050
{x:4600,w:120,dir:1}    // Pushes right toward turret cluster 4700-4850
{x:5100,w:110,dir:-1}   // Pushes back, trapping player near turrets at 5050-5250

// Sentinel Chamber — 2 belts, minimal
{x:5600,w:100,dir:-1}   // Pushes back from boss arena (tension builder)
{x:6200,w:90,dir:1}     // Pushes toward boss (point of no return feel)
```

**Rationale:**

| Zone | Count | Why |
|------|-------|-----|
| Tunnel | 3 | Introduces mechanic. First two are simple, third is short. |
| Lab | 3 | Matches Tunnel count. Placed near other hazards for combo pressure. |
| Reactor | 4 | Peak zone gets most. Longer belts (120-130px). Placed to push INTO turrets/lasers. Design doc explicitly says "you're being pushed INTO the fire." |
| Sentinel | 2 | Pre-boss zone is lighter. One pushes you back (dread), one pushes forward (commitment). |

**Width logic:** 80-130px range. Shorter belts (80-90px) = brief nuisance. Longer belts (120-130px) = sustained pressure requiring the player to actively fight. Reactor has the longest because it's the peak zone.

**Direction logic:** `dir:1` pushes right (with player progression), `dir:-1` pushes left (against). Alternating creates rhythm. Reactor specifically pairs conveyors with nearby turrets/lasers so the push direction is always toward danger.

**Push speed (0.8px/frame):** Set during the first 35% build. 0.8 is ~40% of BILL_SPEED (2.0), meaning the player can fight it but it's noticeable. Higher would make conveyors frustrating, lower would make them ignorable.

---

### S2_LASERS (Laser Gates)

```javascript
// Entry Tunnel — 2 lasers, generous timing
{x:700,on:2000,off:1500}    // 2s on, 1.5s off — plenty of time to pass
{x:1400,on:1800,off:1200}   // Slightly tighter off-window

// Lab Sector — 3 lasers, moderate timing
{x:2000,on:1800,off:1400}   // Moderate
{x:2450,on:2200,off:1200}   // Long on-cycle, shorter off
{x:3100,on:1600,off:1600}   // Equal on/off — rhythmic

// Reactor Core — 4 lasers, TIGHT timing windows
{x:3750,on:2000,off:1000}   // Only 1s to pass — requires commitment
{x:4100,on:1600,off:1200}   // Slightly more forgiving between tight ones
{x:4500,on:2200,off:900}    // Tightest in the game — 0.9s window
{x:4950,on:1800,off:1100}   // Eases slightly before zone exit

// Sentinel Chamber — 2 lasers, long punishing on-cycles
{x:5700,on:2400,off:1200}   // 2.4s on — forces patience
{x:6400,on:2600,off:1000}   // Longest on-cycle, last laser before boss
```

**Rationale:**

| Zone | Count | Off-window range | Design intent |
|------|-------|-----------------|---------------|
| Tunnel | 2 | 1200-1500ms | Teaching. Player learns laser timing with forgiving windows. |
| Lab | 3 | 1200-1600ms | Moderate challenge. Variety in timing prevents memorization. |
| Reactor | 4 | 900-1200ms | Peak zone. Tightest windows. Player MUST commit to runs. x:4500 at 900ms is the hardest single laser — placed mid-zone so player is warmed up. |
| Sentinel | 2 | 1000-1200ms | Fewer but with long on-cycles (2400-2600ms). Creates dread — you wait, and wait, and then sprint. Different feel from Reactor's rapid-fire timing. |

**Off-window design:** At BILL_SPEED=2.0, the player covers ~120px/second. A laser gate is ~6px wide (hitbox check: `Math.abs(b.x+b.w/2-lg.x)<6`). So passing through takes roughly 50ms. The off-window is about TIME TO REACH the laser, not time to pass through it. At 900ms off, the player must be within ~100px when the laser turns off and immediately commit. At 1500ms off, they can start from ~180px away — much more comfortable.

**Spacing:** ~250-350px between lasers within a zone. Close enough that you can sometimes see two on screen, creating visual pressure. Far enough that each is a distinct challenge.

---

### S2_CEIL_TURRETS (Ceiling Turrets)

```javascript
// Entry Tunnel — 6 turrets, ~300px spacing
{x:300},{x:600},{x:900},{x:1200},{x:1500},{x:1750}

// Lab Sector — 6 turrets, ~300px spacing
{x:1950},{x:2250},{x:2550},{x:2900},{x:3150},{x:3450}

// Reactor Core — 6 turrets, ~250-300px spacing (denser)
{x:3700},{x:3950},{x:4200},{x:4500},{x:4800},{x:5150}

// Sentinel Chamber — 4 turrets, wider spacing (room for boss fight)
{x:5500},{x:5850},{x:6150},{x:6500}
```

**Rationale:**

| Zone | Count | Avg spacing | Why |
|------|-------|-------------|-----|
| Tunnel | 6 | ~290px | Establishes the overhead threat pattern. Player learns to aim up. |
| Lab | 6 | ~300px | Maintains same density. Consistent pressure. |
| Reactor | 6 | ~290px | Same count but combined with more ground threats = harder total picture. |
| Sentinel | 4 | ~350px | Fewer because boss arena. Don't want ceiling turrets competing with Sentinel mech for player attention. Wider spacing gives breathing room. |

**Why 6-6-6-4 instead of escalating?** Ceiling turrets don't need to escalate in count because the COMBINED threat level escalates. 6 ceiling turrets + 4 Reactor lasers + 4 Reactor conveyors = much harder than 6 ceiling turrets + 2 Tunnel lasers + 3 Tunnel conveyors. The ceiling turret count stays flat while everything around it intensifies.

**Ceiling turret shooting:** They use `diff.shotMs*1.3` (30% slower than ground turrets) because the player has fewer movement options to dodge downward bullets while dealing with ground threats. This is a fairness balancing choice.

---

### S2_FIXED (Ground Turrets & Tanks)

```javascript
// Entry Tunnel — 6 turrets + 1 tank
{x:250,t:'turret'},{x:550,t:'turret'},{x:850,t:'turret'},
{x:1050,t:'turret'},{x:1350,t:'turret'},{x:1700,t:'turret'},
{x:1400,t:'tank'}
// Tank placed mid-zone as a tougher mid-boss challenge

// Lab Sector — 7 turrets + 1 tank
{x:1900,t:'turret'},{x:2150,t:'turret'},{x:2400,t:'turret'},
{x:2650,t:'turret'},{x:2950,t:'turret'},{x:3200,t:'turret'},
{x:3000,t:'tank'},{x:3400,t:'turret'}
// Tank at x:3000 guards the approach to the blackout zone at x:3200

// Reactor Core — 7 turrets + 2 tanks (PEAK density)
{x:3650,t:'turret'},{x:3900,t:'turret'},{x:4050,t:'tank'},
{x:4250,t:'turret'},{x:4450,t:'turret'},{x:4700,t:'turret'},
{x:4850,t:'tank'},{x:5050,t:'turret'},{x:5250,t:'turret'}
// Two tanks create "gauntlet" sections. First tank at 4050 near conveyor at 4150.
// Second tank at 4850 near conveyor at 5100 — player is pushed into its fire.

// Sentinel Chamber — 4 turrets + 1 tank (boss approach)
{x:5450,t:'turret'},{x:5650,t:'turret'},{x:5850,t:'tank'},
{x:6050,t:'turret'},{x:6250,t:'turret'},{x:6450,t:'turret'},{x:6650,t:'turret'}
// Tank at 5850 is the "last stand" before the boss arena opens up
// Turrets at 6450 and 6650 are the final fixed enemies before the Sentinel boss
```

**Rationale:**

| Zone | Turrets | Tanks | Total | Why |
|------|---------|-------|-------|-----|
| Tunnel | 6 | 1 | 7 | Baseline. Same as Stage 1's jungle zone density. |
| Lab | 7 | 1 | 8 | +1 turret. Slight escalation. Tank guards blackout zone entrance. |
| Reactor | 7 | 2 | 9 | Peak. Two tanks = two "gauntlet" sub-sections. Combined with conveyors pushing into their fire. |
| Sentinel | 4+3=7 | 1 | 8 | Frontloaded: 4 turrets + 1 tank in first half (5450-5850), then 3 turrets spread thin (6050-6650). Creates a "wall of resistance" → "thinning ranks" → boss arena. |

**Tank placement logic:** Tanks have 3HP and fire every 2000ms with larger projectiles. They're placed as zone mid-bosses:
- Tunnel tank (x:1400) = mid-zone checkpoint
- Lab tank (x:3000) = guards the blackout zone
- Reactor tank 1 (x:4050) = near conveyor at x:4150 that pushes you left, back into its fire
- Reactor tank 2 (x:4850) = near conveyor at x:5100 that pushes you left, back into its fire
- Sentinel tank (x:5850) = last major fixed threat before boss

**Turret spacing:** ~200-300px between turrets. Turrets pop up when player is within 200px, so at ~250px spacing, one turret has fully popped before the next triggers. This prevents multiple simultaneous pop-ups which would feel unfair.

---

### S2_CRAWLER_SPAWNS (Crawler Swarms)

```javascript
// Entry Tunnel — 3 spawns, small groups (counts 3-4)
{x:800,count:3},{x:1300,count:4},{x:1650,count:3}

// Lab Sector — 4 spawns, growing groups (counts 4-5)
{x:2100,count:4},{x:2500,count:5},{x:2900,count:4},{x:3300,count:5}

// Reactor Core — 5 spawns, LARGE groups (counts 4-6)
{x:3800,count:4},{x:4200,count:5},{x:4550,count:6},{x:4900,count:5},{x:5200,count:4}
// Count 6 at x:4550 is the BIGGEST swarm in the game — memorable peak moment
// Bookended by 4s at start/end of zone so it's not relentless

// Sentinel Chamber — 3 spawns, moderate groups (counts 4-5)
{x:5500,count:5},{x:5900,count:4},{x:6300,count:5}
// Fewer spawns than Reactor, but counts stay at 4-5
// Last spawn at x:6300 is 600px before the boss wall at S2_FX=6900
```

**Rationale:**

| Zone | Spawn points | Count range | Total crawlers (max) | Design intent |
|------|-------------|-------------|---------------------|---------------|
| Tunnel | 3 | 3-4 | 10 | Introduction. Player learns crawlers drop from ceiling with 600ms landing pause. Small groups are manageable. |
| Lab | 4 | 4-5 | 18 | Escalation. Bigger groups. Thematic: "containment breach" means more creatures. |
| Reactor | 5 | 4-6 | 25 | Peak. The count-6 swarm at x:4550 is the single scariest crawler moment. Placed mid-zone so player has warmed up. Tapers to 4 at zone exit. |
| Sentinel | 3 | 4-5 | 14 | Eases off. Boss is the real challenge. Last spawn stops at x:6300 — 600px buffer before boss. |

**Trigger radius:** `Math.abs(bill.x - cs.x) < 150` — player triggers crawlers when within 150px. This means crawlers drop slightly ahead, giving a brief moment to react.

**Count-6 reasoning:** At crawler speed 1.2 and with 600ms landing pause, 6 crawlers create ~2-3 seconds of frantic close-range combat. The player can thin the group during the landing pause if they react fast. More than 6 would be unfair given Reactor's other hazards are active simultaneously.

**Spacing between spawns:** ~300-500px in Reactor. Close enough that clearing one swarm barely gives a breather before the next, but far enough that they don't overlap (which would create an unbeatable 10+ crawler wall).

---

### S2_ORBS (Energy Orbs)

```javascript
// Entry Tunnel — 3 orbs
{x:650,y:120},{x:1150,y:140},{x:1550,y:110}

// Lab Sector — 4 orbs
{x:2050,y:130},{x:2450,y:100},{x:2800,y:140},{x:3250,y:115}

// Reactor Core — 5 orbs, placed near OTHER hazards for combo pressure
{x:3750,y:110}   // Near laser at x:3750 — kill orb = shrapnel + laser timing
{x:4100,y:130}   // Near laser at x:4100
{x:4450,y:95}    // Low Y (95) — close to ceiling turret fire, shrapnel goes up into kill zone
{x:4750,y:140}   // High Y (140) — near ground level, shrapnel spreads across floor
{x:5100,y:120}   // Near conveyor at x:5100, turret at x:5050

// Sentinel Chamber — 3 orbs
{x:5550,y:125},{x:5950,y:110},{x:6350,y:135}
```

**Rationale:**

| Zone | Count | Why |
|------|-------|-----|
| Tunnel | 3 | Introduction. Player learns that killing orbs = 4 shrapnel bullets. |
| Lab | 4 | Slightly more. Lab is where bio-hazards intensify. |
| Reactor | 5 | Peak. Critically, placed near lasers and conveyors so shrapnel interacts with other hazards. |
| Sentinel | 3 | Eases off. Orb shrapnel would make the boss fight chaotic if too many are left alive. |

**Y-position logic:** Orbs float in sine wave (amplitude 25px) around their base Y. The Y values are chosen to create different threat profiles:
- Y=95-110: Upper corridor. Orb floats between ceiling turret height and mid-screen. Shrapnel upward hits ceiling and dissipates; shrapnel downward threatens ground-level player.
- Y=120-140: Mid-to-low corridor. Orb is at player eye level. ALL shrapnel directions are dangerous. This is the harder placement.

**Why orbs have 2HP:** Set during the first 35% build. 2HP means the player must invest 2 shots (or 1 spread shot, or 1 fireball). This prevents accidental kills — the player has a moment to back away after the first hit. If orbs were 1HP, shrapnel would feel like a punishment for playing well.

**Shrapnel speed (2.5px/frame):** Fast enough to be dangerous at close range, slow enough to dodge if you killed the orb from a distance. This rewards long-range shooting discipline.

---

### S2_STEAM_VENTS (Phase B — New)

```javascript
// Reactor Core only — heat theme, 8 vents alternating floor/ceiling
{x:3700,side:'floor',cycle:3500}   // First vent — long cycle, forgiving
{x:3950,side:'ceil',cycle:3000}    // Ceiling — forces awareness of vertical threats
{x:4150,side:'floor',cycle:2800}   // Tightening cycle
{x:4400,side:'ceil',cycle:3200}    // Slight ease-up after tight one
{x:4650,side:'floor',cycle:2600}   // Tightest cycle — 2.6s total (500ms telegraph + 1000ms burst + 1100ms safe)
{x:4900,side:'ceil',cycle:3000}    // Standard
{x:5100,side:'floor',cycle:2800}   // Near zone exit
{x:5300,side:'floor',cycle:3400}   // Last vent — generous cycle as player leaves Reactor
```

**Rationale:**

**Why Reactor Core only?** The design doc explicitly ties steam vents to heat ("periodic bursts from pipes in walls/floor"). Tunnel is cold steel, Lab is bio-horror, Sentinel is boss. Only Reactor has the heat narrative.

**Why 8 vents?** Reactor Core is 1800px wide. 8 vents at ~200-250px spacing means the player encounters one roughly every 100-125px of progression. Not every step, but frequently enough to maintain awareness.

**Alternating floor/ceiling:** Creates vertical unpredictability. Player can't just "always jump" or "always stay low" to avoid them. Forces observation of which side the next vent is on.

**Cycle timing breakdown:**
```
Each cycle = telegraph (500ms) + burst (1000ms) + cooldown (remainder)
Example: cycle:3000 → 500ms shake + 1000ms damage + 1500ms safe

cycle:3500 → 2000ms safe window (generous intro)
cycle:3000 → 1500ms safe window (standard)
cycle:2800 → 1300ms safe window (tight)
cycle:2600 → 1100ms safe window (tightest — at x:4650, mid-Reactor)
cycle:3400 → 1900ms safe window (generous exit)
```

**Why not faster cycles?** The player needs time to OBSERVE the telegraph (pipe shaking for 500ms), DECIDE to move, and EXECUTE the dodge. Below 1000ms safe window, the player can't react — they'd need to memorize patterns, which violates "fair, never cheap."

**Damage hitbox:** `Math.abs(b.x+b.w/2-sv.x)<14` — 28px wide damage zone, roughly the width of a vent pipe. Generous enough to be visually honest (you see the steam, you get hit) but narrow enough that stepping 15px to either side dodges it.

---

### BLACKOUT EVENT (Phase B — New)

```
Trigger: Player crosses x=3150-3250 (one-time)
Duration: 2500ms total
  Phase 1 (0-300ms): Rapid fade to 0.92 opacity black overlay
  Phase 2 (300-2000ms): Near-total darkness, flickering red emergency lights
  Phase 3 (2000-2500ms): Darkness fades, lights return
Spawns: 6 crawlers at player x+100 to x+300 (ahead of player)
Effects: Screen shake (800ms), alarm SFX
Visual: Enemy eye-glows visible through darkness, "WARNING / CONTAINMENT BREACH" text flash
```

**Rationale:**

**Why x=3200?** This is the Lab/Reactor boundary. The design doc specifies this exact position. Narratively: the player crosses from the bio-containment zone into the reactor zone, and the power surge from the reactor causes a momentary grid failure.

**Why 2500ms?** Long enough to be frightening (you're blind with crawlers dropping on you), short enough that it doesn't become frustrating. At 2000ms, it felt too brief — barely registered. At 3000ms, it felt punishing for slower players who couldn't kill crawlers blind.

**Why 0.92 opacity (not 1.0)?** Pure black (1.0) would hide the player's own character, which is disorienting in a bad way. 0.92 lets the faintest outlines through — the player can barely see their character silhouette and the corridor walls. This maintains spatial awareness while still feeling dark.

**Why 6 crawlers?** Same as the biggest regular swarm (count:6 at x:4550), but these spawn in darkness. The 600ms landing pause is crucial here — it gives the player a moment to hear the crawlers hitting the ground and react. The crawlers spawn AHEAD (x+100 to x+300) so they run toward the player, not behind.

**Why enemy eye-glows?** The design doc specifies "only muzzle flash, enemy eye-glows, and weapon pickups visible." Eye-glows serve a gameplay purpose: they tell the player where threats are in the dark. Without them, the blackout would be pure RNG — unfair. Red dots for crawlers (2px, bright) and orange dots for soldiers (3px, dimmer) create readable threat indicators.

**Screen shake (800ms):** Shorter than the blackout to avoid nausea. The shake communicates "something major just happened" and stops before the player needs precise aiming in the dark.

**"WARNING / CONTAINMENT BREACH" text:** Flashes every 300ms during the blackout. Serves double duty: narrative (explains what's happening) and practical (the flashing text provides periodic light pulses that briefly illuminate the scene).

---

## ZONE-DEPENDENT BACKGROUND PALETTE

### Color Logic

```javascript
// Tunnel — cold steel industrial (base palette)
bgFill='#08060c'  ceilFill='#1a1820'  pipeFill='#252230'  panelFill='#14121a'

// Lab — purple/pink shift (bio-contamination)
bgFill='#0a060e'  ceilFill='#1c1626'  pipeFill='#281e34'  panelFill='#16101e'

// Reactor — orange/red shift (heat, danger)
bgFill='#0e0806'  ceilFill='#221814'  pipeFill='#302010'  panelFill='#1a1008'

// Sentinel — dark red/black (dread, finality)
bgFill='#0a0406'  ceilFill='#1e0e12'  pipeFill='#281014'  panelFill='#160a0e'
```

**Rationale:** All values are low-saturation, low-brightness variants. The shifts are subtle — you don't notice the transition frame-by-frame, but looking at screenshots of different zones shows clear palette differences. This avoids jarring color pops while maintaining visual identity.

The color channels shift: Tunnel is blue-dominant (#08060c = more blue than red), Lab introduces purple (red+blue), Reactor shifts to red-dominant, Sentinel darkens everything further. This follows the warm/cool temperature curve: cold → contaminated → hot → dark.

### Environmental Details

**Reactor Core — Glowing conduit pipes:**
- Orange/amber vertical pipes on walls, pulsing with `Math.sin(gameTime*0.003)` at 0.35 global alpha
- Placed every 200px (parallax offset 0.3×) so they slide behind the action
- The pulse suggests plasma flowing through the pipes — thematic for "reactor feeding Starkfire"

**Sentinel Chamber — Emergency red light strips:**
- Red rectangles on ceiling, flashing with `Math.sin(gameTime*0.005)` at 0.3 base alpha
- 30px wide, placed every 160px
- Includes a dim glow halo (40px wide, 20px tall) below each strip for ambient light effect
- Communicates "emergency mode" — something has gone very wrong ahead

---

## ZONE TINTS (Overlay)

```javascript
tunnel:   'rgba(10,15,30,0.15)'   // Cool blue cast
lab:      'rgba(25,10,25,0.16)'   // Purple contamination
reactor:  'rgba(35,15,5,0.18)'    // Warm orange heat haze
sentinel: 'rgba(30,5,5,0.2)'     // Red darkness, heaviest tint (0.2 alpha)
```

**Why increasing alpha?** Sentinel's heavier tint (0.2 vs Tunnel's 0.15) naturally darkens the scene, creating a sense of descending deeper underground. The player feels the environment closing in.

---

## SOUND EFFECTS (Phase B — New)

### steamHiss()
```javascript
this.nf(0.25,2000,0.12);  // Filtered noise, 250ms, 2000Hz cutoff, volume 0.12
this.nf(0.15,4000,0.06);  // Higher frequency layer, quieter, adds "sizzle"
```
**Why two layers?** Single filtered noise sounds like radio static. Two layers at different cutoff frequencies (2000Hz + 4000Hz) create a more natural steam sound — low whoosh + high sizzle.

### blackoutAlarm()
```javascript
this.sweep(800,200,0.4,'square',0.15);          // Descending sweep — "power down" feel
this.d(()=>this.sweep(200,800,0.3,'square',0.12),200);  // Ascending sweep — "alarm rising"
this.d(()=>this.n(0.15,0.2),100);                       // Noise burst — impact
```
**Why the up-down sweep pattern?** The descending sweep (800→200Hz) communicates "power failing." The ascending sweep (200→800Hz) is the alarm kicking in. The noise burst adds physical impact. Together they read as: lights die → alarm triggers → something broke.

---

## WAVE SPAWNING (Mobile Enemies)

The existing zone-based wave spawning already handles all 4 zones:

```javascript
// Reactor and Sentinel share the "else" block:
etype = r < 0.3 ? 'soldier' : r < 0.65 ? 'charger' : 'sniper';
// 30% soldier, 35% charger, 35% sniper
```

**Why heavier charger/sniper mix in later zones?** Soldiers are the baseline threat — they walk and shoot. Chargers at 2.2× speed create panic in tight corridors. Snipers are stationary but shoot at half the normal interval. By Reactor/Sentinel, the player has mastered basic soldier combat, so the wave composition shifts to faster and more accurate enemies.

**Behind-spawning (10% in Stage 2):** `if(currentStage===2 && Math.random() < 0.1) x = camera.x - 40`. Only 10% because higher rates feel cheap — the player can't watch both directions constantly. 10% is enough to keep them checking their back occasionally without punishing forward progress.

---

## WHAT'S NOT YET BUILT (Remaining ~45%)

After Phase A+B, the remaining work is:

1. **Alien Warrior enemy** — 3HP, charges+leaps, mini-boss class. Appears 2-3 times in Reactor/Sentinel zones. Needs new sprite (from NES sheet), charge AI, leap arc, mid-air projectile throw.

2. **Shield Trooper enemy** — 2HP, frontal energy shield blocks shots. Must hit from behind, above, or with grenades. Procedural sprite (soldier + blue shield bar). Forces positional play.

3. **The Sentinel boss** — 3-phase walking mech. Phase 1: dual arm cannons (5HP each). Phase 2: stomp shockwaves + mini-drones. Phase 3: exposed core (20HP) + eye laser sweep. This is the biggest single feature remaining.

4. **Stage 2 BGM: Industrial Pulse** — ~140 BPM, sawtooth bass, metallic hi-hat 16th notes, industrial kick, synth stabs. Web Audio API synthesis like Stage 1's War Drums.

5. **Stage 2 stage-clear screen** — Update `renderStageClear()` to show "THE CORRIDOR COMPLETE" with Stage 2 specific text instead of "JUNGLE BREACH COMPLETE."

6. **End-of-level trigger for Stage 2** — Currently the level uses `fortressOpen` which is tied to Stage 1's boss. Stage 2 needs its own boss-defeat → `fortressOpen=true` flow through the Sentinel boss.

---

## CONVENTIONS FOR FUTURE AGENTS

1. **Always gate with `currentStage===2`** — Stage 1 code is LOCKED. Never modify it.
2. **All coords clamped with `Math.max(0, ...)`** — NEVER hardcode negative coordinates.
3. **Array removals: iterate backwards** — `for(let i=arr.length-1; i>=0; i--)`.
4. **Timers use `gameTime`** — NEVER `setTimeout` for gameplay logic (death/respawn/spawning).
5. **Validate sprite bounds** before every `ctx.drawImage()`.
6. **Test platform reachability** — max 50px vertical gap between connected platforms.
7. **Enemy viewport check** — ALL enemies must check they're on-screen before firing.
8. **Spacing consistency** — maintain ~200-300px between same-type elements.
9. **Show mockups BEFORE coding** visual changes (per rules.md).
10. **Single HTML file** — no build step, no external dependencies. All assets base64, all audio Web Audio API.

---

## TWO-FILE WORKFLOW (commit 5da0d90)

### Why two files?
Stage 1 is production-ready and played by real users. Stage 2 is under active development. Keeping them in separate files prevents WIP bugs from breaking the live game while allowing full testing of Stage 2 in isolation.

### Files
| File | `currentStage` | `updateStageClear()` | Purpose |
|------|---------------|---------------------|---------|
| `index.html` (GitHub) | `1` | Returns to title | Production — players only see Stage 1 |
| `breach-s2-wip.html` (local) | `2` | Stage 1 → advances to Stage 2 | Testing — devs test Stage 2 directly |

### Merge procedure
When Stage 2 testing is confirmed complete:
1. Copy `breach-s2-wip.html` → `index.html`
2. Change `let currentStage = 2` → `let currentStage = 1`
3. The WIP's `updateStageClear()` already has the correct auto-advance logic — no change needed
4. Commit with message describing Stage 2 merge

### What NOT to do
- Do NOT develop Stage 2 features in `index.html` — always work in the WIP file
- Do NOT set `currentStage=2` in production — Stage 2 won't have proper flow without the WIP's `updateStageClear()`
- Do NOT delete Stage 2 code from `index.html` — it's gated and harmless, and keeping it means the merge is just two line changes

---

## DIFFICULTY REBALANCE (commit 5da0d90)

### Problem
Player feedback: "even Easy feels extremely hard." Root causes:
1. `crawlerSpeed` was defined in DIFFICULTY but hardcoded at 1.2 in `updateEnemies()` — Easy/Normal/Hard crawlers all moved at same speed
2. `laserOffBonus` was defined but never applied — lasers had same timing on all difficulties
3. Steam vents had no difficulty scaling at all
4. Blackout spawned 6 crawlers regardless of difficulty
5. Base Easy numbers (7HP, 4 lives, 0.5 density) were still too aggressive for novices

### Fix: Wire + Rebalance
All difficulty properties now connected to game logic:

| Property | What it does | Where it's used |
|---|---|---|
| `crawlerSpeed` | Multiplied into crawler chase `e.vx` | `updateEnemies()` crawler block |
| `laserOffBonus` | Added to laser `off` time in cycle | `updateBill()` laser damage + `render()` laser visual |
| `steamCycleBonus` | Added to steam vent cycle time | `updateBill()` steam damage + `render()` steam visual |
| `blackoutCrawlers` | Controls crawler count during blackout | `updateBill()` blackout trigger |

### New values rationale (60% novice / 30% amateur / 10% expert)

**Easy (novices — majority of players):**
- 9HP + 5 lives = very forgiving. A novice can take ~45 hits across all lives before game over.
- `enemyDensity:0.35` = only 35% of fixed enemies spawn. In Reactor Core, instead of 9 ground enemies they face ~3.
- `bulletMul:0.35` = enemy bullets travel at 0.875 px/frame (base 2.5 × 0.35). Easily dodgeable by walking.
- `speedMul:0.5` = enemies move at half speed. Chargers at 2.2×0.5=1.1 speed — barely faster than the player's 2.0.
- `crawlerSpeed:0.6` = crawlers slower than the player. Can outrun them.
- `laserOffBonus:800` = adds 800ms to every laser off-window. The tightest laser (900ms off) becomes 1700ms — comfortable.
- `steamCycleBonus:1200` = steam vents are practically harmless. Tightest cycle (2600ms) becomes 3800ms safe window.
- `blackoutCrawlers:3` = only 3 crawlers in the dark. Manageable.

**Normal (amateurs — significant minority):**
- 6HP + 4 lives = moderate forgiveness (~24 total hits).
- 60% density, 55% bullet speed, 75% enemy speed.
- Crawlers at 0.9 = slightly slower than player.
- Laser bonus 400ms, steam bonus 500ms — noticeable help but not trivial.

**Hard (experts — 10% of players):**
- Unchanged from before. 3HP, 2 lives, full density, full speed. This is the Contra experience.
