# Weapon System [SYS-002]

**Document ID:** GDD-SYS-002
**Version:** 1.0.0
**Status:** Draft
**Author:** Design Team
**Last Updated:** 2026-05-05
**Dependencies:** [SYS-001] Movement System, [SYS-003] Health & Damage
**Cross-References:** `health_and_damage.md`, `addon_system.md`, `../06_cross_cutting/tuning_constants.md`

---

## 1. Design Philosophy

The weapon system is the primary vector of player agency in combat. Every design decision below serves the project's core pillars:

1. **Three weapon types, three skill profiles.** Hitscan rewards precision and tracking. Projectile rewards prediction and spatial awareness. Beam rewards sustained focus and resource management. No type is objectively superior — each dominates a different engagement window.

2. **Low TTK, high readability.** Weapons are lethal. Engagements end in fractions of a second. This is intentional. To keep fights fair, every weapon's behavior must be visually and audibly legible to both the user and the target. If a player dies, they should be able to identify why within one replay frame.

3. **Loadout constraint breeds identity.** Players select exactly **1 primary weapon + 1 secondary weapon** from a shared pool. The restriction forces a meaningful choice. A player's loadout is their combat identity for that life.

4. **No trash weapons.** Every weapon must be the correct answer at its intended range and engagement profile. If a weapon is never the right pick, it is a design failure, not a player failure. Balance targets viability, not uniformity.

5. **Satisfying feedback, not visual noise.** Weapon effects must feel impactful without cluttering the viewport or obscuring enemy movement. This aligns with Pillar 4 — visual clarity is non-negotiable.

---

## 2. Weapon Type Definitions

### 2.1 Hitscan [W-CLASS-001]

**Mechanism:** On fire input, an instantaneous raycast is traced from the camera center along the look vector. The ray terminates at the first entity hit (player, destructible object) or the first world surface. Damage is applied immediately. There is no travel time — the shot connects or misses in the same frame.

**Netcode Model:** Server-authoritative with lag compensation via the Lightyear built-in rewind system. When the server receives a fire command, it rewinds the world state to the client's view at the timestamp of the shot, then performs the raycast. This ensures that what the player aimed at is what the server evaluates. Approved hit results are broadcast to all clients. Rejected hits are silently discarded — no visual correction on the shooter's screen beyond the absence of a hit marker.

**Damage Application:** Single-instance per fire event. Damage is calculated once per raycast. Headshot detection is determined by the hit collider tag (see `health_and_damage.md` for collider group definitions).

**Parameter Set:**

| Parameter | Type | Unit | Description |
|---|---|---|---|
| `damage_per_shot` | float | HP | Base damage on body hit |
| `fire_rate` | int | RPM | Rounds per minute |
| `headshot_multiplier` | float | multiplier | Applied to `damage_per_shot` on head collider hit |
| `falloff_start` | float | meters | Distance where damage begins to decrease |
| `falloff_end` | float | meters | Distance where damage reaches minimum |
| `min_damage` | float | HP | Floor damage at or beyond `falloff_end` |
| `magazine_size` | int | rounds | Rounds per full magazine |
| `reload_time` | float | seconds | Full reload duration |
| `spread_hip` | float | degrees | Maximum angular deviation from center, hip-fire stance |
| `spread_ads` | float | degrees | Maximum angular deviation from center, aimed stance |
| `recoil_vertical` | float | degrees | Vertical kick per shot |
| `recoil_horizontal` | float | degrees | Maximum horizontal drift per shot (randomized sign) |
| `recoil_recovery` | float | degrees/sec | Rate at which aim returns to center after firing stops |

**Tick Budget:** Each hitscan raycast must complete within **0.1ms** server-side. The raycast checks against the player collider set only (not full world geometry for damage purposes — world hit is a separate, cheaper check for decals only).

---

### 2.2 Projectile [W-CLASS-002]

**Mechanism:** On fire input, a physical entity is spawned at the weapon muzzle with an initial velocity vector aligned to the player's aim. The entity travels through the world over subsequent frames. Hit detection is performed by the Avian collision system. On contact with a player collider, the projectile detonates and applies damage. On contact with world geometry, the projectile detonates and applies splash damage to any players within radius.

