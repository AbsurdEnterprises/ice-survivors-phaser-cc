# ICE Survivors — Phaser.js V1: Complete Agent Specification

## System Role

You are a Senior Game Developer specializing in Phaser 3 (latest stable) browser games. You will build a visually polished 2D bullet-heaven survival game — a single HTML file with embedded JavaScript that runs directly in any modern browser. No build tools, no bundler, no npm. One file. Load Phaser from CDN.

This is NOT a geometry prototype. V1 must look and feel like a real indie game. You will achieve this entirely through **procedural graphics** — drawing sprites at runtime onto offscreen canvases, using gradients, glow effects, particle emitters, screen shake, and a cohesive color palette. No external image assets are loaded.

---

## Visual Identity and Art Direction

### Color Palette

Every visual element in the game draws from this palette. Do not deviate.

| Role | Hex | Usage |
|------|-----|-------|
| Background Base | `#0B0E17` | Deep blue-black, cold winter night |
| Background Accent | `#151B2B` | Slightly lighter, for building silhouettes |
| Player Core | `#4FC3F7` | Bright ice-blue, player character fill |
| Player Glow | `#81D4FA` | Player outline glow, pickup pulse |
| Enemy Fodder | `#EF5350` | Low-tier enemies, warm red |
| Enemy Elite | `#AB47BC` | Mid-tier enemies, purple |
| Enemy Tank | `#FF7043` | Heavy enemies, deep orange |
| Enemy Ranged | `#66BB6A` | Ranged/botnet enemies, toxic green |
| Hazard | `#FFEE58` | Vehicles, environmental danger, bright yellow |
| Projectile Player | `#FFF176` | Player weapon projectiles, warm yellow-white |
| Projectile Enemy | `#E53935` | Enemy projectiles, hot red |
| XP Gem | `#69F0AE` | Experience pickups, mint green |
| XP Gem Rare | `#FFD54F` | High-value merged gems, gold |
| Health Bar Fill | `#66BB6A` | Green health |
| Health Bar Damage | `#EF5350` | Red when low |
| UI Text | `#ECEFF1` | HUD text, clean off-white |
| UI Accent | `#FFC107` | Gold accent for level-ups, evolutions |
| Freeze Effect | `#E1F5FE` | Frozen enemies tint |
| Heal Pulse | `#A5D6A7` | HP recovery flash |

### Procedural Sprite Generation

At game boot (in the `preload` or `create` phase), generate ALL game sprites by drawing to offscreen `Canvas` elements and converting them to Phaser textures via `game.textures.addCanvas()` or `textures.createCanvas()`. This runs once and produces every texture the game needs.

**Player sprite (32×32):** Draw a rounded rectangle body in `#4FC3F7` with a 2px `#81D4FA` glow outline. Add a small darker visor rectangle near the top. Add two small leg rectangles at the bottom. This creates a simple but readable humanoid silhouette.

**Fodder enemy (24×24):** Red (`#EF5350`) circle with a darker center dot and a 1px black outline. Slightly pulsate scale via a tween (0.95–1.05 over 400ms, yoyo, repeat forever) to give it a "breathing" feel.

**Elite enemy (28×28):** Purple (`#AB47BC`) hexagon shape. Draw 6-sided polygon path on the offscreen canvas. Add inner highlight line.

**Tank enemy (36×36):** Orange (`#FF7043`) square with an inner cross pattern (like body armor). Thicker 2px outline. Larger = reads as tougher.

**Ranged enemy (22×22):** Green (`#66BB6A`) diamond (rotated square). Small antenna line on top.

**Hazard vehicle (80×28):** Yellow (`#FFEE58`) elongated rectangle with darker stripe down center. Speed lines trailing behind via a particle emitter.

**Boss sprites:** 3× to 5× the size of standard enemies, with a visible inner pulsing glow (sine-wave alpha on an additive-blend inner shape).

**XP gem (10×10):** Mint green (`#69F0AE`) diamond. For rare/merged gems, use gold (`#FFD54F`) and make them 16×16.

**Projectiles:** Small (6×6 to 10×10) circles with radial gradients — bright center fading to transparent edge. This creates a natural glow-bullet look with zero effort.

### Particle Effects (Critical for Game Feel)

Phaser 3's built-in particle system is your best friend. Create these emitter configurations:

**Enemy death burst:** On enemy kill, emit 8–15 particles of the enemy's color, exploding outward with `speed: { min: 80, max: 200 }`, `lifespan: 300`, `alpha: { start: 1, end: 0 }`, `scale: { start: 0.5, end: 0 }`. This alone makes kills feel satisfying.

**XP gem sparkle:** Gems emit 1 tiny white particle every 200ms, `speed: 15`, floating upward, `lifespan: 400`. Makes gems visible in chaos.

**Player damage flash:** On hit, emit 6 red particles from player position outward, and tint the player sprite red for 100ms before starting i-frame blink.

**Level-up burst:** Large ring of 30 gold (`#FFC107`) particles expanding outward from player, `speed: 250`, `lifespan: 600`. Satisfying power moment.

**Evolution fanfare:** Same as level-up but double the particle count, plus a brief screen flash (white overlay alpha 0→0.6→0 over 300ms).

**Weapon trails:** For projectiles, add a 3-particle trail using `emitParticle()` each frame with small, fast-fading particles in the projectile's color.

### Screen Effects

**Screen shake:** `this.cameras.main.shake(duration, intensity)`.
- Player hit: `shake(100, 0.005)`
- Boss spawn: `shake(300, 0.01)`
- Screen nuke pickup: `shake(200, 0.015)`

