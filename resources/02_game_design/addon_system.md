# Add-On System [SYS-003]

> **Document ID:** SYS-003
> **Version:** 1.0.0
> **Status:** Draft
> **Author:** Design Team
> **Last Updated:** 2026-05-05
> **Related Systems:** [Weapon System](weapon_system.md), [Health & Damage](health_and_damage.md), [Movement System](movement_system.md), [Tuning Constants](../06_cross_cutting/tuning_constants.md)

---

## 1. Overview

The Add-On System is the sole layer of player customization in ASTRA. Every player spawns with the same base frame — identical health, movement, and weapon handling. There are no heroes, no classes, and no talent trees. Customization and strategic expression come entirely from a three-slot add-on loadout chosen before each spawn.

This system must accomplish three things simultaneously:

1. **Provide meaningful choice.** Each slot represents a real decision with tangible in-game consequences. No add-on should be an automatic pick.
2. **Remain readable.** Opponents must be able to identify what a player is running through visual and audio cues alone. No hidden power.
3. **Guarantee counter-play.** For every add-on, there must be a viable response. No combination should be unbeatable.

The add-on system is not a progression system. All add-ons are available to all players from the start. Unlocking, leveling, or gating content behind playtime is outside the scope of this document.

---

## 2. Design Philosophy

### 2.1 Flat Baseline

All players share an identical base frame. No stats differ between players before add-ons are applied. This ensures that mechanical skill and strategic decision-making — not loadout advantage — determine outcomes. The base frame specifications are:

| Attribute | Base Value |
|---|---|
| Health | 100 HP |
| Sprint Speed | 7.0 m/s |
| Regen Delay | 3 seconds |
| Regen Rate | 10 HP/s |
| Double Jump | Yes (1 charge) |
| Wall Run Duration | 1.5 seconds |

### 2.2 Three Slots, Three Decisions

Every loadout consists of exactly three add-on slots. Each slot holds one add-on. Slots are not typed — any add-on can go in any slot. This means a player can run three damage add-ons, three movement add-ons, or any other combination. The constraint is purely numerical: three picks, no more, no less.

While same-category stacking is allowed, tradeoffs are designed to make pure stacking suboptimal in most situations. A player running three damage add-ons will lack mobility and survivability. A player running three health add-ons will lack offensive pressure and informational tools. Mixed loadouts are generally stronger, but niche all-in builds have situational viability.

### 2.3 Mixed Passive and Active

Add-ons are split between passive and active types:

- **Passive** add-ons are always active. They modify stats or behaviors without player intervention. Their tradeoffs are baked into the stat changes.
- **Active** add-ons require deliberate activation via a key press. They have cooldowns and durations. Their tradeoffs are concentrated into the activation window.

This mix ensures that loadout expression isn't purely mechanical (choosing when to press buttons) or purely strategic (choosing stat profiles). Both dimensions matter.

### 2.4 Full Readability

Every add-on produces visible and audible tells. Opponents should be able to read a player's loadout within the first few seconds of engagement by observing:

- Persistent visual indicators (glows, trails, auras)
- Activation effects (flashes, pulses, distinct sounds)
- Behavioral patterns (movement anomalies, damage quirks)

This is not optional. If an add-on cannot be made readable without adding a HUD element, the add-on needs a redesign.

### 2.5 Universal Counter-Play

No add-on or combination of add-ons may lack a counter. Counter-play must be achievable through positioning, timing, or loadout selection. If a counter only exists at the loadout screen (i.e., "pick X to beat Y"), the system has failed. In-game counters must always exist.

---

## 3. Slot System

### 3.1 Slot Rules

| Rule | Detail |
|---|---|
| Slot count | 3 per loadout |
| Slot typing | None — any add-on fits any slot |
| Stacking | Same-category add-ons allowed in multiple slots |
| Duplication | The same add-on cannot be equipped twice |
| Swapping | Add-ons can be changed on the loadout screen or on death |

