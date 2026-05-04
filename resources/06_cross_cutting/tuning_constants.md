# Tuning Constants

## Document Metadata

| Field | Value |
|---|---|
| Document ID | CCD-004 |
| Version | 1.0 |
| Status | Draft |
| Owner | Design Lead |
| Last Updated | 2026-05-05 |

---

## 1. Purpose

This document is the **master reference** for every tunable value in JuanSchott. All constants from all game design documents are consolidated here. Developers and designers use this document as the single source of truth for gameplay numbers.

Constants are organized by system. Each constant includes: the variable name (as it appears in code), the value, the unit, and notes explaining the design intent or relevant context.

**Rule**: If a number affects gameplay, it must be documented here. If it is not documented here, it does not exist as a design constant. Undocumented magic numbers in code are bugs.

---

## 2. How to Read This Document

| Column | Description |
|---|---|
| Name | The constant's variable name. Matches the codebase. |
| Value | The current tuned value. |
| Unit | The unit of measurement (meters, seconds, etc.). |
| Notes | Design intent, constraints, or dependencies. |

All distance values are in **meters** (Bevy world units = meters). All time values are in **seconds**. All speed values are in **meters per second** (m/s). All acceleration values are in **meters per second squared** (m/s^2). All angular values are in **degrees**. All damage values are in **HP** (Health Points).

---

## 3. Movement Constants

Source: `movement_system.md`

### 3.1 Ground Movement

| Name | Value | Unit | Notes |
|---|---|---|---|
| WALK_SPEED | 5.0 | m/s | Base movement speed. No stamina cost. |
| SPRINT_SPEED | 8.0 | m/s | 60% faster than walk. Louder footsteps (20m audible range vs 8m). |
| GROUND_ACCEL | 40.0 | m/s^2 | How quickly the player reaches max speed on the ground. High value for snappy feel. |
| GROUND_DECEL | 40.0 | m/s^2 | How quickly the player stops on the ground. Matches accel for symmetric feel. |
| AIR_CONTROL | 0.3 | ratio | Fraction of ground accel applied while airborne. 0.3 = 30% of GROUND_ACCEL. Limits mid-air direction changes. |

### 3.3 Slide

| Name | Value | Unit | Notes |
|---|---|---|---|
| SLIDE_INITIAL_SPEED | 12.0 | m/s | Speed at the start of a slide. 50% faster than sprint. |
| SLIDE_DECEL | 6.0 | m/s^2 | Deceleration during slide. Player slows from 12 m/s to sprint speed over the slide duration. |
| SLIDE_MAX_DURATION | 1.2 | s | Maximum slide time. Slide ends early if speed drops below sprint speed. |

### 3.4 Jump and Gravity

| Name | Value | Unit | Notes |
|---|---|---|---|
| JUMP_VELOCITY | 7.0 | m/s | Initial upward velocity of a ground jump. Produces a jump height of approximately 2.5m. |
| DOUBLE_JUMP_VELOCITY | 6.0 | m/s | Upward velocity of the double jump. Slightly less than ground jump. |
| GRAVITY | 20.0 | m/s^2 | Downward acceleration applied to airborne players. Higher than Earth gravity (9.8) for snappier falls. |
| JUMP_BUFFER | 0.15 | s | Input window before landing where a jump command is stored and executed on landing. |

### 3.5 Wall Movement

| Name | Value | Unit | Notes |
|---|---|---|---|
| WALL_RUN_SPEED | 7.0 | m/s | Lateral speed while wall-running. Slightly less than sprint speed. |
| WALL_RUN_DURATION | 1.5 | s | Maximum wall-run time. Wall-run ends when timer expires. |
| WALL_RUN_GRAVITY | 4.0 | m/s^2 | Reduced gravity during wall-run. 20% of normal gravity. Allows sustained lateral movement. |
| WALL_JUMP_LATERAL | 7.0 | m/s | Lateral force of a wall jump. Propels player away from the wall. |
| WALL_JUMP_VERTICAL | 5.0 | m/s | Upward force of a wall jump. Less than a ground jump. |

### 3.6 Player Dimensions

| Name | Value | Unit | Notes |
|---|---|---|---|
| PLAYER_HEIGHT | 1.8 | m | Standing height. Used for capsule collider height. |
| CROUCH_HEIGHT | 1.2 | m | Crouching height. 66% of standing height. Used during slide and crouch. |
| EYE_HEIGHT | 1.6 | m | Camera height offset from player's feet when standing. ~89% of PLAYER_HEIGHT. |
| PLAYER_RADIUS | 0.3 | m | Capsule collider radius. Determines collision width. |

