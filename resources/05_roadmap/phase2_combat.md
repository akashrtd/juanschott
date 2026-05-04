# Phase 2 — Combat

**Version Target:** v0.2  
**Estimated Duration:** 6–8 weeks  
**Status:** Planning  
**Dependencies:** Phase 1 (v0.1) complete  

---

## 1. Overview

Phase 2 transforms the movement prototype into a **combat prototype.** Players can now shoot, deal damage, take damage, die, and respawn. This phase introduces the full weapon system (hitscan, projectile, and beam weapon types), the health and damage pipeline, death and respawn mechanics, a basic add-on modification system, and the advanced movement technique of wall-run/wall-jump.

By the end of Phase 2, two players on the same machine should be able to engage in a complete combat loop: spawn, select weapons, fight, kill, respawn, and fight again. The time-to-kill (TTK) should be low — consistent with the game's arena shooter identity — and health regeneration should keep players in the fight between engagements.

This phase is the largest single phase in the roadmap because combat is the game's core loop. Every weapon, every damage calculation, every respawn mechanic must feel right before multiplayer networking is layered on top in Phase 3.

---

## 2. Goals

- Implement three weapon archetypes: hitscan, projectile, and beam
- Build a health system with regeneration
- Create a damage pipeline that handles all weapon-to-player interactions
- Implement death detection and respawn mechanics
- Add a weapon view model (first-person gun model visible to the player)
- Implement a basic add-on system for weapon modifications
- Add wall-run and wall-jump as advanced movement techniques
- Validate low TTK through playtesting

---

## 3. Task Breakdown

### 3.1 Health System

**Task 2.1: Health component and resource**

Create a `Health` component for the player entity:
- `current: f32` — current health points
- `max: f32` — maximum health (default: 100)
- `regen_delay: f32` — seconds since last damage before regen begins
- `regen_rate: f32` — health points per second during regen
- `time_since_damage: f32` — tracks the regen delay timer

**Task 2.2: Health regeneration system**

Each frame, if `time_since_damage > regen_delay`:
- Increase `current` by `regen_rate * delta_time`
- Clamp to `max`
- The regen model is: take damage → wait briefly → rapidly regenerate to full. This keeps players aggressive and reduces downtime. The exact delay and rate will be tuned, but the target is full regen within 3–5 seconds of the last hit.

**Task 2.3: Damage application function**

Create a centralized `apply_damage` function:
- Accepts: target entity, damage amount, source entity (optional), damage type
- Reduces target health by damage amount
- Resets `time_since_damage` to 0 (interrupting regen)
- Fires a `DamageEvent` for other systems to react to (hit markers, sounds, etc.)
- Checks if health reaches 0 and fires a `DeathEvent` if so

**Estimated effort:** 4–5 hours

---

### 3.2 Damage System

**Task 2.4: Damage event pipeline**

Establish an event-driven damage pipeline:
- `DamageEvent` carries: target, source, amount, damage type, hit location (optional)
- `DeathEvent` carries: killed entity, killer entity, weapon used, death location
- `RespawnEvent` carries: respawning entity, spawn point

All damage flows through `apply_damage`. No system should directly modify health. This centralization is critical for Phase 3, where damage must be validated server-side.

**Task 2.5: Hit feedback**

Implement basic hit feedback:
- Screen flash or crosshair indicator when dealing damage
- Damage direction indicator when taking damage (optional for v0.2)
- Hit marker sound (placeholder audio is acceptable)
- Kill notification in console/log (no UI yet — HUD comes in Phase 5)

**Estimated effort:** 3–4 hours

---

### 3.3 Hitscan Weapons

**Task 2.6: Hitscan weapon definition**

Define hitscan weapon data:
- `damage: f32`
- `range: f32` — maximum distance the ray travels
- `fire_rate: f32` — shots per second
- `spread: f32` — maximum random deviation angle
- `headshot_multiplier: f32` — damage multiplier for headshots (if hitbox supports it)

**Task 2.7: Hitscan raycast**

On fire:
- Cast a ray from the camera position in the camera's forward direction
- Apply spread as random angular offset
- Check for intersection with entities that have `Health` components
- If hit: call `apply_damage` on the hit entity
- If hit a wall: spawn a bullet impact effect (placeholder)