### 3.2 Selection Flow

1. Player opens loadout screen (pre-match or on death).
2. Player selects up to three add-ons from the full catalog.
3. Loadout is locked on spawn. No mid-life changes.
4. On death, player returns to loadout screen and may reconfigure.

### 3.3 Stacking Tradeoffs

Stacking same-category add-ons is permitted but each add-on carries its own tradeoff. Since tradeoffs are not category-aligned, stacking amplifies downsides:

- Three damage add-ons would reduce overall sustained DPS, remove mobility tools, and eliminate survivability.
- Three movement add-ons would make the player extremely evasive but unable to deal meaningful pressure or survive focused fire.
- Three health add-ons would create a durable but predictable target with no offensive tools or informational advantage.

The goal is for mixed loadouts to be the default, with niche stacking builds serving as situational gambits rather than optimal strategies.

---

## 4. Add-On Catalog

### 4.1 Damage Add-Ons

---

#### [ADD-D001] Kinetic Amplifier

| Field | Value |
|---|---|
| **Type** | Passive |
| **Category** | Damage |
| **Effect** | First shot after 1.5 seconds of not firing deals +15% damage |
| **Tradeoff** | All subsequent shots in the same burst deal -5% damage |
| **Stacking** | Bonus does not compound across slots — only one Kinetic Amplifier can be equipped |

**Behavior:**
The kinetic amplifier rewards disciplined, deliberate fire. After the player stops shooting for 1.5 seconds, the next shot is enhanced. If the player continues firing, each shot after the first is slightly weaker than baseline. This creates a tempo pattern: fire, pause, fire, pause. Players who hold trigger lose overall DPS.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Weapon barrel emits a faint glow when the first-shot bonus is charged |
| Audio | Subtle low-frequency hum audible at close range when charged |
| Behavioral | Opponent fires slowly and deliberately rather than in sustained bursts |

**Counter-Play:**
Push aggressively to force premature shots. The amplifier loses value if the player is pressured into sustained fire. Close the distance — the bonus only applies to one shot, so forcing a spray fight neutralizes the advantage.

---

#### [ADD-D002] Explosive Rounds

| Field | Value |
|---|---|
| **Type** | Active |
| **Category** | Damage |
| **Cooldown** | 15 seconds |
| **Duration** | 5 seconds |
| **Effect** | Bullets deal splash damage (2-meter radius, 30% of bullet damage) |
| **Tradeoff** | 20% reduced direct-hit damage during active duration |
| **Activation** | Dedicated add-on activation key |

**Behavior:**
For 5 seconds after activation, every bullet that impacts a surface or target creates a small explosion in a 2-meter radius. The explosion deals 30% of the bullet's base damage to all targets in range, including the original target. Direct-hit damage is reduced by 20% during this window, making the add-on weaker in 1v1 duels but stronger against grouped enemies or for area denial.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Weapon glows orange during active; explosions leave temporary scorch marks on surfaces |
| Audio | Distinct explosive impact sound (different from grenades); activation chime audible to nearby players |
| Behavioral | Player fires at floors and walls near targets rather than directly at targets |

**Counter-Play:**
Stay spread out. Explosive Rounds reward clustering. In a 1v1, the reduced direct-hit damage makes the user weaker — press them in isolated duels. The 15-second cooldown creates a 10-second window of baseline damage where the add-on provides no benefit.

---

#### [ADD-D003] Piercing Shot

| Field | Value |
|---|---|
| **Type** | Passive |
| **Category** | Damage |
| **Effect** | Bullets pass through the first target and can hit a second target at 50% damage |
| **Tradeoff** | -10% damage to the first (primary) target |
| **Pierce limit** | 1 additional target per bullet |

