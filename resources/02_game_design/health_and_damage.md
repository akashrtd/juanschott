# Health and Damage System [SYS-004]

**Document ID:** SYS-004
**Category:** Core Systems
**Status:** Design Complete
**Last Updated:** 2026-05-05
**Dependencies:** SYS-005 (Weapon System), SYS-006 (Add-on System), SYS-007 (Game Modes)

---

## 1. Design Philosophy

The health and damage system exists to answer one question: **how do players die, and how do they recover?** Every decision in this document traces back to three principles derived from the project pillars (see `player_experience.md`):

1. **Readability over depth.** No armor layers, no damage types, no elemental resistances. HP goes down, HP goes back up. Anyone can understand it in seconds.
2. **Time-to-kill that rewards skill without punishing aggression.** A headshot-focused player should win most fights, but the TTK window must be wide enough that movement and positioning matter.
3. **Minimal UI noise.** No floating damage numbers, no health bars above player models, no on-screen combat logs. The kill feed and hit markers are the only combat feedback vectors.

---

## 2. Health System

### 2.1 Base Health Pool

| Parameter | Value | Notes |
|---|---|---|
| Base HP | 100 | All players, all species. Identical base frame — no variance. |
| HP with Second Wind add-on | 125 | The only way to increase max HP. See `addon_system.md`. |
| Armor | None | No armor system. Deliberately omitted to preserve readability. |

**Rationale for uniform HP:** Species differentiation lives in movement traits and hitbox dimensions, not in health pools. Giving different HP values per species creates balancing nightmares and undermines readability. A player should never have to wonder "how many shots does this enemy take?" — the answer is always the same.

### 2.2 Regeneration

Health regeneration is the sole recovery mechanic. There are no health pickups, no healer roles, no passive regeneration items. This keeps the economy of engagement simple: take damage, disengage, recover, re-engage.

**Parameters:**

| Parameter | Value |
|---|---|
| Regen delay | 3 seconds after last weapon damage taken |
| Regen rate | 50 HP/s |
| Time from 1 HP to full | ~2.0 seconds |
| Time from 1 HP to full (125 HP) | ~2.5 seconds |

**Regen trigger rules:**

- The 3-second delay timer resets only on **weapon damage**. Environmental effects (zero-damage triggers, map hazard ticks that deal 0 damage) do not interrupt an active regeneration cycle.
- Once regeneration begins, it continues uninterrupted until the player reaches full HP. Only weapon damage received during regeneration halts the process and restarts the delay timer.
- The regen delay is configurable by add-ons (see `addon_system.md`). The base value of 3 seconds can be reduced or increased by specific add-on selections, allowing players to tune their sustain.

**Edge case handling:**

- If a player takes damage at exactly 3.000 seconds (within the same frame as regen activation), regen is canceled and the timer restarts. Server tick resolution handles this.
- If a player is at full HP, the regen system is dormant. No unnecessary computation.
- Regen is paused while the player is dead or in the respawn queue.

### 2.3 Health Feedback

Visual and audio feedback for health state is minimal by design. The player should *feel* their health state through atmosphere, not through numbers.

**Visual feedback:**

- Screen edge vignette (red tint) scales in intensity based on missing HP. At 1 HP the vignette is prominent; at 100 HP it is invisible.
- Vignette fades smoothly as regeneration restores HP. The fade-out is a direct 1:1 mapping — at 50 HP the vignette is half as strong as at 1 HP.
- No health bar rendered on the player model. The 3D space must remain clean and readable for all players.
- Health is displayed as a numeric value in the HUD (bottom-left corner, standardized across all HUD layouts).

**Audio feedback:**

- A subtle "healing pulse" sound plays when regeneration begins. This sound is audible only to the regenerating player. It is a single, short tone — not a sustained loop.
- No audio plays during the regen process itself. The pulse serves as confirmation that regen has started; the visual vignette fade provides ongoing feedback.
- At low HP (below 25), a faint heartbeat sound loops. This stops the moment regen begins.

---

## 3. Damage System

### 3.1 Damage Sources

All damage in the game originates from one of four source types. Each type has distinct registration mechanics:

| Source Type | Registration | Examples |
|---|---|---|
| Hitscan | Instant raycast on fire | Assault rifle, pistol, sniper |
| Projectile | Travel-time entity with collision | Grenade launcher, rocket |
| Beam | Continuous tick-based | Laser, plasma cutter |
| Environmental | Trigger volume or physics event | Fall damage, map hazards |

**Server authority:** All damage is calculated and confirmed server-side. The client sends hit registration requests; the server validates and applies damage. Client-side prediction handles visual feedback (hit markers, blood effects) immediately for responsiveness, but damage values are authoritative from the server.