**Netcode Model:** Client predicts the projectile spawn and renders it locally. The server spawns its own authoritative copy simultaneously. The server's copy is the source of truth for hit registration. If client and server disagree on a hit, the server wins. The client-side projectile is a visual ghost — it may show a hit that the server rejects, in which case the impact effect plays but no damage is dealt. This is an accepted trade-off for responsiveness.

**Gravity Behavior:** Each projectile defines a `projectile_gravity` value. At `0.0`, the projectile travels in a straight line (rockets, energy orbs). At `9.81`, the projectile follows a parabolic arc (grenades, mortars). Intermediate values are permitted for exotic weapons in future expansions.

**Parameter Set:**

| Parameter | Type | Unit | Description |
|---|---|---|---|
| `damage_direct` | float | HP | Damage on direct player hit |
| `splash_damage_max` | float | HP | Splash damage at explosion center |
| `splash_damage_min` | float | HP | Splash damage at splash radius edge |
| `splash_radius` | float | meters | Radius of splash damage zone |
| `splash_falloff` | enum | — | `LINEAR` or `INVERSE_SQUARE` |
| `projectile_speed` | float | m/s | Initial velocity magnitude |
| `projectile_gravity` | float | m/s² | Gravitational acceleration applied per frame |
| `fire_rate` | int | RPM | Rounds per minute |
| `magazine_size` | int | rounds | Projectiles per full magazine |
| `reload_time` | float | seconds | Full reload duration |
| `projectile_lifetime` | float | seconds | Max time before forced despawn |
| `bounce_count` | int | — | Number of world-surface bounces before detonation (0 = detonate on first contact) |

**Tick Budget:** Projectile position updates must complete within **0.05ms** per projectile per frame. Avian broadphase should keep active projectile checks under **0.2ms** total in worst-case (12 simultaneous projectiles).

---

### 2.3 Beam [W-CLASS-003]

**Mechanism:** While the fire input is held, a continuous raycast is traced from the weapon muzzle to the first obstructing surface or player entity. Damage is applied in discrete ticks at a fixed interval. Ammo is consumed continuously at a fixed rate. The beam is rendered as a visible line from the weapon to the impact point, visible to all players in line of sight.

**Netcode Model:** Server-authoritative. The client sends a `beam_start` message when fire is pressed and a `beam_end` message when fire is released. While active, the server performs its own raycast each tick to verify line of sight and target presence. If the server determines the beam should not be hitting (target moved behind cover, target died, etc.), that tick deals no damage. The client visually tracks the target and may display a beam hitting a target that the server has already broken — this is corrected on the next tick update.

**Overheat Mechanic:** Beams do not use traditional magazines. Instead, they consume from an energy pool and build heat. If the weapon is fired continuously past its `overheat_threshold`, the weapon enters a mandatory cooldown period during which it cannot fire. This prevents indefinite suppression and forces rhythm.

**Parameter Set:**

| Parameter | Type | Unit | Description |
|---|---|---|---|
| `damage_per_tick` | float | HP | Damage applied per tick event |
| `tick_rate` | int | ms | Milliseconds between damage applications |
| `range` | float | meters | Maximum beam reach |
| `ammo_per_tick` | float | units | Ammo consumed per tick event |
| `magazine_size` | float | energy units | Total energy pool |
| `overheat_threshold` | float | seconds | Continuous fire time before overheat triggers |
| `overheat_cooldown` | float | seconds | Mandatory wait time after overheat |
| `ramp_type` | enum | — | `NONE` (constant damage), `RAMP_UP` (increases over time on target), `RAMP_DOWN` (decreases over time) |
| `ramp_max_multiplier` | float | multiplier | Maximum damage multiplier at full ramp |
| `ramp_time` | float | seconds | Time to reach full ramp |
| `beam_width` | float | meters | Visual beam width (cosmetic only) |

**Tick Budget:** Beam raycast must complete within **0.1ms** per active beam per tick. Maximum 2 beams active simultaneously in standard play (one per player in a 1v1).

---

## 3. Damage Falloff Formula

All weapon types that support distance-based falloff (hitscan and beam) use the following formula. Projectile weapons use splash falloff instead (see §2.2).