**Slow-motion on level-up:** When pausing for level-up selection, don't instant-pause. Instead, lerp `game.time.timeScale` from 1.0 to 0.0 over 300ms, THEN show the UI. On resume, lerp back. This feels cinematic.

**Vignette overlay:** A persistent dark radial gradient overlay on the camera (drawn once as a large texture, added to the UI camera layer at low alpha ~0.3). Darkens screen edges, focuses attention on player. Extremely cheap, massive atmosphere upgrade.

**Snow particle layer:** A camera-fixed particle emitter dropping tiny white dots downward across the full viewport. `speedY: { min: 20, max: 60 }`, `speedX: { min: -10, max: 10 }`, `alpha: { min: 0.2, max: 0.5 }`, `lifespan: 8000`, `quantity: 1` every 100ms. Sets the Minnesota winter tone with nearly zero performance cost.

### HUD Design

Do NOT use Phaser's default text rendering for the main HUD. Instead:

**HP Bar:** A container at top-left. Dark grey (`#1A1A2E`) background rectangle, filled rectangle that shrinks from right. Color interpolates from green to yellow to red as HP drops. 2px border in `#ECEFF1`. Width: 200px, height: 16px.

**XP Bar:** Full-width bar at the very bottom of the screen. Thin (8px). Fill color: `#69F0AE`. Background: `#1A1A2E`. When XP is gained, briefly flash the bar brighter (tween alpha).

**Timer:** Top-center. Large, clean bitmap font (generate a bitmap font texture procedurally from canvas — draw digits 0-9 and colon in a monospace grid, white on transparent). Format: `MM:SS`. This timer is the heartbeat of the game.

**Kill counter:** Top-right. Same bitmap font. Just a number.

**Level indicator:** Next to HP bar. `"LV. XX"` in the bitmap font.

**Weapon inventory:** Bottom-left. Row of 6 small (24×24) square slots. Empty slots: dark outline only. Filled slots: weapon icon (a tiny colored shape matching the weapon type). Show weapon level as a small number overlay.

**Passive inventory:** Bottom-right. Same as weapons but for passives.

---

## Technical Architecture

### Single-File Structure

The entire game lives in one HTML file. Structure the JavaScript using clearly separated class definitions and a boot sequence:

```
index.html
├── <style> — minimal CSS: black background, centered canvas, no scrollbars
├── <script src="phaser.min.js CDN">
├── <script>
│   ├── // === CONFIGURATION ===
│   │   ├── WEAPON_DATA (const object)
│   │   ├── PASSIVE_DATA
│   │   ├── EVOLUTION_DATA
│   │   ├── ENEMY_DATA
│   │   ├── CHARACTER_DATA
│   │   ├── BOSS_DATA
│   │   └── META_UPGRADE_DATA
│   │
│   ├── // === UTILITY CLASSES ===
│   │   ├── class ObjectPool
│   │   ├── class SpatialHashGrid
│   │   └── class ProceduralSprites (generates all textures)
│   │
│   ├── // === GAME ENTITIES ===
│   │   ├── class Player extends Phaser.GameObjects.Sprite
│   │   ├── class Enemy extends Phaser.GameObjects.Sprite
│   │   ├── class Projectile extends Phaser.GameObjects.Sprite
│   │   ├── class XPGem extends Phaser.GameObjects.Sprite
│   │   └── class Destructible extends Phaser.GameObjects.Sprite
│   │
│   ├── // === WEAPON CLASSES ===
│   │   ├── class WeaponBase
│   │   ├── class WeaponMeleeSweep extends WeaponBase
│   │   ├── class WeaponAutoTarget extends WeaponBase
│   │   ├── class WeaponBarrage extends WeaponBase
│   │   ├── class WeaponArcLob extends WeaponBase
│   │   ├── class WeaponAura extends WeaponBase
│   │   ├── class WeaponGroundPool extends WeaponBase
│   │   ├── class WeaponOrbit extends WeaponBase
│   │   ├── class WeaponRandomStrike extends WeaponBase
│   │   ├── class WeaponBounce extends WeaponBase
│   │   └── class WeaponFreezeBeam extends WeaponBase
│   │
│   ├── // === SCENES ===
│   │   ├── class BootScene extends Phaser.Scene
│   │   ├── class MenuScene extends Phaser.Scene
│   │   ├── class GameScene extends Phaser.Scene
│   │   ├── class LevelUpScene extends Phaser.Scene (overlay)
│   │   └── class GameOverScene extends Phaser.Scene
│   │
│   └── // === BOOT ===
│       └── new Phaser.Game(config)
```

### Phaser Configuration

```javascript
const config = {
    type: Phaser.AUTO,  // WebGL preferred, Canvas fallback
    width: 1280,
    height: 720,
    backgroundColor: '#0B0E17',
    parent: 'game-container',
    physics: {
        default: 'arcade',
        arcade: {
            gravity: { y: 0 },
            debug: false
        }
    },
    scene: [BootScene, MenuScene, GameScene, LevelUpScene, GameOverScene],
    scale: {
        mode: Phaser.Scale.FIT,
        autoCenter: Phaser.Scale.CENTER_BOTH
    },
    fps: {
        target: 60,
        forceSetTimeOut: false
    }
};
```

### Critical Performance Rules

1. **Object Pooling via Phaser Groups.** Use `this.physics.add.group({ classType: Enemy, maxSize: 500, runChildUpdate: true })` for enemies, projectiles, and XP gems. Kill/deactivate entities with `entity.setActive(false).setVisible(false)`. Reactivate from pool with `group.get()`. NEVER use `destroy()` during gameplay.