### 3.2 Damage Application

When a hit is registered:

1. Server calculates final damage (base damage × headshot multiplier × falloff).
2. Server subtracts damage from target's current HP.
3. Server sends confirmation to both attacker and victim.
4. Attacker receives: hit marker on crosshair + audio cue.
5. Victim receives: directional damage vignette (screen edge flash from the direction of the attacker).

**Damage numbers are never displayed.** No floating text, no on-screen damage counters. This is a Pillar 4 decision — clean and readable combat. Players judge weapon effectiveness through TTK feel, not through numbers.

### 3.3 Headshot System

Headshots are the primary skill expression vector in the damage system.

| Parameter | Value |
|---|---|
| Headshot multiplier | 2.5× (all weapons) |
| Head hitbox zone | Top 20% of player capsule collider |
| Headshot hit marker | Distinct visual (sharper, brighter flash) and audio (crisp "ping") |

**Why 2.5× multiplier:**

The multiplier is calibrated around the assault rifle baseline:

- Assault rifle body shot: 18 damage
- Assault rifle headshot: 18 × 2.5 = 45 damage
- Kill from full HP (100): 3 headshots = 135 damage ✓
- Kill from full HP with Second Wind (125): 3 headshots = 135 damage ✓

At 2.5×, headshots are rewarding but not instant. A player must land three consecutive headshots for a kill, which creates a TTK window long enough for the target to react, reposition, or return fire. This preserves the "breathing room" tenet of the combat pacing.

**Why the same multiplier for all weapons:**

Different multipliers per weapon would mean players need to learn weapon-specific headshot values, undermining readability. A 2.5× headshot multiplier on every weapon means the player always knows roughly how many headshots are needed. The variable is body-shot damage per weapon, which is intuitive.

**Head hitbox definition:**

The head zone is defined as the upper 20% of the capsule collider's vertical extent. This is a fixed ratio, not a separate mesh collider. Benefits:

- Consistent across all player orientations and animation states.
- Easy to debug and visualize in development.
- Not affected by cosmetic or species model differences.

### 3.4 Damage Falloff

Falloff reduces weapon damage at long range for hitscan weapons only. Projectiles use travel time as their natural range limiter. Beams use a fixed range cutoff (damage drops to 0 instantly at max range).

**Falloff formula:**

```
if distance <= falloff_start:
    multiplier = 1.0
elif distance >= falloff_end:
    multiplier = min_damage_ratio
else:
    t = (distance - falloff_start) / (falloff_end - falloff_start)
    multiplier = lerp(1.0, min_damage_ratio, t)

final_damage = base_damage × multiplier × (headshot_multiplier if headshot)
```

Specific `falloff_start`, `falloff_end`, and `min_damage_ratio` values are defined per weapon in `weapon_system.md`.

**Falloff applies after headshot multiplier.** This means a distant headshot still deals more damage than a distant body shot, but both are reduced by the same falloff factor.

### 3.5 Splash Damage

Splash damage applies to projectile weapons with explosive payloads. It is the only form of area-of-effect damage in the game.

**Splash calculation:**

```
distance = player_distance_from_impact_point
if distance > splash_radius:
    splash_damage = 0
elif distance == 0:
    splash_damage = direct_hit_damage + full_splash_damage
else:
    t = distance / splash_radius
    splash_damage = full_splash_damage × (1.0 - t)
```

**Direct hits:** A direct hit deals full projectile damage plus full splash damage. The two are additive, not exclusive. This rewards precision even with explosive weapons.

**Self-damage:** The firer receives 50% of the splash damage they would have taken at their distance from the impact point. This is a deliberate design choice:

- 100% self-damage makes explosive weapons feel punishing and discourages creative close-range use.
- 0% self-damage removes the skill ceiling of spacing your explosions.
- 50% is the middle ground: close-range rockets are risky but not suicidal. A player with 60 HP who rocket-jumps or point-blank rockets takes meaningful damage but can survive with smart play.

**Friendly fire:** Splash damage to teammates is controlled by game mode rules. See `game_modes.md` for per-mode friendly fire settings.

---

## 4. Death and Respawn

### 4.1 Death Event

Death occurs instantly when HP reaches 0. There is no "downed" state, no bleed-out timer, no revive mechanic. This is intentional — the game's pace is maintained by keeping players either fully alive or fully dead.

**On death:**

1. Player's HP is clamped to 0. No overkill tracking (no reason to track excess damage).
2. Death camera activates: a 1.0-second view from the dying player's perspective, centered on the killer. The camera does not follow the killer after death — it freezes on the last known position.
3. Kill is confirmed on the attacker's screen: a clean HUD notification (player name + weapon icon) with a short audio cue (distinct from the hit marker sound).
4. Kill feed updates: the event is pushed to the kill feed on the right side of all players' screens.