---

## 4. Health Constants

Source: `health_and_damage.md`

| Name | Value | Unit | Notes |
|---|---|---|---|
| BASE_HP | 100 | HP | Starting and maximum HP for all players. |
| REGEN_DELAY | 3.0 | s | Time after last damage before health regeneration begins. |
| REGEN_RATE | 50.0 | HP/s | HP recovered per second during regeneration. Full regen (100→100) in 2 seconds from any non-zero HP. |
| HEADSHOT_MULTIPLIER | 2.5 | x | Damage multiplier for hits to the top 20% of the player capsule. A 40-damage body shot becomes 100 damage on headshot — a one-shot kill for many weapons. |
| SPAWN_PROTECT_DURATION | 1.5 | s | Invulnerability duration after respawning. Player can move but cannot deal damage during this window. |

---

## 5. Weapon Constants

Source: `weapon_system.md`

### 5.1 Pistol

| Name | Value | Unit | Notes |
|---|---|---|---|
| PISTOL_DAMAGE | 25 | HP | Per shot. 4 body shots to kill. 2 headshots to kill (62.5 each). |
| PISTOL_FIRE_RATE | 0.2 | s | Time between shots. 5 shots per second. |
| PISTOL_FALLOFF_START | 20 | m | Full damage up to this range. |
| PISTOL_FALLOFF_END | 50 | m | Minimum damage beyond this range. |
| PISTOL_MIN_DAMAGE | 12 | HP | Minimum damage at maximum range. |
| PISTOL_MAGAZINE | 12 | rounds | Ammo per magazine. |
| PISTOL_RELOAD_TIME | 1.2 | s | Time to reload. |
| PISTOL_SPREAD_HIP | 3.0 | deg | Spread angle when hip-firing. |
| PISTOL_SPREAD_ADS | 0.5 | deg | Spread angle when aiming down sights. |
| PISTOL_RECOIL | 2.0 | deg | Recoil per shot. |

### 5.2 Assault Rifle

| Name | Value | Unit | Notes |
|---|---|---|---|
| RIFLE_DAMAGE | 18 | HP | Per shot. ~6 body shots to kill. 3 headshots to kill. |
| RIFLE_FIRE_RATE | 0.1 | s | Time between shots. 10 shots per second. |
| RIFLE_FALLOFF_START | 15 | m | Full damage up to this range. |
| RIFLE_FALLOFF_END | 45 | m | Minimum damage beyond this range. |
| RIFLE_MIN_DAMAGE | 9 | HP | Minimum damage at maximum range. |
| RIFLE_MAGAZINE | 30 | rounds | Ammo per magazine. |
| RIFLE_RELOAD_TIME | 1.8 | s | Time to reload. |
| RIFLE_SPREAD_HIP | 4.0 | deg | Spread angle when hip-firing. |
| RIFLE_SPREAD_ADS | 1.0 | deg | Spread angle when aiming down sights. |
| RIFLE_RECOIL | 1.5 | deg | Recoil per shot. Cumulative during sustained fire. |

### 5.3 Shotgun

| Name | Value | Unit | Notes |
|---|---|---|---|
| SHOTGUN_DAMAGE | 8 | HP | Per pellet. 12 pellets per shot. Max 96 damage at point-blank. |
| SHOTGUN_PELLETS | 12 | count | Number of pellets per shot. |
| SHOTGUN_FIRE_RATE | 0.8 | s | Time between shots. |
| SHOTGUN_FALLOFF_START | 5 | m | Full damage up to this range. |
| SHOTGUN_FALLOFF_END | 15 | m | Minimum damage beyond this range. |
| SHOTGUN_MIN_DAMAGE | 3 | HP | Minimum damage per pellet at max range. |
| SHOTGUN_MAGAZINE | 6 | rounds | Ammo per magazine. |
| SHOTGUN_RELOAD_TIME | 2.2 | s | Time to reload. |
| SHOTGUN_SPREAD_HIP | 8.0 | deg | Spread angle when hip-firing. Wide cone. |
| SHOTGUN_SPREAD_ADS | 5.0 | deg | Spread angle when aiming down sights. Still wide — tighter grouping. |
| SHOTGUN_RECOIL | 5.0 | deg | Heavy recoil per shot. |

### 5.4 Beam Weapon