**Behavior:**
Every bullet that hits a target continues traveling and can strike a second target behind the first. The second target takes 50% of the bullet's original damage (before the -10% penalty is applied to the first target). Bullets do not pierce walls or geometry — only players. This add-on excels when enemies are lined up and punishes careless positioning.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Tracer rounds leave a visible trail that continues through hit targets |
| Audio | Distinct "pass-through" sound on pierce — a wetter, heavier impact |
| Behavioral | Player positions to angle shots through multiple enemies |

**Counter-Play:**
Do not stand in lines. Use vertical positioning — stacking above and below rather than side by side. Spread horizontally in team fights. Against a solo player, this add-on provides no benefit beyond the tradeoff, making it a net negative in 1v1 duels.

---

#### [ADD-D004] Overcharge

| Field | Value |
|---|---|
| **Type** | Active |
| **Category** | Damage |
| **Cooldown** | 20 seconds |
| **Duration** | 3 seconds |
| **Effect** | Fire rate increased by 40% for 3 seconds |
| **Tradeoff** | Weapon overheats after duration — 1.5-second full lockout where player cannot fire |
| **Activation** | Dedicated add-on activation key |

**Behavior:**
Overcharge dumps raw fire rate for a brief window. The player fires 40% faster for 3 seconds, then their weapon locks out entirely for 1.5 seconds. This creates a high-risk burst window — if the player doesn't secure a kill during the overcharge, they are defenseless during lockout. The lockout is absolute: no shooting, no reloading, no melee.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Weapon vents steam and glows red during overcharge; visible mechanical shutdown animation during lockout |
| Audio | Ramp-up whine during overcharge; distinct mechanical click and hiss at lockout start |
| Behavioral | Player commits aggressively for 3 seconds, then goes silent |

**Counter-Play:**
Survive the burst. Use cover, health add-ons, or movement to ride out the 3-second window. Once lockout triggers, the player is completely vulnerable for 1.5 seconds — that is the punish window.

---

### 4.2 Movement Add-Ons

---

#### [ADD-M001] Dash

| Field | Value |
|---|---|
| **Type** | Active |
| **Category** | Movement |
| **Cooldown** | 10 seconds |
| **Duration** | 0.2 seconds |
| **Effect** | Instant horizontal burst of 8 meters in the current movement direction |
| **Tradeoff** | Cannot shoot during dash; 0.3-second recovery after dash before shooting resumes |
| **Activation** | Double-tap movement direction key |

**Behavior:**
The player warps 8 meters in the direction they are currently moving. If stationary, the dash goes in the direction the player is facing. The dash is extremely fast (0.2 seconds) but leaves a 0.3-second recovery window where the player cannot fire. Total vulnerability: 0.5 seconds from activation to ready-to-fire.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Motion blur trail along the dash path; dust burst at start and end points |
| Audio | Distinct whoosh sound; footstep skip audible |
| Behavioral | Player suddenly repositions — predict the endpoint |

**Counter-Play:**
Pre-aim the dash endpoint. The dash direction is readable from the player's movement before activation. Predictive aiming at the end position punishes the recovery window. Area-denial tools (Explosive Rounds, grenades) placed at likely dash destinations are effective.

---

#### [ADD-M002] Grapple Hook

| Field | Value |
|---|---|
| **Type** | Active |
| **Category** | Movement |
| **Cooldown** | 12 seconds |
| **Duration** | Variable (travel time to target, ~0.8s average) |
| **Effect** | Fire a grappling hook that pulls the player to the target surface (max 25m range) |
| **Tradeoff** | Cannot shoot while grappling; vulnerable during travel |
| **Activation** | Press activation key while aiming at a valid surface |

**Behavior:**
The player fires a visible hook/beam at a surface. Once attached, the player is pulled along the line to the surface. Travel time depends on distance (~0.8 seconds at maximum range, faster at close range). The player cannot fire, reload, or use other add-ons during travel. On arrival, there is a 0.2-second landing recovery before normal actions resume.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Visible rope/beam from player to target surface; player model visibly pulled through the air |
| Audio | Grapple fire sound on launch; tension/whirring sound during travel |
| Behavioral | Player aims upward before firing — telegraphs intent to grapple |