```
func calculate_falloff(base_damage: float, min_damage: float,
                       falloff_start: float, falloff_end: float,
                       distance: float) -> float:
    if distance <= falloff_start:
        return base_damage
    elif distance >= falloff_end:
        return min_damage
    else:
        t = (distance - falloff_start) / (falloff_end - falloff_start)
        return base_damage - t * (base_damage - min_damage)
```

**Invariants:**
- `min_damage` is always >= 1.0. No weapon deals zero damage at any range.
- `falloff_end` is always > `falloff_start`. If equal, no falloff occurs (damage is constant).
- The result is linearly interpolated. No easing curves. Predictable and tunable.

**Splash Damage Falloff** (projectile weapons):

```
func calculate_splash(splash_max: float, splash_min: float,
                      splash_radius: float, distance_from_center: float) -> float:
    if distance_from_center >= splash_radius:
        return 0.0
    elif distance_from_center <= 0.0:
        return splash_max
    else:
        t = distance_from_center / splash_radius
        return splash_max - t * (splash_max - splash_min)
```

---

## 4. Example Weapon Roster

The following six weapons constitute the launch roster. All stats are initial tuning targets and will be iterated via the process described in `../06_cross_cutting/tuning_constants.md`.

### 4.1 Assault Rifle [WEP-001]

**Class:** Hitscan [W-CLASS-001]
**Slot:** Primary
**Role:** Versatile mid-range engagement. The baseline weapon against which all others are measured. Effective from 10m to 50m. Forgiving enough for new players, deep enough for skilled players via recoil control and headshot consistency.

| Parameter | Value |
|---|---|
| `damage_per_shot` | 18 |
| `fire_rate` | 600 RPM |
| `headshot_multiplier` | 2.5x |
| `falloff_start` | 30m |
| `falloff_end` | 60m |
| `min_damage` | 12 |
| `magazine_size` | 30 |
| `reload_time` | 2.0s |
| `spread_hip` | 2.0° |
| `spread_ads` | 0.5° |
| `recoil_vertical` | 1.2° |
| `recoil_horizontal` | 0.4° |
| `recoil_recovery` | 8.0°/s |

**Time-to-Kill Analysis (body shots, point blank):**
- 100 HP / 18 damage = 5.556 → 6 shots to kill
- At 600 RPM, 6 shots take 600ms
- **TTK (body, optimal range): 600ms**

**Time-to-Kill Analysis (headshots, point blank):**
- Headshot damage: 18 × 2.5 = 45
- 100 HP / 45 = 2.222 → 3 shots to kill
- At 600 RPM, 3 shots take 200ms
- **TTK (headshot, optimal range): 200ms**

---

### 4.2 Sniper Rifle [WEP-002]

**Class:** Hitscan [W-CLASS-001]
**Slot:** Primary
**Role:** Long-range precision elimination. Punishes stationary targets and predictable movement. Weak at close range due to low fire rate and severe hip-fire spread. The only weapon with perfect ADS accuracy.

| Parameter | Value |
|---|---|
| `damage_per_shot` | 55 |
| `fire_rate` | 40 RPM (bolt-action) |
| `headshot_multiplier` | 2.5x |
| `falloff_start` | 50m |
| `falloff_end` | 120m |
| `min_damage` | 40 |
| `magazine_size` | 5 |
| `reload_time` | 3.0s (individual rounds, 0.6s per round) |
| `spread_hip` | 5.0° |
| `spread_ads` | 0.0° |
| `recoil_vertical` | 3.5° |
| `recoil_horizontal` | 0.2° |
| `recoil_recovery` | 4.0°/s |

**Special — Optics:** The Sniper Rifle features a toggleable scope with two magnification levels:
- **2x zoom:** Wide-angle for mid-range awareness. Scope-in time: 0.25s.
- **4x zoom:** Narrow field of view for long-range targets. Scope-in time: 0.35s.
- Transition between zoom levels: 0.15s.

**Time-to-Kill Analysis (body, optimal range):**
- 100 HP / 55 = 1.818 → 2 shots to kill
- At 40 RPM, interval between shots is 1.5s
- **TTK (body, optimal range): 1500ms + 55ms = 1555ms (first shot + bolt cycle + second shot)**