| Name | Value | Unit | Notes |
|---|---|---|---|
| BEAM_DAMAGE_PER_TICK | 8 | HP | Damage per tick. |
| BEAM_TICK_RATE | 0.1 | s | Time between damage ticks. 10 ticks per second. Effective DPS: 80. |
| BEAM_FALLOFF_START | 10 | m | Full damage up to this range. |
| BEAM_FALLOFF_END | 35 | m | Minimum damage beyond this range. |
| BEAM_MIN_DAMAGE_PER_TICK | 3 | HP | Minimum damage per tick at max range. |
| BEAM_AMMO | 100 | units | Ammo units. Each tick consumes 1 unit. 10 seconds of sustained fire. |
| BEAM_RELOAD_TIME | 2.5 | s | Time to reload. |
| BEAM_SPREAD_HIP | 1.5 | deg | Minimal spread — beam is nearly precise. |
| BEAM_SPREAD_ADS | 0.3 | deg | Near-perfect accuracy when ADS. |
| BEAM_RECOIL | 0.5 | deg | Very low recoil. Beam weapons are about tracking, not recoil control. |

### 5.5 Projectile Launcher

| Name | Value | Unit | Notes |
|---|---|---|---|
| PROJECTILE_DAMAGE | 80 | HP | Direct hit damage. High — projectile is hard to land. |
| PROJECTILE_SPLASH_DAMAGE | 50 | HP | Splash damage at impact point. |
| PROJECTILE_SPLASH_RADIUS | 4.0 | m | Radius of splash damage. Linear falloff from center to edge. |
| PROJECTILE_SPEED | 30.0 | m/s | Projectile travel speed. Fast enough to be threatening, slow enough to dodge. |
| PROJECTILE_FIRE_RATE | 1.2 | s | Time between shots. |
| PROJECTILE_MAGAZINE | 4 | rounds | Ammo per magazine. |
| PROJECTILE_RELOAD_TIME | 2.5 | s | Time to reload. |
| PROJECTILE_SPREAD_HIP | 1.0 | deg | Low spread — projectile goes where aimed. |
| PROJECTILE_SPREAD_ADS | 0.5 | deg | Near-perfect accuracy when ADS. |
| PROJECTILE_RECOIL | 4.0 | deg | Moderate recoil. |
| PROJECTILE_MIN_DAMAGE | 40 | HP | Minimum direct hit damage (no falloff for projectiles — this is splash minimum at edge). |

---

## 6. Add-On Constants

Source: `addon_system.md`

### 6.1 Grapple Hook

| Name | Value | Unit | Notes |
|---|---|---|---|
| GRAPPLE_COOLDOWN | 8.0 | s | Time between uses. |
| GRAPPLE_MAX_RANGE | 25.0 | m | Maximum grapple cable length. |
| GRAPPLE_PULL_SPEED | 18.0 | m/s | Speed of pull toward grapple point. |
| GRAPPLE_TRADEOFF | -15% | speed | Movement speed penalty for 2s after grapple ends. |

### 6.2 Shield Dome

| Name | Value | Unit | Notes |
|---|---|---|---|
| SHIELD_COOLDOWN | 20.0 | s | Time between uses. |
| SHIELD_DURATION | 4.0 | s | How long the shield lasts. |
| SHIELD_RADIUS | 3.0 | m | Radius of the shield dome. |
| SHIELD_HP | 150 | HP | Damage the shield can absorb before breaking. |
| SHIELD_TRADEOFF | -20% | speed | Movement speed penalty while shield is active. |

### 6.3 Speed Boost

| Name | Value | Unit | Notes |
|---|---|---|---|
| SPEED_BOOST_COOLDOWN | 12.0 | s | Time between uses. |
| SPEED_BOOST_DURATION | 3.0 | s | How long the speed boost lasts. |
| SPEED_BOOST_MULTIPLIER | 1.4 | x | 40% speed increase. |
| SPEED_BOOST_TRADEOFF | +20% | damage taken | All incoming damage increased by 20% during boost. |

### 6.4 Thermal Vision

| Name | Value | Unit | Notes |
|---|---|---|---|
| THERMAL_COOLDOWN | 15.0 | s | Time between uses. |
| THERMAL_DURATION | 5.0 | s | How long thermal vision lasts. |
| THERMAL_RANGE | 30.0 | m | Maximum detection range. Enemies beyond this range are not highlighted. |
| THERMAL_TRADEOFF | -30% | FOV | Field of view reduced by 30% during thermal vision (tunnel vision effect). |

### 6.5 Phase Shift

| Name | Value | Unit | Notes |
|---|---|---|---|
| PHASE_COOLDOWN | 18.0 | s | Time between uses. |
| PHASE_DURATION | 2.0 | s | How long the phase shift lasts. |
| PHASE_TRADEOFF | full | vulnerability | Player cannot shoot, use abilities, or interact during phase. Movement only. |