**Counter-Play:**
Shoot them mid-flight. Grappling players are committed to a predictable trajectory for up to 0.8 seconds. They cannot return fire. A grappling player is the easiest target in the game. If they grapple away, let them go — they've used a 12-second cooldown to escape, and you've gained positioning advantage.

---

#### [ADD-M003] Wall Cling

| Field | Value |
|---|---|
| **Type** | Passive |
| **Category** | Movement |
| **Effect** | Can cling to any wall for up to 2 seconds; extends wall-run time by 2 seconds or allows stationary wall cling |
| **Tradeoff** | Cannot shoot while clinging; 0.5-second delay after releasing cling before player can fire |
| **Activation** | Automatic on wall contact (hold movement into wall) |

**Behavior:**
When a player contacts a wall (during a wall-run or by jumping into a wall), they can hold position for up to 2 seconds. This can extend a wall-run by 2 additional seconds or allow a completely stationary cling. The player can look around while clinging but cannot fire. Upon release, there is a 0.5-second delay before shooting resumes.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Visible energy anchors or grip effect at wall contact points on player's hands/feet |
| Audio | Distinct attach/detach sound — a mechanical clamping noise |
| Behavioral | Player stops moving on a wall — a clear and unusual visual |

**Counter-Play:**
They cannot shoot while clinging. Push them. A clinging player is a stationary target with delayed fire on exit. Pre-aim the wall they're clinging to. Force them to drop by threatening the wall's base or flanking their cling position.

---

#### [ADD-M004] Momentum Shift

| Field | Value |
|---|---|
| **Type** | Active |
| **Category** | Movement |
| **Cooldown** | 14 seconds |
| **Duration** | Instant |
| **Effect** | Instantly reverse horizontal momentum — full speed in the opposite direction |
| **Tradeoff** | Brief 0.2-second stagger on use; cannot shoot during stagger |
| **Activation** | Dedicated add-on activation key |

**Behavior:**
The player instantly reverses their horizontal direction at full speed. If running forward, they are now running backward at full sprint speed. This is useful for juking, escaping, or rapidly changing direction in a fight. The 0.2-second stagger is minimal but noticeable — the player model flinches briefly.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Brief afterimage at the reversal point; small dust puff |
| Audio | Distinct snap/crack sound at reversal |
| Behavioral | Sudden direction change that defies normal deceleration physics |

**Counter-Play:**
The reversal point is telegraphed by the snap sound and is predictable — they can only reverse, not change to an arbitrary direction. Aim at where they were, not where they're going — the stagger window is punish-able with area damage or predictive fire.

---

### 4.3 Health Add-Ons

---

#### [ADD-H001] Fortify

| Field | Value |
|---|---|
| **Type** | Active |
| **Category** | Health |
| **Cooldown** | 20 seconds |
| **Duration** | 3 seconds |
| **Effect** | Take 50% reduced damage for 3 seconds |
| **Tradeoff** | Movement speed reduced by 30% during active duration |
| **Activation** | Dedicated add-on activation key |

**Behavior:**
On activation, the player receives 50% damage reduction for 3 seconds. During this time, movement speed drops from 7.0 m/s to 4.9 m/s. The player can still shoot, jump, and use other add-ons normally — only movement is penalized.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Player model glows with a faint hexagonal shield overlay |
| Audio | Shield activation hum; incoming hits produce a metallic "clink" instead of normal flesh impact |
| Behavioral | Player noticeably slows down and becomes more deliberate |

**Counter-Play:**
Wait it out. Three seconds is short. Reposition during their slow phase — they cannot chase effectively. If they use Fortify to push through, they arrive slowed and you have positional advantage. Burst damage after the duration ends is highly effective.