2. **Spatial Hash Grid.** Even though Phaser has built-in arcade physics overlap, a custom spatial hash is still required for efficient "find nearest enemy" queries (weapon targeting) and XP gem clustering. Divide world into 128×128 cells. Update entity registrations every frame in `update()`.

3. **Bitmap font for ALL dynamic text.** Generate a texture atlas of digits 0-9, letters A-Z, and symbols (colon, period, slash, percent) at boot time. Render all HUD text, damage numbers, and level-up text by compositing these character sprites. Never call `this.add.text()` in the game loop — Phaser's text objects trigger canvas re-rasterization and will destroy FPS at high entity counts.

4. **XP Gem Aggregation.** If active gem count exceeds 200, merge all into a single high-value gem at their centroid. Change its texture to the gold rare gem.

5. **Entity hard cap.** Max active enemies: 500 (initial target). Max active projectiles: 1000. Max active XP gems: 250 (before merge triggers). Max active particles: managed by Phaser's emitter — set `maxParticles` on each emitter config.

6. **Camera bounds.** The game world is effectively infinite. Use `this.cameras.main.startFollow(player)` with deadzone. Do NOT set world bounds — the player roams freely and enemies spawn relative to camera viewport.

---

## Core Gameplay Loop

Each run lasts exactly **30 minutes**. Every frame in `GameScene.update(time, delta)`:

1. Process player input → 8-directional movement (WASD + arrow keys)
2. Camera follows player (automatic via Phaser `startFollow`)
3. Run enemy spawner: check spawn timer against N(t) formula, pull from pool, position on viewport ellipse
4. Update all active enemies: execute AI behavior script (move toward player, orbit, etc.)
5. Update all active weapons: check cooldown timers, fire if ready, manage active projectiles
6. Phaser arcade physics handles overlap detection for: projectile↔enemy, enemy↔player, player↔xp_gem, player↔pickup
7. Process damage: apply formula, trigger i-frames, spawn particles, update HP bar
8. Check XP threshold → launch LevelUpScene as overlay (pause GameScene)
9. Update HUD: HP bar width, XP bar fill, timer text, kill count
10. Check timer → spawn bosses at minute 10, 20, 30

---

## Mathematical Models

### Enemy Spawning Curve

```
N(t) = min(M_CAP, floor(B_S * (1 + r)^t + S(t)))
```

| Variable | Value | Description |
|----------|-------|-------------|
| `M_CAP` | 500 | Max concurrent active enemies |
| `B_S` | 8 | Base spawn count per cycle |
| `r` | 0.12 | Exponential growth rate |
| `t` | Minutes elapsed (float) | Time into run |

**Spawn cycle:** Every 1.0 seconds, calculate how many enemies should exist via N(t). If current active count < N(t), spawn the difference (capped at 15 per cycle to prevent frame spikes).

**Surge waves S(t):**
- t = 5.0 min → +40 enemies over 10 seconds (4/sec burst)
- t = 10.0 min → +80 over 10 seconds (Boss 1 simultaneous)
- t = 15.0 min → +120 over 10 seconds
- t = 20.0 min → +160 over 10 seconds (Boss 2 simultaneous)
- t = 25.0 min → +200 over 10 seconds
- t = 30.0 min → Final boss, standard spawning stops

**Spawn position:** Ellipse around player, just outside viewport:
```
θ = random(0, 2π)
spawn_x = player.x + (viewport_width/2 + 80) * cos(θ)
spawn_y = player.y + (viewport_height/2 + 80) * sin(θ)
```

### Enemy Health and Speed Scaling

```
H_e(t) = H_BASE * (1 + t * 0.15) * C_MOD
V_e(t) = V_BASE + (t * 0.02 * V_BASE)
```

| Enemy Class ID | `H_BASE` | `C_MOD` | `V_BASE` (px/s) | Contact Dmg | XP Value | Behavior |
|---------------|----------|---------|-----------------|-------------|----------|----------|
| `fodder_01` | 10 | 0.8 | 40 | 5 | 1 | Direct path to player |
| `erratic_02` | 15 | 1.0 | 55 | 8 | 2 | Path to player, random 0.5–2s pauses |
| `tank_03` | 40 | 5.0 | 35 | 15 | 10 | Direct path, immune to knockback |
| `ranged_04` | 20 | 1.2 | 25 | 12 (proj) | 5 | Orbits at 400px, fires inward every 3s |
| `hazard_05` | 999 | 1.0 | 300 | 50 | 0 | Straight line across screen, despawns at edge |

**Enemy class composition by time:**

| Time Range | fodder | erratic | tank | ranged | hazard |
|------------|--------|---------|------|--------|--------|
| 0–3 min | 100% | 0% | 0% | 0% | 0% |
| 3–7 min | 60% | 35% | 0% | 5% | 0% |
| 7–12 min | 40% | 30% | 15% | 10% | 5% |
| 12–20 min | 25% | 25% | 25% | 15% | 10% |
| 20–30 min | 15% | 20% | 30% | 20% | 15% |

### Player Damage Formula

```
damage_dealt = BASE_DMG * (1 + (weapon_level - 1) * 0.25) * (1 + area_bonus) * crit_mult * (1 + global_dmg_bonus)
```

| Variable | Description |
|----------|-------------|
| `BASE_DMG` | Per-weapon (see weapon table) |
| `weapon_level` | 1–8 |
| `area_bonus` | Sum of character area modifier + passive_04 bonus + meta_03 bonus |
| `crit_mult` | 2.0 if crit triggers, else 1.0 |
| `crit_chance` | 0.05 + (luck * 0.01), capped at 0.50 |
| `global_dmg_bonus` | From passive_10 + character modifier |