### 4.2 Kill Feed

The kill feed is the only persistent combat log visible to players.

**Specifications:**

- **Position:** Right side of screen, vertically stacked, newest on top.
- **Capacity:** Shows the last 5 kills.
- **Fade behavior:** Each entry fades after 4 seconds. Faded entries are removed from display.
- **Entry format:** `[Killer Name] [Weapon Icon] [Victim Name]`
- **Headshot indicator:** A small headshot icon is appended to headshot kills.
- **Assist indicator:** If an assist was involved, the format becomes `[Killer Name] + [Assister Name] [Weapon Icon] [Victim Name]`

### 4.3 Respawn System

**Respawn timers:**

| Mode | Timer |
|---|---|
| 1v1 (Duel) | 3 seconds |
| 3v3 | 4 seconds |
| 6v6 | 5 seconds |

**Rationale for scaling timers:** Larger modes have more players creating pressure. Longer respawn timers in 6v6 give the surviving team a meaningful window to capitalize on a kill (push a point, secure a zone). In 1v1, a 3-second death is already a long time to be absent from the fight.

**Respawn location:**

- Players respawn at a randomized position within their team's designated spawn zone.
- Spawn zones are defined as rectangular volumes in the map, not fixed points. This prevents spawn camping at specific locations.
- Randomization is weighted: positions further from enemy players receive higher weight, reducing the chance of spawning in immediate danger.
- Spawn zones may shift dynamically in certain game modes (see `game_modes.md`).

**Spawn protection:**

- Duration: 0.5 seconds of full invulnerability after respawn.
- Visual indicator: a faint glow on the respawning player's model, visible to all players.
- The glow fades over the last 0.1 seconds as a warning that protection is ending.
- Protected players can deal damage but cannot capture objectives. Protection is canceled immediately if the protected player fires a weapon.

**Loadout changes:**

- During the respawn timer, players can access the loadout menu to change their weapon and add-on selection.
- Changes take effect on respawn. If no changes are made, the previous loadout is retained.
- The loadout menu is not accessible while alive — this prevents mid-combat switching and preserves the commitment of loadout choice.

---

## 5. Assist System

The assist system ensures that players who contribute to a kill are recognized, even if they don't land the final shot.

**Assist trigger conditions:**

1. Player A deals damage to an enemy.
2. Player B (teammate of A) kills that enemy within 3 seconds of Player A's last damage instance.
3. Player A receives an assist credit.

**Assist reward:**

- Score: 50% of the kill score (varies by game mode — see `game_modes.md`).
- Kill feed: assist is displayed alongside the kill entry.
- HUD notification: "Assist" text appears briefly on Player A's screen.

**Design intent:**

The assist system exists to eliminate "kill steal" frustration. In a team game, any damage that leads to a kill is valuable. By awarding 50% score and clearly displaying the assist, the system communicates that team play is valued equally to securing the final shot. The 3-second window is generous enough to cover sustained team fights without crediting unrelated earlier engagements.

**Edge cases:**

- If two players both damage an enemy within 3 seconds of death, both receive assist credit.
- Self-damage and environmental damage do not generate assist credits.
- If the damager and the killer are the same player, no assist is generated (it's just a kill).

---

## 6. System Integration

This system interacts with several other systems. The following table summarizes touchpoints:

| Connected System | Interaction | Reference |
|---|---|---|
| Weapon System | Base damage, falloff curves, headshot multiplier applied here | `weapon_system.md` |
| Add-on System | Second Wind (+25 HP), regen delay modifiers | `addon_system.md` |
| Game Modes | Respawn timers, friendly fire, score values | `game_modes.md` |
| Movement System | Fall damage thresholds, capsule collider dimensions | `movement_system.md` |
| HUD / UI | Health display, vignette, kill feed, hit markers | `player_experience.md` |

---

## 7. Open Questions

| # | Question | Status |
|---|---|---|
| 1 | Should beam weapons apply headshot multipliers? Currently yes (same system), but beam tracking makes headshots trivial. | Under discussion |
| 2 | Should self-damage from projectiles be affected by add-ons (e.g., reduced self-damage)? | Deferred to add-on system design |
| 3 | What is the exact numeric threshold for "low HP" heartbeat audio? Current assumption: 25 HP. | Needs playtesting |
| 4 | Should spawn protection prevent all damage or only lethal damage? Current assumption: all damage. | Needs playtesting |

---

## 8. Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-05-05 | Design Team | Initial document |

---

**See Also:** `weapon_system.md` · `addon_system.md` · `game_modes.md` · `player_experience.md` · `movement_system.md`