---

#### [ADD-H002] Second Wind

| Field | Value |
|---|---|
| **Type** | Passive |
| **Category** | Health |
| **Effect** | Maximum HP increased by 25 (total 125 HP instead of 100 HP) |
| **Tradeoff** | Health regeneration delay increased from 3 seconds to 5 seconds |
| **Activation** | Always active |

**Behavior:**
The player has a larger health pool at the cost of slower recovery between fights. In a single engagement, 125 HP vs 100 HP is a significant advantage. Between engagements, the 5-second regen delay means the player must wait longer before re-entering. This add-on favors players who win fights cleanly and take little damage — the extra HP is a buffer, not a heal.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Health bar is visibly wider in the HUD; opponents may notice prolonged survival via damage numbers |
| Audio | No distinct audio tell — opponents must infer from TTK (time to kill) anomalies |
| Behavioral | Player takes noticeably more damage to down, especially with precision weapons |

**Counter-Play:**
Burst damage. High-damage weapons and combos that would kill a 100 HP player in one burst will still leave this player alive — but barely. Follow up immediately. Between fights, pressure them so they never get the 5 seconds they need to start regenerating.

---

#### [ADD-H003] Adaptive Regeneration

| Field | Value |
|---|---|
| **Type** | Passive |
| **Category** | Health |
| **Effect** | Regeneration delay reduced from 3 seconds to 1.5 seconds |
| **Tradeoff** | Movement speed reduced by 15% (sprint: 5.95 m/s instead of 7.0 m/s) |
| **Activation** | Always active |

**Behavior:**
The player recovers from damage significantly faster than baseline — regen kicks in after only 1.5 seconds instead of 3. This makes disengage-and-recover strategies much more effective. The tradeoff is a permanent speed reduction that affects every aspect of mobility: sprinting, dodging, rotating, and escaping.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Faint green pulse around player model when regen activates |
| Audio | Subtle regen activation sound audible at close range |
| Behavioral | Player is noticeably slower in all movement; tends to disengage from fights frequently |

**Counter-Play:**
Aggressive pushing. Do not let them get 1.5 seconds of breathing room. Chase relentlessly. The speed penalty means they cannot escape a determined pursuer. If they try to disengage, they move at 5.95 m/s — close the gap and finish them.

---

#### [ADD-H004] Pain Response

| Field | Value |
|---|---|
| **Type** | Passive |
| **Category** | Health |
| **Effect** | On taking damage below 30 HP, gain a brief 25% speed boost for 2 seconds (triggers once per life) |
| **Tradeoff** | Maximum HP reduced by 10 (total 90 HP instead of 100 HP) |
| **Activation** | Automatic when HP drops below 30 |

**Behavior:**
When the player's HP drops below 30, they receive a one-time 2-second speed burst at 125% of base sprint speed (8.75 m/s). This can be used to escape a losing fight or to close distance for a desperate final play. The lower max HP of 90 makes the player easier to kill in general, and the speed boost only triggers once — if they survive, they have no additional tool.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Brief red flash/pulse on the player model when the speed boost triggers |
| Audio | Adrenaline-rush sound — sharp intake of breath audible to nearby players |
| Behavioral | Player suddenly accelerates when near death |

**Counter-Play:**
Burst them from above 30 HP to dead in a single shot or burst. If they trigger the speed boost, they are still at low HP — one clean hit finishes them. Predict the escape direction and cut them off. The lower max HP means they die faster to everything.

---

### 4.4 Perk Add-Ons

---

#### [ADD-P001] Echo Location

| Field | Value |
|---|---|
| **Type** | Active |
| **Category** | Perk |
| **Cooldown** | 25 seconds |
| **Duration** | 4 seconds |
| **Effect** | Wallhack pulse reveals enemy positions through walls for 4 seconds |
| **Tradeoff** | A visible pulse emanates from the player, revealing their position to all enemies |
| **Activation** | Dedicated add-on activation key |