**Task 2.8: Fire rate limiting**

Track time since last shot per weapon. Only allow firing when `time_since_last_shot >= 1.0 / fire_rate`. Support automatic weapons (hold to fire) and semi-automatic weapons (click to fire).

**Estimated effort:** 5–7 hours

---

### 3.4 Projectile Weapons

**Task 2.9: Projectile entity definition**

Define a projectile entity:
- Speed, damage, lifetime (seconds before despawn), radius (for collision)
- Visual: a small mesh (sphere or custom shape) with a trail effect (stretch the mesh in the direction of travel)
- Physics: a sensor collider (no physical force, just detection) or manual ray-based collision per frame

**Task 2.10: Projectile spawning and movement**

On fire:
- Spawn a projectile entity at the weapon's muzzle position
- Set velocity in the camera's forward direction (plus any spread)
- Each frame, move the projectile by `velocity * delta_time`
- Check for collision with health entities and geometry
- On hit entity: `apply_damage`, despawn projectile
- On hit geometry: spawn impact effect, despawn projectile
- On lifetime expired: despawn without effect

**Task 2.11: Projectile types**

Implement at least two projectile archetypes:
- **Fast projectile:** High speed, low drop, low radius. For rocket-launcher-style weapons.
- **Slow projectile:** Lower speed, larger radius, possible area-of-effect on impact. For grenade-like weapons.

**Estimated effort:** 5–7 hours

---

### 3.5 Beam Weapons

**Task 2.12: Beam weapon definition**

Define beam weapon data:
- `damage_per_second: f32` — continuous damage while beam is on target
- `range: f32`
- `width: f32` — beam visual thickness
- `ramp_up_time: f32` — seconds to reach full damage (optional mechanic)

**Task 2.13: Beam rendering and damage**

While firing:
- Cast a ray from the camera each frame
- If hitting an entity with health: apply `damage_per_second * delta_time`
- Render the beam as a line or stretched mesh from muzzle to hit point
- The beam should visually connect instantly (no travel time — it is an energy weapon)
- Add a glow or particle effect at the impact point

**Estimated effort:** 4–6 hours

---

### 3.6 Weapon View Model

**Task 2.14: First-person weapon model**