**DPS verification (t=25 min):** A `tank_03` has HP = `40 * (1 + 25*0.15) * 5.0 = 950`. A Level 8 `weapon_02` deals `12 * 2.75 = 33 dmg` at 0.3s cooldown = 110 DPS. One weapon alone takes 8.6s per tank. With 4-5 weapons plus evolutions, total DPS reaches 500–800. The math works — solo weapons feel weak (intended), combinations feel powerful (intended).

### Player Damage Intake

```
damage_taken = max(1, raw_damage - player_armor)
```

Player armor comes from: character base + passive_09 + meta_01. Minimum damage is always 1.

### Invincibility Frames

After taking damage: 0.5 seconds of invulnerability. During i-frames:
- `player.isInvulnerable = true`
- Toggle `player.alpha` between 0.3 and 1.0 every 50ms (blink effect)
- Player can still move, collect XP, deal damage
- On i-frame end: reset alpha to 1.0, set flag false

### Experience and Leveling

```
XP_required(L) = floor(10 * L^1.5 + 50)
```

| Level | XP Needed | Cumulative |
|-------|-----------|------------|
| 1→2 | 60 | 60 |
| 5→6 | 162 | 640 |
| 10→11 | 366 | 2,206 |
| 20→21 | 944 | 9,778 |

### Level-Up Selection

Pause GameScene (with slow-mo transition). Launch `LevelUpScene` as overlay.

Display **4 options** (3 at Level 1). Each option is a card — dark rectangle (`#1A1A2E`) with colored border matching item type (gold for weapon, cyan for passive), containing the item's icon and a short description.