**Behavior:**
On activation, a sonar-like pulse expands from the player's position. For 4 seconds, all enemy positions are visible as highlighted outlines through geometry. However, the pulse itself is visible to all players — enemies see the wave originate from the user's position. The user gains information but loses the element of surprise.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Expanding wave pulse visible from player position; enemy outlines glow through walls |
| Audio | Distinct sonar ping audible to all players map-wide |
| Behavioral | Player suddenly changes direction or begins aiming at targets through walls |

**Counter-Play:**
The pulse reveals the user's position. Rush them while they're looking at their wallhack — they're distracted processing information. Alternatively, use the knowledge that they know where you are to set a trap. The 25-second cooldown means 21 seconds of blindness after the effect ends.

---

#### [ADD-P002] Footstep Reader

| Field | Value |
|---|---|
| **Type** | Passive |
| **Category** | Perk |
| **Effect** | See recent enemy footprints as faint glowing marks on surfaces (last 3 seconds of movement) |
| **Tradeoff** | Cannot use any other perk add-on (this occupies the entire perk slot with a stronger effect) |
| **Activation** | Always active |

**Behavior:**
Enemy movement leaves visible traces on surfaces for 3 seconds. These traces appear as faint colored marks showing direction and recency (brighter = more recent). Wall-running and double-jumping leave fewer or no traces. The effect is strong enough that it occupies a full perk-category slot — the player cannot run another perk add-on alongside this one.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Faint glowing marks on ground and walls where enemies have walked; direction indicated by mark orientation |
| Audio | No audio tell — opponent must infer from the player's tracking behavior |
| Behavioral | Player consistently finds and flanks enemies without direct line of sight |

**Counter-Play:**
Use vertical movement. Wall-runs, double-jumps, and grapples leave no footprints. Vary your routes. If you suspect a Footstep Reader is tracking you, double back or go vertical. The 3-second window is short — fast movement outruns the traces.

---

#### [ADD-P003] Decoy

| Field | Value |
|---|---|
| **Type** | Active |
| **Category** | Perk |
| **Cooldown** | 18 seconds |
| **Duration** | 5 seconds |
| **Effect** | Deploy a holographic copy that runs in a designated direction for 5 seconds |
| **Tradeoff** | Cannot shoot during 0.3-second deployment cast time |
| **Activation** | Dedicated add-on activation key + directional input |

**Behavior:**
The player deploys a holographic duplicate that runs in the specified direction. The decoy has no collision, no hitbox, and cannot deal damage. It mimics the player's model exactly, including current add-on visual effects. At close range, a subtle shimmer is visible on the decoy. The decoy produces fake footstep sounds.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Deployment flash at player position; decoy has subtle shimmer visible within ~5 meters |
| Audio | Fake footstep sounds; deployment sound audible to nearby players |
| Behavioral | Two identical player models moving in different directions |

**Counter-Play:**
Look for the shimmer at close range. Watch for lack of collision — bullets pass through the decoy, it doesn't interact with physics objects. Impact effects (bullet holes, blood) won't appear on the decoy. If you're unsure, shoot both — the real one bleeds.

---

#### [ADD-P004] Tag Round

| Field | Value |
|---|---|
| **Type** | Active |
| **Category** | Perk |
| **Cooldown** | 15 seconds |
| **Duration** | Tag persists for 6 seconds |
| **Effect** | Next bullet that hits an enemy tags them — their position is revealed to your team through walls for 6 seconds |
| **Tradeoff** | Tagged enemy is alerted they've been tagged (visual/audio warning) |
| **Activation** | Dedicated add-on activation key (buffs next shot) |

**Behavior:**
The player activates Tag Round, and the next bullet that connects with an enemy applies a 6-second tracking mark. The marked player's position is visible to the entire user's team through walls. The tagged player receives a clear warning that they've been marked — they know they're being tracked and can adjust behavior accordingly.