Spawn a separate entity hierarchy for the weapon view model:
- A child of the camera (moves with the player's view)
- Rendered on a separate camera layer so it does not clip into walls or other players
- Positioned in the lower-right of the screen (standard FPS positioning)
- Placeholder models are acceptable — simple geometric shapes representing weapon types

**Task 2.15: Weapon animations**

Implement basic weapon animations:
- Idle: subtle sway or breathing motion
- Fire: recoil kickback (translate and rotate backward, then return)
- Switch: lower current weapon, raise new weapon (when changing weapons)

These can be procedural (code-driven transforms) rather than skeletal animations for v0.2.

**Estimated effort:** 5–7 hours

---

### 3.7 Weapon Switching and Inventory

**Task 2.16: Weapon inventory system**

Create a `WeaponInventory` component:
- Slots for carried weapons (e.g., 2–3 weapons)
- Current active weapon index
- Methods to switch, add, and remove weapons

**Task 2.17: Weapon switch input**

Bind number keys (1, 2, 3) or scroll wheel to weapon switching. Implement:
- Switch delay (cannot fire during the switch animation)
- Cancel switch if the player switches again before the first switch completes

**Estimated effort:** 3–4 hours

---

### 3.8 Death and Respawn

**Task 2.18: Death handling**

When a `DeathEvent` fires:
- Disable the dead player's input and movement
- Hide or ragdoll the player entity (placeholder: just make invisible)
- Log the kill
- Start a respawn timer (e.g., 3 seconds)

**Task 2.19: Respawn system**

When the respawn timer expires:
- Move the player to a random spawn point
- Reset health to max
- Re-enable input and movement
- Make the player visible again
- Grant brief invulnerability (e.g., 1 second) to prevent spawn camping during local testing

**Task 2.20: Spawn point system**

Define spawn points in the test map:
- Several locations around the arena, spread out to prevent immediate re-engagement
- Marked as entities or configured as positions in a resource
- Respawn picks a random point, ideally far from the killer's current position

**Estimated effort:** 4–5 hours

---

### 3.9 Add-On System (Basic)

**Task 2.21: Add-on definition framework**

Create the data structures for weapon add-ons:
- `AddOn` struct: name, type (scope, barrel, magazine, grip, etc.), stat modifications
- Stat modifications are key-value pairs that modify weapon properties (e.g., `damage: +10%`, `fire_rate: +15%`, `spread: -20%`)
- Each weapon has a number of add-on slots

**Task 2.22: Add-on application**

When an add-on is equipped:
- Iterate the add-on's stat modifications
- Apply multipliers to the weapon's base stats
- Store the modified stats for use in gameplay
- Support removing add-ons and reverting stats

**Task 2.23: Basic add-on implementation**

Implement 3–4 simple add-ons to validate the system:
- **Extended Magazine:** Increases ammo capacity (if ammo tracking exists) or reduces reload time
- **Quickdraw:** Reduces weapon switch time
- **Laser Sight:** Reduces spread
- **High-Caliber:** Increases damage but increases recoil or spread

For v0.2, add-ons are equipped via developer console or hardcoded. The loadout selection UI comes in Phase 3.

**Estimated effort:** 5–7 hours

---

### 3.10 Wall-Run / Wall-Jump

**Task 2.24: Wall detection**

Implement wall detection for the player:
- Cast rays from the player's sides (left, right, and optionally forward-diagonal)
- If a ray hits a wall within close range while the player is airborne and moving alongside the wall: enter wall-run state
- Walls must be roughly vertical (within a threshold angle) to be wall-runnable

**Task 2.25: Wall-run movement**

During wall-run:
- Maintain the player's forward velocity along the wall surface
- Counteract gravity partially or fully (the player "sticks" to the wall while moving)
- Apply a maximum wall-run duration (e.g., 1.5 seconds) after which the player falls
- The camera tilts slightly toward the wall for visual feedback

**Task 2.26: Wall-jump**

While in wall-run state:
- Pressing jump launches the player away from the wall with an upward component
- The wall-jump velocity should be distinct from a normal jump — more horizontal push-off
- After wall-jump, the player can double-jump (if double jump is still available)
- Chain wall-runs between adjacent walls should be possible for skilled play

**Task 2.27: Wall-run exit conditions**

The player exits wall-run when:
- Wall-run duration expires
- The player moves away from the wall (input away from wall surface)
- The player reaches the top of the wall (no more wall surface ahead)
- The player fires a weapon (optional design decision — some games allow shooting during wall-run)

**Estimated effort:** 8–10 hours

---

## 4. Task Summary

| Task Group | Tasks | Estimated Hours |
|-----------|-------|----------------|
| Health System | 2.1, 2.2, 2.3 | 4–5 |
| Damage System | 2.4, 2.5 | 3–4 |
| Hitscan Weapons | 2.6, 2.7, 2.8 | 5–7 |
| Projectile Weapons | 2.9, 2.10, 2.11 | 5–7 |
| Beam Weapons | 2.12, 2.13 | 4–6 |
| Weapon View Model | 2.14, 2.15 | 5–7 |
| Weapon Inventory | 2.16, 2.17 | 3–4 |
| Death/Respawn | 2.18, 2.19, 2.20 | 4–5 |
| Add-On System | 2.21, 2.22, 2.23 | 5–7 |
| Wall-Run/Wall-Jump | 2.24, 2.25, 2.26, 2.27 | 8–10 |
| **Total** | **27 tasks** | **46–62 hours** |

At 8–12 productive hours per week, this maps to the 6–8 week estimate.

---

## 5. Acceptance Criteria

Phase 2 is complete when **all** of the following are true:

1. **Hitscan Weapons:** Player can fire hitscan weapons. Raycasts hit entities and geometry. Damage is applied correctly. Fire rate limiting works. Spread is visible at range.
2. **Projectile Weapons:** Player can fire projectile weapons. Projectiles travel through space, collide with entities and geometry, and deal damage. Lifetime despawn works.
3. **Beam Weapons:** Player can fire beam weapons. Beams render visually and deal continuous damage per second. Range limiting works.
4. **Health System:** Players have health that depletes on damage and regenerates after a delay. Health clamps at 0 and max. Regen is interrupted by incoming damage.
5. **Death/Respawn:** Players die at 0 health. Dead players respawn after a delay at a spawn point with full health and brief invulnerability.
6. **Weapon View Model:** First-person weapon model is visible. Basic fire and switch animations work. The model does not clip into world geometry.
7. **Add-On System:** At least three add-ons can be applied to weapons. Stat modifications are reflected in gameplay (e.g., reduced spread is noticeable, increased damage is reflected in TTK).
8. **Wall-Run/Wall-Jump:** Player can wall-run along vertical surfaces while airborne. Wall-jump launches the player away from the wall. Chain wall-runs are possible. Duration limit prevents indefinite wall-running.
9. **Low TTK:** Average time-to-kill in a 1v1 engagement is under 2 seconds with optimal accuracy. This will be validated through playtesting and adjusted via weapon balance constants.
10. **Performance:** All combat features run at 120fps on mid-range hardware with two players active. Projectile rendering and beam rendering do not cause frame drops.

---

## 6. Playtest Protocol

### 6.1 Combat Loop Test

1. Spawn two players on the test map.
2. Each player selects different weapon types.
3. Engage in combat. Verify damage is dealt in both directions.
4. Kill the opponent. Verify death animation/state and respawn timer.
5. Verify the opponent respawns at a different location with full health.
6. Repeat for 10 minutes of continuous combat.

### 6.2 Weapon Variety Test

1. Test each weapon type against a stationary target (second player AFK).
2. Record time-to-kill for each weapon at optimal accuracy.
3. Verify hitscan is instant, projectile has travel time, beam deals damage per second.
4. Verify weapon switching works and switch delay is enforced.

### 6.3 Wall-Run Integration Test

1. Add wall-run-compatible surfaces to the test map (tall flat walls with approach angles).
2. Sprint toward a wall, jump, and wall-run along it.
3. Wall-jump off and verify the trajectory.
4. Chain between two adjacent walls.
5. Verify wall-run ends when the player stops moving, the timer expires, or the wall ends.

---

## 7. Design Notes

### Time-to-Kill Target

The game targets a **low TTK** consistent with arena shooters. Design target is 1–2 seconds of focused fire to kill. This means:
- Individual weapon damage per hit is high
- Health regen delay is short (so players heal between fights, not during)
- Headshot multipliers reward accuracy
- Add-ons that increase damage are powerful but come with tradeoffs

### Add-On Philosophy

The add-on system is a core differentiator for JuanSchott. Each weapon has slots for modifications that alter its behavior. The system should be:
- **Composable:** Any add-on can go in any compatible slot. Effects stack predictably.
- **Tunable:** All modifications are multiplier-based on base stats. A spreadsheet can balance all combinations.
- **Visible:** The weapon view model should reflect equipped add-ons visually (even if simplified in v0.2).

### Wall-Run Design

Wall-run is intended as an **escape and repositioning tool**, not a primary combat tool. Players should not be able to fight effectively while wall-running (optional: disable accurate aiming during wall-run). The movement should feel fluid and expressive — a skilled player should be able to chain wall-runs, wall-jumps, and double jumps to navigate the arena in ways that surprise opponents.

---

## 8. Risks and Mitigations

| Risk | Mitigation |
|------|-----------|
| Wall-run physics are complex and bug-prone | Implement wall-run last within the phase. Core combat must be stable without it. Scope wall-run to a simple version and iterate. |
| Weapon balance requires extensive tuning | All weapon stats live in `constants.rs`. Create a balance spreadsheet. Tune continuously during the phase. |
| Two-player local testing is clunky | Support keyboard split (WASD + IJKL for two players) or implement a simple AI target dummy for solo testing. |
| View model rendering has layer issues | Test camera layers early. Ensure the weapon renders on top of the world without z-fighting. |

---

## 9. Exit Criteria

Upon completing Phase 2 and meeting all acceptance criteria:

1. Tag the repository as `v0.2`.
2. Conduct a playtest session with at least 30 minutes of combat.
3. Document weapon balance observations and TTK data.
4. Review the add-on system architecture for extensibility before Phase 3.
5. Proceed to Phase 3 (Multiplayer).