**Selection weighting:**
- New item (player doesn't own): weight = `10 + (luck * 2)`
- Upgrade to existing item: weight = `20`
- Rare passives (passive_05, passive_06): weight = `5 + (luck * 3)`

Draw without replacement.

**Visual:** Cards slide in from the bottom with a staggered animation (100ms delay each). Selected card scales up briefly, unselected cards fade and slide down. Gold particle burst on selection.

### Treasure Chest Logic

Bosses drop a treasure chest on death. Chest is a pulsing gold rectangle with a glow. On player contact:

1. Check evolution table: does player have any Level 8 weapon + required passive(s)?
2. **Single eligible evolution:** Force it. Remove base weapon from inventory, inject evolution. Passive stays. Play evolution fanfare (particles + screen flash + 0.5s slow-mo).
3. **Multiple eligible:** Show selection screen (same as level-up but with evolution cards only).
4. **None eligible:** Award 300 gold + full HP heal + 10 seconds of invulnerability. Heal pulse visual.

---

## Data Schemas

### Weapon Definitions

```javascript
const WEAPON_DATA = {
    weapon_01: {
        type: 'melee_sweep', baseDmg: 15, cooldown: 1.1, area: 1.0,
        knockback: 80, projectileCount: 1, pierce: 999, maxLevel: 8,
        desc: 'Horizontal sweep in facing direction',
        color: '#FFF176', // visual tint for projectile/effect
        levelBonuses: { damage: 0.25, area: 0.05, cooldown: -0.05 }
    },
    weapon_02: {
        type: 'auto_target', baseDmg: 12, cooldown: 0.3, area: 0.5,
        knockback: 10, projectileCount: 1, pierce: 1, maxLevel: 8,
        desc: 'Fires projectile at nearest enemy',
        color: '#81D4FA',
        levelBonuses: { damage: 0.25, projectileCount: 1, cooldown: -0.02 }
        // projectileCount bonus: +1 at levels 3, 5, 7 (checked manually)
    },
    weapon_03: {
        type: 'directional_barrage', baseDmg: 8, cooldown: 0.25, area: 0.3,
        knockback: 30, projectileCount: 3, pierce: 1, maxLevel: 8,
        desc: 'Fires burst in movement direction',
        color: '#CE93D8',
        levelBonuses: { damage: 0.25, projectileCount: 1, cooldown: -0.01 }
    },
    weapon_04: {
        type: 'arcing_lob', baseDmg: 22, cooldown: 1.8, area: 1.2,
        knockback: 40, projectileCount: 1, pierce: 3, maxLevel: 8,
        desc: 'Arcing projectile that falls through enemies',
        color: '#FFAB91',
        levelBonuses: { damage: 0.25, area: 0.08, projectileCount: 0.5 }
    },
    weapon_05: {
        type: 'persistent_aura', baseDmg: 5, cooldown: 0.5, area: 1.5,
        knockback: 60, projectileCount: 0, pierce: 999, maxLevel: 8,
        desc: 'Constant damage zone around player',
        color: '#A5D6A7',
        levelBonuses: { damage: 0.25, area: 0.1, knockback: 0.1 }
    },
    weapon_06: {
        type: 'ground_pool', baseDmg: 18, cooldown: 3.0, area: 1.8,
        knockback: 5, projectileCount: 1, pierce: 999, maxLevel: 8,
        desc: 'Drops damaging zone at random nearby position, persists 3s',
        color: '#FFE082',
        levelBonuses: { damage: 0.25, area: 0.1, cooldown: -0.15 }
    },
    weapon_07: {
        type: 'orbiting', baseDmg: 10, cooldown: 0.0, area: 0.8,
        knockback: 50, projectileCount: 3, pierce: 999, maxLevel: 8,
        desc: 'Projectiles orbit player, block and damage enemies',
        color: '#80DEEA',
        levelBonuses: { damage: 0.25, projectileCount: 1, area: 0.05 }
    },
    weapon_08: {
        type: 'random_strike', baseDmg: 30, cooldown: 2.0, area: 1.0,
        knockback: 20, projectileCount: 1, pierce: 999, maxLevel: 8,
        desc: 'Explosive strike at random enemy location',
        color: '#FFF9C4',
        levelBonuses: { damage: 0.25, area: 0.08, cooldown: -0.1 }
    },
    weapon_09: {
        type: 'bouncing', baseDmg: 14, cooldown: 2.5, area: 0.6,
        knockback: 15, projectileCount: 1, pierce: 999, maxLevel: 8,
        desc: 'Bounces off screen edges infinitely, passes through enemies',
        color: '#B0BEC5',
        levelBonuses: { damage: 0.25, speed: 0.1, projectileCount: 0.5 }
    },
    weapon_10: {
        type: 'freeze_beam', baseDmg: 0, cooldown: 0.0, area: 2.0,
        knockback: 0, projectileCount: 1, pierce: 999, maxLevel: 8,
        desc: 'Rotating beam that freezes enemies for 2s',
        color: '#E1F5FE',
        levelBonuses: { area: 0.1, duration: 0.15 }
    }
};
```

### Passive Item Definitions

```javascript
const PASSIVE_DATA = {
    passive_01: { stat: 'maxHp', bonusPerLevel: 0.10, maxLevel: 5, desc: '+10% max HP', color: '#EF5350' },
    passive_02: { stat: 'cooldownReduction', bonusPerLevel: 0.08, maxLevel: 5, desc: '-8% cooldown', color: '#42A5F5' },
    passive_03: { stat: 'projectileSpeed', bonusPerLevel: 0.10, maxLevel: 5, desc: '+10% projectile speed', color: '#FFA726' },
    passive_04: { stat: 'area', bonusPerLevel: 0.10, maxLevel: 5, desc: '+10% AoE size', color: '#AB47BC' },
    passive_05: { stat: 'hpRegen', bonusPerLevel: 0.3, maxLevel: 5, desc: '+0.3 HP/s', color: '#66BB6A' },
    passive_06: { stat: 'xpRadius', bonusPerLevel: 0.20, maxLevel: 5, desc: '+20% XP pickup radius', color: '#26C6DA' },
    passive_07: { stat: 'effectDuration', bonusPerLevel: 0.10, maxLevel: 5, desc: '+10% effect duration', color: '#7E57C2' },
    passive_08: { stat: 'luck', bonusPerLevel: 0.10, maxLevel: 5, desc: '+10% luck', color: '#FFD54F' },
    passive_09: { stat: 'armor', bonusPerLevel: 1.0, maxLevel: 5, desc: '+1 flat armor', color: '#78909C' },
    passive_10: { stat: 'damageBonus', bonusPerLevel: 0.05, maxLevel: 5, desc: '+5% global damage', color: '#FF7043' }
};
```

### Evolution Lookup Table

```javascript
const EVOLUTION_DATA = {
    evo_01: { weapon: 'weapon_01', passive: 'passive_01', replaces: 'weapon_01',
              effect: 'Crits always. Heals 5% of damage dealt.', color: '#FF8A80' },
    evo_02: { weapon: 'weapon_02', passive: 'passive_02', replaces: 'weapon_02',
              effect: 'Continuous piercing beam, zero cooldown.', color: '#82B1FF' },
    evo_03: { weapon: 'weapon_03', passive: 'passive_03', replaces: 'weapon_03',
              effect: 'Continuous barrage, massive knockback.', color: '#EA80FC' },
    evo_04: { weapon: 'weapon_04', passive: 'passive_04', replaces: 'weapon_04',
              effect: 'Ring of projectiles in all directions, full pierce.', color: '#FF9E80' },
    evo_05: { weapon: 'weapon_05', passive: 'passive_05', replaces: 'weapon_05',
              effect: 'Massive aura steals HP. Scales with missing HP.', color: '#B9F6CA' },
    evo_06: { weapon: 'weapon_06', passive: 'passive_06', replaces: 'weapon_06',
              effect: 'Pools drift toward player, merge into mega-pool.', color: '#FFE57F' },
    evo_07: { weapon: 'weapon_07', passive: 'passive_07', replaces: 'weapon_07',
              effect: '8 permanent orbitals, never despawn.', color: '#84FFFF' },
    evo_08: { weapon: 'weapon_08', passive: 'passive_08', replaces: 'weapon_08',
              effect: 'Double strike same location, second hit 3x damage.', color: '#FFFF8D' },
    evo_09: { weapon: 'weapon_09', passive: 'passive_09', replaces: 'weapon_09',
              effect: 'Explodes on every bounce, AoE + pierce.', color: '#CFD8DC' },
    evo_10: { weapon: 'weapon_10', passive: ['passive_10', 'passive_09'], replaces: 'weapon_10',
              effect: 'Freezes all on-screen enemies. Halves all HP each rotation.', color: '#E1F5FE' }
};
```

### Character Definitions

```javascript
const CHARACTER_DATA = {
    char_01: {
        name: 'The Resilient', startWeapon: 'weapon_05',
        stats: { maxHp: 120, moveSpeed: 135, xpRadius: 64, armor: 0, luck: 0, area: 1.0, damage: 1.0 },
        color: '#A5D6A7', desc: 'High HP, slow. Starts with aura.'
    },
    char_02: {
        name: 'The Observer', startWeapon: 'weapon_02',
        stats: { maxHp: 100, moveSpeed: 150, xpRadius: 64, armor: 0, luck: 0, area: 1.3, damage: 0.9 },
        color: '#81D4FA', desc: 'Huge AoE, weak damage. Starts with auto-target.'
    },
    char_03: {
        name: 'The Official', startWeapon: 'weapon_01',
        stats: { maxHp: 100, moveSpeed: 172, xpRadius: 64, armor: -1, luck: 0, area: 1.0, damage: 1.0 },
        color: '#CE93D8', desc: 'Fast but fragile. Starts with sweep.'
    },
    char_04: {
        name: 'The Runner', startWeapon: 'weapon_03',
        stats: { maxHp: 80, moveSpeed: 210, xpRadius: 64, armor: 0, luck: 0, area: 1.0, damage: 1.0 },
        color: '#FFE082', desc: 'Blazing speed, glass cannon. Starts with barrage.'
    },
    char_05: {
        name: 'The Lucky', startWeapon: 'weapon_08',
        stats: { maxHp: 100, moveSpeed: 150, xpRadius: 64, armor: 0, luck: 20, area: 1.0, damage: 1.0 },
        color: '#FFD54F', desc: 'High luck, better drops. Starts with random strike.'
    },
    char_06: {
        name: 'The Immovable', startWeapon: 'weapon_07',
        stats: { maxHp: 100, moveSpeed: 105, xpRadius: 64, armor: 5, luck: 0, area: 1.0, damage: 1.0 },
        color: '#78909C', desc: 'Heavy armor, crawling speed. Starts with orbitals.'
    }
};
```

### Boss Definitions

```javascript
const BOSS_DATA = {
    boss_01: {
        spawnMinute: 10, hpMult: 10, speed: 60, size: 96,
        behavior: 'drone_deployer', color: '#AB47BC',
        desc: 'Spawns 3 fast drones (30HP, speed 200) every 4 seconds. Lock camera.',
        drops: 'treasure_chest'
    },
    boss_02: {
        spawnMinute: 20, hpMult: 20, speed: 40, size: 128,
        behavior: 'aoe_bombarder', color: '#FF7043',
        desc: 'Fires AoE blast every 3s. Blast travels to target, explodes 96px radius, leaves fire 4s at 5 DPS. Lock camera.',
        drops: 'treasure_chest'
    },
    boss_final: {
        spawnMinute: 30, hpFormula: 'playerLevel * 655350', speed: 999, size: 200,
        behavior: 'death_wall', color: '#F44336',
        desc: 'Unkillable. Faster than max player speed. Instant kill. Ends the run.'
    }
};
```

Boss HP (except final): `500 * (1 + minutesElapsed * 0.15) * hpMult`

### Meta Progression

```javascript
const META_UPGRADE_DATA = {
    meta_01: { stat: 'armor', bonusPerLevel: 1, maxLevel: 5, baseCost: 200, desc: '+1 armor' },
    meta_02: { stat: 'cooldownReduction', bonusPerLevel: 0.03, maxLevel: 5, baseCost: 300, desc: '-3% cooldown' },
    meta_03: { stat: 'area', bonusPerLevel: 0.05, maxLevel: 5, baseCost: 250, desc: '+5% AoE' },
    meta_04: { stat: 'revive', bonusPerLevel: 1, maxLevel: 1, baseCost: 5000, desc: 'Auto-revive once per run at 50% HP' }
};
// Cost formula: baseCost * (1.5 * levelToPurchase)
// Level 1 of meta_01 = 200 * 1.5 = 300 gold
// Level 3 of meta_01 = 200 * 4.5 = 900 gold
```

Save to `localStorage`:
```javascript
const saveData = {
    gold: 0,
    metaLevels: { meta_01: 0, meta_02: 0, meta_03: 0, meta_04: 0 },
    unlockedCharacters: ['char_01'],
    unlockedStages: ['stage_01'],
    bestTime: 0,
    totalKills: 0
};
```

---

## Destructible Objects

Spawn at random world positions. Density: ~1 per 512×512 area around player. Render as small grey rectangles with darker outlines.

| Object | HP | Drops |
|--------|-----|-------|
| prop_small | 5 | 50% nothing, 30% small_heal (15 HP), 15% gold (1-3), 5% screen_nuke |
| prop_large | 10 | 40% nothing, 25% medium_heal (30 HP), 20% gold (2-5), 10% magnet, 5% screen_nuke |

**Magnet:** All XP gems on map fly to player over 2 seconds. Use Phaser tween on each gem.
**Screen Nuke:** Kill all non-boss enemies on screen. White flash overlay (alpha 0→0.7→0, 200ms). Screen shake.

---

## Stage Design

### Stage 1: Urban Avenue (default)

- Horizontally infinite, vertically constrained to 600px band
- Background: procedurally draw building silhouettes (dark rectangles of varying height) along top and bottom edges. Parallax scroll at 0.3x player speed for depth.
- Ground: slightly lighter than background (`#1A1A2E`) with periodic lighter lines (sidewalk cracks)
- Snow overlay particle system active
- Hazard: every 45 seconds, a `hazard_05` vehicle crosses horizontally

### Stage 2: Mall (unlock after first boss kill)

- Infinite in X and Y
- Background: dark floor with grid pattern (faint lines every 256px)
- Static rectangular obstacle kiosks: 64×64 blocks in 8×8 grid with 256px spacing. These have physics bodies — they block movement and projectiles.
- Escalator zones: colored rectangles that apply forced movement vector when player overlaps

### Stage 3: Wilderness (unlock after surviving past minute 20)

- Infinite in X and Y
- Background: dark green-black (`#0B1A0B`)
- Dense procedural tree placement using Poisson disc sampling (min distance 96px). Trees are green circles (various sizes 24-48px) with darker outlines, impassable collision bodies.
- Enemies spawn from nearest tree cluster edge, not screen edge — creates ambush feel.

---

## Main Menu Design

The menu is its own Phaser Scene (`MenuScene`). Visual style:

- Same dark background with snow particle overlay
- Game title rendered large using the bitmap font, centered, with a subtle bobbing tween (y +-3px, 2s loop)
- Character select: 6 cards arranged in a row. Each card shows the character's colored shape, name, starting weapon icon, and stat summary. Selected card has glowing border.
- Below characters: "START RUN" button — bright gold rectangle with text, pulse animation
- Right side: Meta-upgrade shop. List of upgrades with current level, cost, and buy button. Grey out if insufficient gold.
- Gold counter displayed prominently
- Bottom: Stage select (locked stages show a lock icon)

---

## Development Phases — Execute In Strict Order

### PHASE 1: Boilerplate, Procedural Sprites, and Player Movement
- Create the single HTML file structure with Phaser CDN import
- Implement `ProceduralSprites` class: generate ALL textures (player, all enemy types, projectile, xp gem variants, destructibles, chest, pickup icons) at boot time using offscreen canvas drawing
- Generate bitmap font texture (digits, letters, colon, period)
- Implement `BootScene`: generate textures, then transition to `GameScene` (skip menu for now)
- Implement `GameScene.create()`: place player sprite, set up camera follow
- Implement 8-directional movement (WASD + arrows) in `GameScene.update()`
- Implement infinite scrolling background for Stage 1 (building silhouettes, parallax)
- Add snow particle overlay
- Add vignette overlay
- **Test:** Player moves smoothly, camera follows, background scrolls with parallax, snow falls. Should already look atmospheric.

### PHASE 2: Enemy Pool, Spawning, and Basic AI
- Implement `ObjectPool` class (or use Phaser physics group with maxSize)
- Create enemy physics group with pool of 500
- Implement `enemySpawner`: timer-based, calculates N(t), spawns on viewport ellipse
- Implement `fodder_01` AI: each frame, move toward player position
- Implement `erratic_02` AI: same but with random pause timer
- Add enemy "breathing" scale tween on activation
- Implement enemy death: deactivate, return to pool, spawn death particle burst
- **Test:** Enemies spawn at correct rates, converge on player, die when... well they can't die yet. Just verify spawning, movement, pooling, and particle death (trigger death manually in console for now).

### PHASE 3: Collision, Damage, HP, and I-Frames
- Set up Phaser arcade overlap: enemy↔player
- Implement player HP system, damage intake formula
- Implement i-frames (0.5s, alpha blink)
- Implement player damage flash (red tint + particles)
- Build HUD: HP bar (top-left), timer (top-center), kill counter (top-right) using bitmap font
- Implement player death → transition to `GameOverScene` (simple "GAME OVER" text + restart button for now)
- **Test:** Enemies damage player on contact, i-frames prevent stacking, HP bar updates, death triggers game over.

### PHASE 4: XP System and Level-Up
- Create XP gem physics group with pool of 300
- Implement gem drop on enemy death (spawn from pool at enemy position)
- Implement player↔gem overlap collection
- Implement XP bar (bottom of screen)
- Implement XP(L) formula and level tracking
- Implement gem aggregation (merge when >200 active)
- Implement gem sparkle particles
- Build `LevelUpScene`: overlay that pauses GameScene
- Implement slow-mo transition (timeScale lerp)
- Implement weighted random selection algorithm
- Implement card UI: 4 options with slide-in animation, descriptions, click/key selection
- Implement selection particle burst
- **Test:** Kill enemies → gems drop → collect → bar fills → level up → slow-mo → cards appear → select → game resumes with item.

### PHASE 5: Weapon System (Core)
- Implement `WeaponBase` class: cooldown timer, level tracking, damage calculation
- Create projectile physics group with pool of 1000
- Implement `WeaponAutoTarget` (weapon_02): find nearest enemy via spatial hash, fire projectile with trail particles
- Implement `WeaponAura` (weapon_05): persistent Area2D around player, damage tick
- Implement `WeaponMeleeSweep` (weapon_01): arc hitbox in facing direction
- Implement projectile↔enemy overlap: deal damage, apply knockback, trigger enemy damage flash
- Implement floating damage numbers using bitmap font sprites (pooled, float upward, fade out over 0.5s)
- Implement weapon leveling: apply per-level stat bonuses
- **Test:** Each implemented weapon fires correctly, damages enemies with visible numbers, levels up when selected.

### PHASE 6: Remaining Weapons
- Implement `WeaponBarrage` (weapon_03): multi-projectile burst in movement direction
- Implement `WeaponArcLob` (weapon_04): parabolic trajectory, passes through enemies on descent
- Implement `WeaponGroundPool` (weapon_06): random position near player, persistent damage zone with visual (colored circle that pulses)
- Implement `WeaponOrbit` (weapon_07): projectiles orbit at fixed radius using sin/cos
- Implement `WeaponRandomStrike` (weapon_08): pick random enemy position, instant AoE flash + damage
- Implement `WeaponBounce` (weapon_09): bounce off viewport-relative bounds, pierce enemies
- Implement `WeaponFreezeBeam` (weapon_10): rotating beam from player center, applies freeze status (enemy speed = 0, tint blue for 2s)
- **Test:** All 10 weapons function. Equip each one individually and verify behavior, damage, and visuals.

### PHASE 7: Passive Items and Inventory
- Implement passive item system: when selected at level-up, add to passive inventory array (max 6)
- Implement stat application: recalculate player effective stats whenever passive is added or leveled
- Implement weapon inventory display (bottom-left HUD)
- Implement passive inventory display (bottom-right HUD)
- Cap enforcement: if player has 6 weapons, do not offer new weapons at level-up (only upgrades to existing). Same for passives.
- **Test:** Select passives, verify stat changes (e.g., pick passive_01, confirm HP increases). Verify inventory display updates.

### PHASE 8: Evolution System
- Implement evolution lookup check
- Implement treasure chest entity: pulsing gold sprite, dropped at boss death position
- Implement player↔chest overlap trigger
- Implement evolution logic: check table, remove base weapon, inject evolution with upgraded stats and behavior
- Implement chest fallback (no eligible evo → gold + heal + invuln)
- Implement evolution fanfare (double particles + screen flash + slow-mo)
- Implement at least `evo_02` (continuous beam) and `evo_05` (HP steal aura) fully
- Stub remaining evolutions with 2x stat multiplier on the base weapon as placeholder
- **Test:** Level weapon_02 to 8, have passive_02, spawn a test chest (console command), open it → evolution triggers with fanfare.

### PHASE 9: Boss Encounters
- Implement boss entity class: large sprite, HP bar displayed below them, unique AI
- Implement camera lock on boss spawn (stop follow, set bounds to arena rectangle around current position)
- Implement `boss_01` (drone deployer): spawn mini-drone enemies from pool every 4s, move slowly toward player
- Implement `boss_02` (AoE bombarder): fire slow projectile to target location, explode, leave fire zone
- Implement `boss_final` (death wall): unkillable, faster than player, instant kill, run ender
- Implement boss spawn at correct minutes (10, 20, 30) with screen shake + warning text
- Implement chest drop on boss_01 and boss_02 death
- **Test:** Survive to minute 10, boss spawns, fight it, chest drops, continue. Verify minute 20 and 30 bosses.

### PHASE 10: Remaining Enemy Types and Composition
- Implement `tank_03`: direct path, knockback immunity flag, larger orange sprite
- Implement `ranged_04`: orbit behavior (maintain 400px distance, strafe around player), fire projectile at player every 3s
- Implement `hazard_05`: straight-line cross at 300px/s, no pooling (create/destroy), yellow vehicle sprite
- Implement enemy composition table: spawner selects enemy class based on weighted random using time bracket
- **Test:** Play to minute 20+, verify enemy mix shifts correctly. Tanks feel tanky, ranged enemies orbit, vehicles cross dangerously.

### PHASE 11: Destructibles, Pickups, and Stage Hazards
- Implement destructible objects: small grey rectangles with HP, spawned procedurally around player as they explore
- Implement drop table on destruction
- Implement heal pickup (green cross icon, restores HP)
- Implement gold pickup (gold coin icon, adds to run gold counter)
- Implement magnet pickup (tweens all gems to player)
- Implement screen nuke pickup (kill all non-boss enemies, white flash, shake)
- Implement Stage 1 periodic vehicle hazard
- **Test:** Break objects, collect pickups, verify effects.

### PHASE 12: Character Select and Meta-Progression
- Implement `MenuScene`: full main menu with character select, meta-upgrade shop, stage select
- Implement character stat application on run start
- Implement gold persistence via `localStorage`
- Implement meta-upgrade purchases with cost formula
- Implement meta-upgrade stat application at run start
- Implement `meta_04` auto-revive: on death, if purchased, restore 50% HP + 3s invuln instead of game over. Mark as consumed for this run.
- Implement unlock tracking: char_01 available by default, others unlock based on achievements (first boss kill, survive 20 min, etc.)
- Implement `GameOverScene`: show run stats (time survived, kills, gold earned, XP collected), "Return to Menu" button
- **Test:** Full loop: menu → select character → play run → die → game over screen → return to menu → buy upgrades → start new run with boosted stats.

### PHASE 13: Stages 2 and 3
- Implement Stage 2 (Mall): infinite grid, kiosk obstacles with physics bodies, escalator force zones
- Implement Stage 3 (Wilderness): Poisson disc tree generation, tree collision bodies, enemy spawn from tree edges
- Implement stage selection in menu
- **Test:** Each stage plays differently. Kiosks block movement/projectiles in Stage 2. Trees create maze in Stage 3.

### PHASE 14: Final Polish
- Screen shake on all appropriate events (hit, boss spawn, nuke, evolution)
- Verify all particle effects fire correctly
- Add boss warning text ("WARNING" flashing in bitmap font 2 seconds before boss spawn)
- Add run timer reaching 30:00 → final boss spawn feels climactic
- Kill counter feels satisfying (brief scale pop on increment)
- Verify frame rate stays above 50 FPS with 400+ enemies active (check with browser dev tools performance tab)
- If FPS drops below target, reduce particle counts and max enemy cap
- Verify `localStorage` persistence across page refreshes
- Final playthrough: complete a full 30-minute run to verify pacing, balance, and no crashes

---

## Final Notes for the Agent

- **This is a SINGLE HTML FILE.** Everything — all JavaScript classes, all data, all scenes — lives in one file between `<script>` tags. Phaser is loaded from CDN. There are no external assets.
- **All sprites are procedurally generated at boot.** Draw to offscreen canvases, convert to Phaser textures. This is non-negotiable.
- **Test each phase before proceeding.** Log entity counts, pool utilization, and frame timing to console.
- **The bitmap font system is critical.** Never use `this.add.text()` in the game loop. Generate the font atlas at boot and composite character sprites for all dynamic text.
- **All numeric values in this document are authoritative.** Implement formulas exactly. Do not round, approximate, or invent new values.
- **Pool sizes are starting points.** If you hit the cap during testing, increase the pool size in the config, not by creating new instances at runtime.
- **Use `delta` for all movement and timers.** Never assume a fixed frame rate. All movement: `entity.x += speed * (delta / 1000)`. All cooldowns: decrement by `delta / 1000`.
- **The game must be playable and fun at the end of Phase 14.** Not a tech demo. A game someone would actually play for 30 minutes.