**Readability:**

| Cue | Detail |
|---|---|
| Visual | Tagged player emits a visible marker/outline; activation produces a brief muzzle flash in a distinct color |
| Audio | Tag application produces a distinct "ping" sound heard by the tagged player and nearby enemies |
| Behavioral | Enemy team suddenly converges on the tagged player's position |

**Counter-Play:**
When tagged, play defensively for 6 seconds. Use the warning to reposition into a strong defensive position. Alternatively, use yourself as bait — let the enemy team chase your known position while your team flanks. The 15-second cooldown limits tagging frequency.

---

## 5. Counter-Play Matrix

The add-on system is designed around a cyclical counter relationship between categories. No single category dominates all others. The counter cycle is:

```
Damage  →  Health   (overcome extra HP and damage reduction through raw output)
Health  →  Perk     (survive the informational advantage with durability)
Perk    →  Movement (track and predict repositioning with information)
Movement→  Damage   (dodge and reposition away from damage boosts and burst)
```

| | **Damage Enemy Has** | **Movement Enemy Has** | **Health Enemy Has** | **Perk Enemy Has** |
|---|---|---|---|---|
| **You Run Damage** | Mirror — skill matchup | You struggle — they can dodge your burst | You excel — raw damage overcomes their durability | Even — you can kill them before intel matters |
| **You Run Movement** | You excel — you can dodge their damage | Mirror — skill matchup | You struggle — can't burst through their durability | Even — you can escape their intel plays |
| **You Run Health** | You struggle — they output more than you can tank | Even — you're slow but tanky enough to survive repositioning | Mirror — stat check | You excel — you survive their intel-driven attacks |
| **You Run Perk** | Even — intel helps but you're fragile | You excel — you can track their movement | You struggle — you know where they are but can't kill them | Mirror — information warfare |

This matrix is a guideline, not a hard counter chart. Player skill, map knowledge, and situational awareness always trump loadout advantages. The matrix describes statistical tendencies, not guarantees.

---

## 6. Example Loadouts

### 6.1 Aggressor Build

| Slot | Add-On | Role |
|---|---|---|
| 1 | **Kinetic Amplifier** | Damage — first-shot burst for opening picks |
| 2 | **Dash** | Movement — close distance or escape after committing |
| 3 | **Fortify** | Health — survive the push when closing distance |

**Strategy:** This build is designed for aggressive entry. Kinetic Amplifier rewards the first shot in every engagement, making ambushes and quick peeks lethal. Dash provides the mobility to close distance or reposition after committing. Fortify gives a 3-second damage reduction window for pushing through choke points or surviving a direct duel. The player excels at initiating fights and has tools for both entry and escape.

**Weakness:** Information. The Aggressor has no way to know where enemies are before pushing. Against a Perk-heavy opponent who can see them coming, the Aggressor can be baited into bad engagements.

---

### 6.2 Recon Build

| Slot | Add-On | Role |
|---|---|---|
| 1 | **Piercing Shot** | Damage — punish enemies who cluster after being revealed |
| 2 | **Echo Location** | Perk — reveal enemy positions for team coordination |
| 3 | **Adaptive Regeneration** | Health — quick recovery after taking damage from aggressive pushes |

**Strategy:** This build is an information hub. Echo Location reveals enemy positions, Piercing Shot punishes enemies who line up or cluster in response, and Adaptive Regeneration allows quick recovery between engagements. The player functions as a scout and support — they don't need to win every duel, they need to provide information and opportunistic damage.

**Weakness:** Mobility. The Recon player is slow (Adaptive Regeneration penalty) and has no movement tool. If caught in a bad position, they cannot escape. Aggressive players with Dash or Grapple can close on them and exploit the lack of mobility.

---

### 6.3 Anchor Build