**Time-to-Kill Analysis (headshot, any range within falloff):**
- Headshot at point blank: 55 × 2.5 = 137.5 → one-shot kill
- Headshot at max falloff: 40 × 2.5 = 100 → one-shot kill
- **TTK (headshot): 0ms (instant on hit)**

---

### 4.3 Rocket Launcher [WEP-003]

**Class:** Projectile [W-CLASS-002]
**Slot:** Primary
**Role:** Area denial and burst damage. Forces enemies to reposition. Effective against grouped targets. Weak against fast-moving targets at range due to projectile travel time and no gravity arc (cannot lob over cover). Direct hits are rewarding but difficult.

| Parameter | Value |
|---|---|
| `damage_direct` | 80 |
| `splash_damage_max` | 40 |
| `splash_damage_min` | 10 |
| `splash_radius` | 3.0m |
| `splash_falloff` | LINEAR |
| `projectile_speed` | 30 m/s |
| `projectile_gravity` | 0.0 m/s² |
| `fire_rate` | 60 RPM |
| `magazine_size` | 4 |
| `reload_time` | 2.5s |
| `projectile_lifetime` | 5.0s |
| `bounce_count` | 0 |

**Design Note — No Rocket Jumping:** Self-damage is enabled (player takes splash damage from their own rockets). However, the upward velocity imparted by self-splash does not exceed standard jump velocity. Rocket jumping as an evasion mechanic is explicitly disabled — this conflicts with Pillar 1, which states that movement options must be grounded and predictable. The rocket launcher is a weapon, not a movement tool.

**Time-to-Kill Analysis (direct hit):**
- 80 damage on direct hit. Does not kill a full-HP target (100 HP).
- Requires follow-up: either a second rocket (1.0s gap at 60 RPM) or a secondary weapon shot.
- **TTK (direct hit + direct hit): ~1060ms**

**Time-to-Kill Analysis (splash center):**
- 40 splash damage. Requires 3 splash hits to kill.
- **TTK (splash-only): ~2160ms** (impractical; this is not the intended use case)

---

### 4.4 Grenade Launcher [WEP-004]

**Class:** Projectile [W-CLASS-002]
**Slot:** Secondary
**Role:** Indirect fire and area control. Grenades arc over cover and bounce once before detonating. Ideal for flushing enemies from defensible positions. Requires prediction of enemy position 1-2 seconds in advance.

| Parameter | Value |
|---|---|
| `damage_direct` | 60 |
| `splash_damage_max` | 30 |
| `splash_damage_min` | 5 |
| `splash_radius` | 2.5m |
| `splash_falloff` | LINEAR |
| `projectile_speed` | 20 m/s |
| `projectile_gravity` | 9.81 m/s² |
| `fire_rate` | 90 RPM |
| `magazine_size` | 6 |
| `reload_time` | 2.2s |
| `projectile_lifetime` | 2.5s |
| `bounce_count` | 1 |

**Bounce Behavior:** On first contact with world geometry, the grenade bounces. Velocity after bounce is reduced to 40% of impact velocity. Angle of incidence equals angle of reflection (standard elastic bounce with damping). On second contact, or after 2.5s total lifetime, the grenade detonates. Grenades do not detonate on player contact — only on world geometry or timeout.

**Time-to-Kill Analysis (direct hit):**
- 60 damage on direct hit. Does not kill.
- **TTK (direct hit + direct hit): ~727ms** at 90 RPM interval

**Time-to-Kill Analysis (splash center + direct hit):**
- 30 splash + 60 direct = 90 total. Requires one more source of damage.
- This weapon is designed as a secondary — it excels when combined with primary weapon damage.

---

### 4.5 Energy Rifle [WEP-005]

**Class:** Beam [W-CLASS-003]
**Slot:** Primary
**Role:** Sustained mid-range damage. The only primary that can apply continuous pressure without reload breaks (until overheat). Rewards tracking skill — maintaining beam alignment on a moving target. Punished by overheat if the player is greedy.

| Parameter | Value |
|---|---|
| `damage_per_tick` | 8 |
| `tick_rate` | 100ms (10 ticks/second) |
| `range` | 40m |
| `ammo_per_tick` | 2 |
| `magazine_size` | 200 energy units |
| `overheat_threshold` | 3.0s (30 ticks) |
| `overheat_cooldown` | 1.5s |
| `ramp_type` | NONE |
| `ramp_max_multiplier` | 1.0 |
| `ramp_time` | 0.0s |
| `beam_width` | 0.05m (visual) |