---

## 7. Game Mode Constants

Source: `game_modes.md`

### 7.1 Deathmatch

| Name | Value | Unit | Notes |
|---|---|---|---|
| DM_PLAYER_COUNT | 8-12 | players | Free-for-all. |
| DM_KILL_LIMIT | 30 | kills | First player to reach this wins. |
| DM_TIME_LIMIT | 600 | s | 10 minutes. If kill limit not reached, highest kills wins. |
| DM_RESPAWN_TIME | 3.0 | s | Time before respawn after death. |

### 7.2 Team Deathmatch

| Name | Value | Unit | Notes |
|---|---|---|---|
| TDM_PLAYER_COUNT | 6v6 | players | Two teams. |
| TDM_KILL_LIMIT | 75 | kills | First team to reach this wins. |
| TDM_TIME_LIMIT | 600 | s | 10 minutes. |
| TDM_RESPAWN_TIME | 4.0 | s | Slightly longer than DM to emphasize team coordination. |

### 7.3 Elimination

| Name | Value | Unit | Notes |
|---|---|---|---|
| ELIM_PLAYER_COUNT | 3v3 or 5v5 | players | No respawn during round. |
| ELIM_ROUND_LIMIT | 5 | rounds | First team to win 5 rounds wins the match. |
| ELIM_ROUND_TIME | 120 | s | 2 minutes per round. |
| ELIM_RESPAWN_TIME | N/A | — | No respawn. Dead until round ends. |

### 7.4 Ranked (uses Elimination format)

| Name | Value | Unit | Notes |
|---|---|---|---|
| RANKED_PLAYER_COUNT | 5v5 | players | Fixed team size. |
| RANKED_ROUND_LIMIT | 7 | rounds | First to 4 round wins. |
| RANKED_ROUND_TIME | 120 | s | 2 minutes per round. |
| RANKED_RESPAWN_TIME | N/A | — | No respawn per round. |

---

## 8. Networking Constants

Source: `networking_spec.md`

| Name | Value | Unit | Notes |
|---|---|---|---|
| SERVER_TICK_RATE | 60 | Hz | Server updates per second. ~16.67ms per tick. |
| CLIENT_SEND_RATE | 60 | Hz | Client input sends per second. |
| REPLICATION_RATE | 20 | Hz | Server state replication to clients. Every 3rd server tick. |
| MAX_PING_KICK | 300 | ms | Players with sustained ping above this may be disconnected. |
| PREDCTION_WINDOW | 200 | ms | Maximum prediction time before forcing a rollback. |

---

## 9. Audio Constants

Source: `audio_design.md`

| Name | Value | Unit | Notes |
|---|---|---|---|
| FOOTSTEP_WALK_INTERVAL | 0.45 | s | Time between footstep sounds when walking. |
| FOOTSTEP_SPRINT_INTERVAL | 0.28 | s | Time between footstep sounds when sprinting. |
| FOOTSTEP_WALK_RANGE | 8.0 | m | Audible range of walk footsteps. |
| FOOTSTEP_SPRINT_RANGE | 20.0 | m | Audible range of sprint footsteps. |
| SLIDE_AUDIBLE_RANGE | 15.0 | m | Audible range of slide sound. |
| WALL_RUN_AUDIBLE_RANGE | 12.0 | m | Audible range of wall-run sound. |
| JUMP_AUDIBLE_RANGE | 8.0 | m | Audible range of jump sound. |
| DOUBLE_JUMP_AUDIBLE_RANGE | 10.0 | m | Audible range of double-jump sound. |
| RELOAD_AUDIBLE_RANGE | 12.0 | m | Audible range of reload sounds. |

---

## 10. Change Log

All changes to tuning constants must be logged here. Include the date, the constant changed, the old value, the new value, and the reason.

| Date | Constant | Old Value | New Value | Reason |
|---|---|---|---|---|
| 2026-05-05 | (Initial) | — | (All values above) | Initial design pass. All values subject to playtest revision. |

---

## 11. Notes for Designers

- **All values in this document are initial design targets.** They will be adjusted through playtesting. This document will be updated to reflect all changes.
- **Do not change constants in code without updating this document.** The document is the source of truth. Code mirrors the document, not the other way around.
- **When proposing a change, document the reason.** Future designers need to understand why a value was changed, not just what it was changed to.
- **Constants interact.** Changing WALK_SPEED affects slide balance, wall-run approach speed, and map traversal times. Consider cascading effects.
- **Playtest before committing.** A change that looks correct on paper may feel wrong in-game. Always validate with play sessions.