| Slot | Add-On | Role |
|---|---|---|
| 1 | **Explosive Rounds** | Damage — area denial and chip damage through splash |
| 2 | **Wall Cling** | Movement — hold elevated positions with extended wall time |
| 3 | **Second Wind** | Health — 125 HP makes dislodging the player more difficult |

**Strategy:** This build holds space. The player finds an elevated or hard-to-reach position using Wall Cling, uses Explosive Rounds for area denial, and benefits from the extra HP of Second Wind to survive return fire. The Anchor excels at locking down zones and forcing enemies into unfavorable engagements.

**Weakness:** Predictability. An Anchor player tends to stay in one position. Perk add-ons (Echo Location, Footstep Reader) reveal their position. Coordinated pushes can overwhelm the area denial. The slow regen from Second Wind means the Anchor is vulnerable between fights if displaced.

---

## 7. Technical Notes

### 7.1 Add-On Effect Priority

When multiple add-ons modify the same stat, effects are applied multiplicatively, not additively. Example: if Kinetic Amplifier (+15%) and a hypothetical future damage buff (+10%) were both active, the result would be `base × 1.15 × 1.10`, not `base × 1.25`. This prevents compounding bonuses from exceeding intended power curves.

### 7.2 Cooldown Behavior

All cooldowns begin at the moment of activation, not at the end of the effect duration. For add-ons with durations, the effective downtime is `cooldown - duration`. Example: Explosive Rounds has a 15-second cooldown and 5-second duration, so the downtime is 10 seconds.

Cooldowns do not pause on death. If a player dies while an add-on is on cooldown, the cooldown continues ticking during the respawn/death screen.

### 7.3 Passive Add-On Stacking

Passive stat modifications from different add-ons stack multiplicatively. However, the same add-on cannot be equipped twice, so no passive stat is modified by more than one source of the same type.

### 7.4 Visual Tell Implementation

All visual tells must be implemented as VFX that are:
- Visible at engagement range (minimum 20 meters)
- Distinct from environment effects and weapon tracers
- Color-coded by category: Damage (red/orange), Movement (blue/cyan), Health (green/yellow), Perk (purple/white)

### 7.5 Audio Tell Implementation

All audio tells must be:
- Audible within the add-on's effective range
- Mixable (players can adjust add-on audio separately from game audio)
- Distinct — no two add-ons share the same activation sound

---

## 8. Balance Tuning Targets

| Metric | Target |
|---|---|
| Win rate deviation by loadout | < 3% from mean |
| Single-add-on pick rate | 15%–35% (no add-on should be in > 35% of loadouts) |
| Category distribution | No category below 20% or above 35% of total picks |
| Average TTK change per damage add-on | ±15% from baseline |
| Loadout diversity | At least 10 distinct loadout archetypes in active use |

Tuning constants for all values referenced in this document are maintained in [Tuning Constants](../06_cross_cutting/tuning_constants.md).

---

## 9. Open Questions

| ID | Question | Status |
|---|---|---|
| OQ-001 | Should add-ons be allowed in competitive/ranked modes with no restrictions, or should a pick/ban phase exist? | Open |
| OQ-002 | Should players be able to save multiple loadout presets for quick switching? | Open |
| OQ-003 | Is the lack of a dedicated "utility" category a problem, or do perk add-ons cover it? | Open |
| OQ-004 | Should add-on visual tells be more prominent at higher skill tiers (SBMM-adjusted readability)? | Open |
| OQ-005 | How many total add-ons should the game launch with? Current catalog: 16. Target: TBD. | Open |

---

## See Also

- [Weapon System](weapon_system.md) — weapon stats that interact with damage add-ons
- [Health & Damage](health_and_damage.md) — base health, regen, and damage formulas
- [Movement System](movement_system.md) — base movement values that movement add-ons modify
- [Tuning Constants](../06_cross_cutting/tuning_constants.md) — all numerical values referenced in this document