**Ammo Efficiency:** At 2 units per tick and 10 ticks/second, the weapon consumes 20 units/second. With a 200-unit pool, the weapon can fire for 10 seconds before depleting its energy (assuming no overheat). Overheat triggers at 3.0s continuous, so the practical firing cycle is: 3.0s fire → 1.5s cooldown → 3.0s fire → ... Energy does not deplete during overheat cooldown.

**Time-to-Kill Analysis (continuous body tracking):**
- 8 damage per tick, 10 ticks/second = 80 DPS
- 100 HP / 80 DPS = 1.25s
- **TTK (body, continuous): 1250ms**

**Overheat Cycle DPS:** 3.0s of fire = 240 damage dealt. Then 1.5s of cooldown = 0 damage. Total cycle: 4.5s for 240 damage. Sustained DPS over a full cycle: **53.3 DPS**. This is the balancing factor — burst DPS is high, sustained DPS is moderate.

---

### 4.6 Laser Cutter [WEP-006]

**Class:** Beam [W-CLASS-003]
**Slot:** Secondary
**Role:** Close-range aggressive finisher. High per-tick damage but extremely short range. Designed as a follow-up weapon — soften with primary, close distance, finish with the Laser Cutter. The overheat penalty is severe to prevent spam.

| Parameter | Value |
|---|---|
| `damage_per_tick` | 12 |
| `tick_rate` | 100ms (10 ticks/second) |
| `range` | 15m |
| `ammo_per_tick` | 3 |
| `magazine_size` | 120 energy units |
| `overheat_threshold` | 2.0s (20 ticks) |
| `overheat_cooldown` | 2.0s |
| `ramp_type` | NONE |
| `ramp_max_multiplier` | 1.0 |
| `ramp_time` | 0.0s |
| `beam_width` | 0.12m (visual) |

**Ammo Efficiency:** At 3 units per tick and 10 ticks/second, the weapon consumes 30 units/second. With a 120-unit pool, the weapon can fire for 4 seconds total before depleting. Overheat triggers at 2.0s continuous. Practical firing cycle: 2.0s fire → 2.0s cooldown → 2.0s fire → depleted.

**Time-to-Kill Analysis (continuous body tracking):**
- 12 damage per tick, 10 ticks/second = 120 DPS
- 100 HP / 120 DPS = 0.833s
- **TTK (body, continuous, within 15m): 833ms**

**Range Constraint:** At 15m maximum range, the Laser Cutter demands close proximity. Combined with the movement system (see `movement_system.md`), closing to 15m requires deliberate approach and exposure to enemy fire. The high DPS is balanced by the risk required to reach effective range.

---

## 5. Weapon Equipping Rules

### 5.1 Loadout Structure

Each player selects a loadout consisting of **exactly 1 primary weapon and 1 secondary weapon** before spawning. This selection persists until the player's next death.

| Slot | Available Weapons | Slot ID |
|---|---|---|
| Primary | Assault Rifle [WEP-001], Sniper Rifle [WEP-002], Rocket Launcher [WEP-003], Energy Rifle [WEP-005] | `SLOT_PRIMARY` |
| Secondary | Pistol [WEP-DEFAULT], Grenade Launcher [WEP-004], Laser Cutter [WEP-006] | `SLOT_SECONDARY` |

### 5.2 Default Pistol [WEP-DEFAULT]

The default secondary for players who do not select an alternative. Intentionally weaker than all other weapons — it is a fallback, not a competitive choice.

| Parameter | Value |
|---|---|
| `damage_per_shot` | 14 |
| `fire_rate` | 360 RPM (semi-automatic) |
| `headshot_multiplier` | 2.0x |
| `falloff_start` | 20m |
| `falloff_end` | 40m |
| `min_damage` | 8 |
| `magazine_size` | 12 |
| `reload_time` | 1.5s |
| `spread_hip` | 3.0° |
| `spread_ads` | 1.0° |
| `recoil_vertical` | 1.5° |
| `recoil_horizontal` | 0.6° |
| `recoil_recovery` | 10.0°/s |

### 5.3 Weapon Swap Mechanics

- **Swap input:** Dedicated key/button to toggle between primary and secondary.
- **Swap time:** 0.5s from input to the new weapon being fire-ready. During swap, the player cannot fire either weapon.
- **Swap animation:** Weapon lowers out of view, then the new weapon raises into view. No complex animations — clarity over flair.
- **Auto-swap:** When a weapon's magazine is empty and the player attempts to fire, the weapon auto-reloads (does NOT auto-swap). The player must manually swap if they prefer to switch to their other weapon rather than reload.

### 5.4 Loadout Changes

- **When:** Loadout can be changed during the loadout selection screen, which appears on death before respawn.
- **Persistence:** The selected loadout persists across deaths until changed. Players are not required to reselect each death.
- **Mid-life changes:** Not permitted. Once spawned, the loadout is locked until death.

---

## 6. Weapon Feedback Standards

Every weapon must implement the full feedback suite below. These are hard requirements — a weapon does not ship without all elements present and signed off by the design lead.

### 6.1 Visual Feedback

| Element | Specification | Notes |
|---|---|---|
| Muzzle flash | Bright flash at barrel exit, 1-2 frames duration, directional cone aligned to fire vector | Must not obscure crosshair or target. Max screen coverage: 2% of viewport. |
| Tracer / beam | Hitscan: thin line from muzzle to impact, visible for 80ms, slight spread alignment. Projectile: attached to projectile entity. Beam: persistent line from muzzle to impact while firing. | Tracers do not perfectly indicate hit location — they show aim direction only. |
| Impact effect (world) | Decal or particle burst at hit point. Material-reactive (sparks on metal, dust on concrete, etc.). | Max 8 particles per impact. Must clean up within 500ms. |
| Impact effect (player) | Brief red flash on hit player's model. No blood. Directional indicator visible to the hit player. | Max visual noise: subtle. The hit player's HUD shows a directional damage indicator (see `player_experience.md`). |
| Damage numbers | Not displayed in competitive modes. Shown only in practice/training modes. | Prevents information overload. |

### 6.2 Audio Feedback

| Element | Specification | Notes |
|---|---|---|
| Fire sound | Distinct per weapon. Must be identifiable by sound alone at up to 30m. | 3D spatialized. Max concurrent instances: 4 (to prevent audio stacking at high RPM). |
| Reload sound | Mechanical click-clack. Two-phase: magazine out (0.5s into reload) and magazine in (0.8s before reload completes). | Audible to enemies within 10m. |
| Empty click | Short, dry click when firing with an empty magazine. | Must be distinct from reload sound. |
| Impact sound (enemy) | Satisfying "thud" or "crack" depending on body vs headshot. Headshot sound has a distinct high-frequency component. | Only audible to the shooter. |
| Impact sound (world) | Material-reactive. Ricochet sound for hitscan on metal at oblique angles. | 3D spatialized. |

### 6.3 Screen Effects

| Element | Specification | Rationale |
|---|---|---|
| Screen shake | **Disabled.** No weapon causes screen shake on the shooter. | Pillar 4: visual clarity. Screen shake obscures information and feels punitive. |
| FOV kick | Sniper Rifle [WEP-002]: 2° FOV decrease on fire, recovered over 0.3s. All other weapons: 0°. | Minimal, subtle. Conveys power without disorienting. |
| Recoil visualization | Camera kicks upward per `recoil_vertical` and drifts per `recoil_horizontal`. Recovery is smooth, interpolated over time by `recoil_recovery`. | The player sees and controls recoil through aim. This is the primary "feel" driver. |
| Hit confirmation | Crosshair flashes briefly (white for body, yellow for headshot) on confirmed hit. 100ms duration. | Subtle but informative. Not noisy. |

---

## 7. Balance Tuning Methodology

### 7.1 Target Metrics

Every weapon is evaluated against these baseline targets:

| Metric | Target | Tolerance |
|---|---|---|
| TTK (body, optimal range) | 600ms – 1000ms | ±150ms |
| TTK (headshot, optimal range) | 0ms – 400ms | ±100ms |
| Pick rate (primary) | 15% – 35% per weapon | Below 10% or above 50% triggers review |
| Win rate delta | < 3% between highest and lowest pick-rate weapon | Measured over 10,000+ matches |

### 7.2 Tuning Levers (Priority Order)

When adjusting a weapon, modify parameters in this order. Higher-priority levers have less collateral impact on game feel:

1. **Damage per shot / per tick** — Most direct. Changes TTK without altering feel.
2. **Falloff distances** — Adjusts effective range without changing close-range balance.
3. **Magazine size** — Affects sustained fights without changing per-engagement power.
4. **Reload time** — Changes downtime between fights.
5. **Spread** — Alters consistency. High impact on feel. Use sparingly.
6. **Recoil** — Changes the skill floor/ceiling. Very high impact on feel. Last resort.

### 7.3 Data Pipeline

All weapon balance changes must follow the tuning pipeline described in `../06_cross_cutting/tuning_constants.md`. No hardcoded values in weapon definitions — all parameters are loaded from tunable constants files that can be hot-reloaded in development builds.

---

## 8. Implementation Notes

### 8.1 Component Architecture

Each weapon is implemented as a self-contained entity with the following components:

- **WeaponFireComponent:** Handles fire input, rate limiting, and shot initiation.
- **WeaponAmmoComponent:** Tracks magazine state, reload logic, and ammo consumption.
- **WeaponFeedbackComponent:** Manages visual and audio playback.
- **WeaponSpreadComponent:** Calculates spread offset per shot based on stance and movement.
- **WeaponRecoilComponent:** Applies and recovers recoil offset.

For hitscan weapons, add:
- **HitscanComponent:** Performs raycast and applies damage.

For projectile weapons, add:
- **ProjectileSpawnComponent:** Spawns projectile entity on fire.

For beam weapons, add:
- **BeamComponent:** Manages continuous raycast, tick timing, and overheat state.

### 8.2 Input Processing

Fire input is processed as follows (per frame):

```
1. Poll fire input state
2. If fire requested AND weapon is fire-ready:
   a. Check ammo > 0
   b. Check not in reload state
   c. Check not in weapon swap state
   d. Check not in overheat cooldown (beam only)
   e. Check fire rate timer has elapsed since last shot
3. If all checks pass:
   a. Consume ammo (1 round for hitscan/projectile, per-tick for beam)
   b. Reset fire rate timer
   c. Execute weapon-type-specific fire logic
   d. Trigger feedback component
4. If ammo == 0 after consumption:
   a. Play empty click (if fire input is still held next frame)
   b. Auto-reload begins after 0.3s delay
```

### 8.3 ADS (Aim Down Sights) Mechanics

- **Transition time:** 0.2s from hip to ADS, 0.2s from ADS to hip.
- **Movement penalty:** Movement speed reduced by 30% while ADS (stacks with movement system — see `movement_system.md`).
- **Spread reduction:** Spread transitions from `spread_hip` to `spread_ads` over the ADS transition time. Intermediate values are linearly interpolated.
- **FOV change:** 5° FOV decrease while ADS (all weapons that support ADS). Sniper Rifle overrides this with its scope FOV.

---

## 9. Future Expansion Guidelines

New weapons must conform to the following requirements before being added to the roster:

1. Must use one of the three existing weapon types [W-CLASS-001], [W-CLASS-002], or [W-CLASS-003]. New types require a separate design document and engineering review.
2. Must define all parameters for its weapon type. No partial definitions.
3. Must not duplicate the role of an existing weapon without a clear mechanical distinction.
4. Must be assigned to either the Primary or Secondary slot. No new slots without design lead approval.
5. Must include the full feedback suite defined in §6.
6. Must undergo balance tuning per §7 before being enabled in competitive play.

---

## 10. See Also

| Document | Relationship |
|---|---|
| `health_and_damage.md` [SYS-003] | Damage application, collider groups, health pool definitions |
| `addon_system.md` [SYS-004] | Weapon modifications that alter parameters defined here |
| `../06_cross_cutting/tuning_constants.md` | Hot-reloadable parameter files and tuning pipeline |
| `movement_system.md` [SYS-001] | Movement speed during ADS, weapon swap movement penalties |
| `player_experience.md` [SYS-005] | HUD elements, damage indicators, crosshair behavior |

---

*End of document. Weapon System [SYS-002] v1.0.0*
