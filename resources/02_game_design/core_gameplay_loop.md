# Core Gameplay Loop

**Document ID:** GDD-02-001
**Version:** 1.0
**Status:** Draft
**Author:** Design Team
**Last Updated:** 2026-05-05

---

## 1. Overview

This document defines the three interlocking gameplay loops that form the backbone of the player experience in JuanSchott. Every system, map, weapon, and mode exists to serve these loops. If a feature does not reinforce at least one of the three loops, it should be questioned.

The loops operate at different time scales:

| Loop | Timescale | Player Focus |
|------|-----------|--------------|
| **Moment-to-Moment** | 5–15 seconds | Individual combat encounters |
| **Match** | 5–15 minutes | Team score and match outcome |
| **Seasonal** | 6–8 weeks | Ranked progression and narrative |

Each loop feeds into the next. A player who masters the combat loop wins more engagements, which contributes to match victories, which accumulate into seasonal rank progression and story advancement. The loops are designed so that a player can derive satisfaction at any level, in any session length, without requiring commitment to all three simultaneously.

---

## 2. Loop 1 — Moment-to-Moment (Combat Loop)

### 2.1 Definition

The combat loop is the 5–15 second cycle that repeats continuously during active engagement with an opponent. It is the atom of gameplay — the irreducible unit of fun. If this loop does not feel exceptional, nothing else matters.

### 2.2 The Five Phases

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│   OBSERVE ──► DECIDE ──► EXECUTE ──► REACT ──► RESET│
│       ▲                                         │    │
│       └─────────────────────────────────────────┘    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

#### Phase 1: Observe (0.5–2.0 seconds)

The player reads the combat environment and gathers decision-making information. This phase is primarily sensory and spatial:

- **Visual scanning:** Enemy positions, teammate locations, map geometry, available cover, vertical options (ledges, rooftops, scaffolding).
- **Audio cues:** Footsteps (directional, with material-specific sounds), weapon fire reports, ability activation sounds, ambient map audio that may mask quieter movements.
- **Spatial awareness:** Minimap data, kill feed information, recent death locations of teammates indicating enemy presence areas.

The quality of the Observe phase determines the quality of the Decide phase. A player who observes poorly will make suboptimal decisions. Map design must support clear readability — sightlines should be legible, cover should be visually distinct, and vertical routes should be discoverable at a glance without requiring memorization.

#### Phase 2: Decide (0.3–1.0 seconds)

The player selects an approach based on observed information. This is the cognitive core of the loop — the moment where game knowledge, situational awareness, and personal preference converge into a commitment:

- **Engage directly:** Take the gunfight. Commit to a正面 confrontation using aim and movement.
- **Flank:** Disengage from current position and reposition to a superior angle. This may involve parkour routing through side paths or vertical traversal.
- **Retreat:** Fall back to a safer position, bait the enemy into a chase, or regroup with teammates.
- **Reposition vertically:** Use wall-runs, mantles, or grapple add-ons to gain height advantage. In JuanSchott, the high ground is not merely advantageous — it is a fundamentally different combat space with different sightlines and movement options.
- **Activate add-on ability:** Deploy a tactical ability (grapple hook, dash, deployable cover, sensor dart, etc.) to shift the engagement terms.

Decision quality is the primary skill differentiator between player skill tiers. At low tiers, players default to engaging. At high tiers, players choose the option that maximizes their probability of winning the exchange given all known variables.

#### Phase 3: Execute (0.5–3.0 seconds)

The player commits to the chosen action. This phase is primarily mechanical — the translation of decision into input:

- **Aim and fire:** Track or flick to target, manage recoil, control burst timing.
- **Parkour execution:** Wall-run initiation, jump timing, mantle inputs, slide-cancel chaining.
- **Ability activation:** Grapple aim and release, dash direction, deployable placement.
- **Combined actions:** Firing while wall-running. Throwing a projectile while air-strafing. Activating a grapple mid-fall to redirect momentum into a flank.

**Critical design principle: Movement IS combat.** The player never stops moving to shoot. Shooting is something that happens during movement, not instead of it. This is the single most important feel target in the entire game. Titanfall 2 demonstrated that the best moments in a movement shooter happen when the player is simultaneously navigating complex geometry and delivering precise fire. JuanSchott must preserve and extend this principle. A stationary player is a dead player. A moving player who can also aim accurately is dangerous.

#### Phase 4: React (0.2–1.0 seconds)

The player processes the outcome of the execution and responds:

- **Enemy down:** Confirm kill, check for additional threats, decide whether to push or reset.
- **Took damage:** Evaluate health state, determine if immediate retreat is necessary, locate nearest cover.
- **Missed:** Reassess — re-aim, reposition, or disengage based on whether the enemy has now acquired the player's position.
- **Partial damage dealt:** Continue the engagement, adjust strategy, or disengage if the commitment was a mistake.

Reaction speed is the mechanical skill floor. Players who cannot react within this window will consistently lose engagements. However, reaction is subordinate to decision — a player who made a better decision in Phase 2 will have a more favorable reaction window in Phase 4.

#### Phase 5: Reset (2–5 seconds)

A brief disengagement period where the player recovers and prepares for the next Observe phase:

- **Health regeneration:** After 3 seconds without taking damage, health begins regenerating. Full regen takes 2 additional seconds. Total recovery window: 5 seconds from last hit to full health.
- **Repositioning:** Move to a new angle, preferably one the enemy does not expect. Use parkour routes to traverse 20–40 meters in 3–4 seconds.
- **Ammo management:** Reload during movement. Weapon swap if primary is empty.
- **Ability cooldown check:** Verify add-on availability for the next engagement.

The Reset phase is not dead time. It is the setup for the next Observe phase. A player who resets well — choosing unexpected repositioning routes, timing their regen, managing resources — enters the next Observe phase with a significant advantage over a player who simply sprints back to the fight.

### 2.3 Key Feel Targets

| Metric | Target Value | Design Rationale |
|--------|-------------|------------------|
| **Time to Kill (clean)** | 0.3–0.8 seconds | Rewards precision. A headshot burst from a competent player should be lethal. Prevents prolonged chip-damage exchanges that slow the game. |
| **Time to Kill (body shots)** | 1.0–1.5 seconds | Allows reaction window for the target. Body-shot-only fights should be survivable if the target repositions quickly. |
| **Time to Reposition** | 2–4 seconds for a new angle via parkour | Ensures the battlefield is constantly shifting. Players cannot hold a single angle indefinitely because opponents can flank quickly. |
| **Time to Regenerate** | 3s delay + 2s regen = 5s total | Long enough that retreating is a meaningful cost. Short enough that players are not removed from action for frustrating durations. |
| **Average Engagement Distance (hitscan)** | 15–30 meters | Close enough that movement and positioning matter. Far enough that aim is a distinct skill. |
| **Average Engagement Distance (projectile)** | 30–60 meters | Projectile weapons must be effective at range where travel time creates meaningful prediction requirements. |
| **Average Combat Loop Duration** | 8–12 seconds | Fits within the 5–15 second target. Fast enough to feel frenetic, slow enough to be legible. |

### 2.4 Feel References

The combat loop should feel like Titanfall 2's pilot combat distilled and refined: the fluid transition from wall-run to aim to kill to continued movement, without any stutter or pause. The specific reference points are:

- **Titanfall 2 — The Effect and Cause level:** The segment where the player fights through a facility while time-shifting. The pace of combat — observe, snap to target, eliminate, immediately resume movement — is the target.
- **Apex Legends — Early game (Season 0–3):** The pacing of a clean 1v1 where both players are moving, using cover, and the fight resolves in under 10 seconds.
- **Quake 3 / CPMA:** The instantaneous transition between movement and combat. There is no "combat mode" and "movement mode" — they are the same mode.

What JuanSchott must avoid:
- **Call of Duty-style ADS lock:** The player should never feel that stopping increases their effectiveness. Movement should not penalize accuracy beyond a manageable spread increase.
- **Overwatch-style ability reliance:** Abilities supplement the gunfight; they do not replace it. A player without ability charges should still be fully competitive.
- **PUBG-style long-range camping:** Map geometry and movement systems should actively punish static play. Every position should be flankable within 4 seconds.

---

## 3. Loop 2 — Match Loop (5–15 Minutes)

### 3.1 Definition

The match loop is the structure that contains individual combat encounters within a single game session. It provides stakes, scoring, and pacing for the combat loop. Without this loop, combat encounters would be meaningless frags. With it, each kill contributes to a team objective, and each death carries a cost beyond personal embarrassment.

### 3.2 Match Lifecycle

```
SPAWN ──► NAVIGATE ──► ENGAGE ──► OUTCOME ──► RESPAWN/RESET ──► ┐
   ▲                                                              │
   └──────────────────────────────────────────────────────────────┘
                                                                  │
RESULT (Win/Loss) ◄───────────────────────────────────────────────┘
```

#### Stage 1: Spawn

The player selects or confirms their loadout: primary weapon, secondary weapon (if applicable), add-on ability, and any cosmetic items. Spawn occurs at the team spawn zone — a designated area of the map that provides safe cover and immediate access to multiple parkour routes toward the center of the map.

Spawn protection lasts 1.5 seconds. During this window, the spawning player cannot deal or receive damage. This prevents spawn-instant-kill scenarios without allowing players to exploit invulnerability offensively.

#### Stage 2: Navigate

The player moves from spawn toward the engagement area. This is not dead travel time — it is route selection. Maps are designed with multiple paths of varying risk and speed:

- **Direct route:** Fastest path, highest exposure. Good for aggressive players who want immediate contact.
- **Flank route:** Longer path with cover, emerges at an unexpected angle. Good for players who prefer to set up ambushes.
- **Vertical route:** Parkour-intensive path that gains height. Requires movement skill but provides superior positioning.

Route choice is the first strategic decision of each life. At high-level play, teams coordinate routes to ensure coverage of multiple approach angles.

#### Stage 3: Engage

The player encounters an enemy. The combat loop (Loop 1) activates. This stage is covered in detail in Section 2.

#### Stage 4: Outcome

The engagement resolves. Three possible results:

| Outcome | Effect |
|---------|--------|
| **Kill** | Team score increments. Player gains temporary momentum advantage (position, information). |
| **Death** | Player enters respawn queue (3–5 second delay). Team loses map presence. |
| **Disengage** | Both players survive. No score change. Positioning resets. |

#### Stage 5: Respawn or Reset

- **If killed:** Player enters a 3–5 second respawn queue. During this time, the player can observe the battlefield from a free camera, plan their next route, and adjust loadout if the game mode permits mid-match changes. Spawn location is randomized within the team spawn zone to prevent spawn camping.
- **If alive:** Player enters the Reset phase of the combat loop. Reposition, regenerate, and prepare for the next engagement.

#### Stage 6: Repeat

The cycle continues until the match end condition is met.

#### Stage 7: Result

The match concludes. The end screen displays:

- Team scores, individual stats (kills, deaths, assists, damage dealt, accuracy, distance traveled)
- MVP highlight (highest-impact player, not just highest kills)
- XP gain (cosmetic progression only — no gameplay-affecting unlocks)
- Rank change (ranked modes only)

### 3.3 Match Structure

**Format:** Score-based team deathmatch.

- **Win condition:** First team to reach the kill target, or highest kills when the time limit expires.
- **Kill target:** Varies by mode (see game_modes.md for specifics). Default: 50 kills for 6v6, 30 kills for 3v3.
- **Time limit:** 10 minutes (6v6), 8 minutes (3v3).
- **No rounds:** Play is continuous. There are no halftime breaks or round resets. This maintains momentum and prevents the stop-start pacing that kills engagement in round-based shooters.
- **Respawn system:** Spawn zones are randomized areas (not fixed points). The spawn algorithm prioritizes locations that are:
  1. Outside enemy line-of-sight
  2. Within 5 seconds of the nearest teammate
  3. At least 10 meters from any enemy
  If no location satisfies all three, priority drops to (1) and (3) only. Spawn camping is further discouraged by ensuring no map position has line-of-sight to the entire spawn zone.

**HUD Elements (always visible during match):**

- Team scores (top center)
- Match timer (top center)
- Player health bar (bottom center)
- Ammo count (bottom right)
- Add-on ability status (bottom right)
- Minimap (top left, shows teammates and detected enemies)
- Kill feed (top right, shows recent kills across both teams)
- Teammate status indicators (health and position via minimap)

### 3.4 Pacing Targets

A well-paced match should feel like a rising wave. The first 2 minutes are positioning and probing — players are learning enemy routes and establishing control. Minutes 2–6 are the core combat phase — constant engagements, shifting map control, and score movement. Minutes 6+ (if the game is close) are the climax — every kill matters, every death is costly, and the tension is palpable.

The match loop should never feel like a grind. If the score differential exceeds 60% (e.g., 30-18 in a 50-kill match), a catch-up mechanic subtly adjusts spawn positioning to give the trailing team slightly more favorable routes to the center of the map. This does not change gun balance or health — it only affects the travel time to engagement, keeping matches competitive without artificial rubber-banding.

---

## 4. Loop 3 — Seasonal Loop (6–8 Weeks)

### 4.1 Definition

The seasonal loop is the meta-structure that provides long-term motivation, narrative progression, and content freshness. Individual matches are meaningful because they contribute to seasonal rank and story. Without this loop, the match loop would become repetitive within days. With it, players have a reason to return for weeks.

### 4.2 Season Lifecycle

#### Phase 1: Season Start (Week 1)

A new Pantheon host is announced — a mythological figure who serves as the narrator and thematic anchor for the season. The host introduces the season's theme through a cinematic: new visual motifs, arena modifications, and any narrative stakes for Earth's status.

New content is introduced:

- **1 new map** (or a significantly reworked existing map — new routes, geometry changes, or environmental modifiers)
- **1 new weapon** (or a significant weapon variant that changes playstyle)
- **0–1 new add-on abilities** (new abilities are rare and require extensive playtesting)

All new content is available to all players immediately. There are no gameplay unlocks. Progression is cosmetic only.

#### Phase 2: Play Matches (Weeks 1–6)

Players compete in three modes:

| Mode | Team Size | Ranked | Purpose |
|------|-----------|--------|---------|
| **Ranked** | 3v3 | Yes | Competitive play. Rank progression. |
| **Casual** | 6v6 | No | Relaxed play. Experimentation. Social play. |
| **Practice** | 1v1 | No | Warm-up. Movement training. Duel skill development. |

Ranked mode uses an Elo-based system with visible rank tiers (Bronze → Silver → Gold → Platinum → Diamond → Champion). Players are matched against opponents of similar rank. Rank resets partially between seasons (soft reset: players drop approximately one tier).

#### Phase 3: Adapt Meta (Weeks 2–4)

The community discovers optimal strategies for new content. Initial assumptions about weapon balance and map control are tested. Counter-strategies emerge. The meta stabilizes by Week 4.

The design team monitors data during this phase but does not intervene unless a strategy is clearly degenerate (defined as: a single strategy with no viable counter that dominates high-level play). Minor imbalances are acceptable and expected — they create meta diversity as players adapt.

#### Phase 4: Season Event (Week 4–5)

A mid-season narrative event introduces a special game mode modifier for a limited time (7–10 days). Examples:

- **Low gravity:** Jump height doubled, fall speed reduced. Parkour routes change fundamentally.
- **Double add-on charges:** Abilities can be used twice before cooldown. Combat pacing shifts dramatically.
- **Fog of war:** Minimap disabled. Audio cues become critical. Slower, more tactical play.

The event includes a lore drop: a narrative beat that advances the Pantheon storyline and provides context for the season's events. This is delivered through a 2–3 minute cinematic accessible from the main menu.

#### Phase 5: Season Climax (Week 6)

The final week of ranked play. XP rewards are increased by 50% to encourage participation. The narrative reaches its peak tension — the Pantheon host's judgment of Earth's combatants approaches.

Ranked play becomes more intense as players make their final push for rank milestones. Match quality may slightly decrease as player volume increases, but the competitive atmosphere compensates.

#### Phase 6: Judgment (End of Week 6 or 7)

The season concludes with a cinematic: the Pantheon host delivers their verdict on Earth's champions. The narrative outcome is determined by aggregate community performance (total kills, total matches played, ranked distribution) — not by any individual player's actions. This creates a sense of collective achievement or failure that motivates the next season.

End-of-season rewards are distributed:

- Rank-based cosmetic items (titles, weapon skins, character flair)
- Season-specific cosmetic sets (themed to the Pantheon host)
- Completion badges for players who participated in the season event

#### Phase 7: Off-Season (1–2 Weeks)

Brief downtime between seasons. Ranked mode is disabled. Casual and practice modes remain available.

During off-season, the design team deploys balance adjustments based on seasonal data. Patch notes are published with detailed reasoning. Players can review their seasonal stats, adjust loadouts, and prepare for the next season's content.

The off-season also serves as a psychological reset — a moment to decompress before the next competitive push.

---

## 5. How the Loops Connect

The three loops are not independent systems. They are a causal chain where mastery at each level enables success at the next:

```
Combat Loop Mastery ──► Wins Engagements ──► Match Loop Success ──► Rank Progression ──► Seasonal Loop Advancement
```

**Combat → Match:** A player who consistently wins 1v1 engagements contributes directly to their team's score. At the match level, individual skill is the foundation. However, match-level strategy (coordinated routes, target prioritization, map control) amplifies individual skill. A team of individually skilled players who do not coordinate will lose to a team of slightly less skilled players who do.

**Match → Season:** Winning matches increases rank. Rank is the primary seasonal progression metric. Players who maintain a high win rate will climb; players who do not will stabilize at their skill-appropriate tier. The match loop's continuous format (no rounds, respawns) ensures that individual matches are representative of skill rather than being decided by a single mistake.

**Season → Motivation:** Rank tiers, narrative progression, and seasonal events provide the long-term hook. A player who has reached Diamond rank has a tangible reason to maintain it. A player who has followed the Pantheon storyline wants to see the next chapter. These motivations sustain engagement across weeks of play.

**Feedback downward:** The seasonal loop also feeds back into the combat loop by introducing new content. A new weapon changes engagement dynamics. A new map changes routing and positioning. This prevents the combat loop from becoming stale and ensures that mastery is an ongoing process, not a destination.

---

## 6. Player Motivation at Each Level

| Loop | Core Motivation | Emotional Reward |
|------|----------------|------------------|
| **Combat** | "I want to outplay my opponent through positioning and aim" | Flow state, mastery satisfaction, clutch adrenaline |
| **Match** | "I want to win this match for my team" | Team coordination, competitive achievement, social bonding |
| **Season** | "I want to climb the ranked ladder and see the story unfold" | Long-term progression, narrative investment, community belonging |

Each motivation is self-sustaining. A player who only cares about the combat loop can play casual 6v6 indefinitely and derive full satisfaction. A player who only cares about rank can focus exclusively on 3v3 ranked. A player who is invested in the narrative will participate in seasonal events and follow the story. The loops serve different player types without requiring commitment to all three.

---

## 7. Session Structure

The loop design supports multiple session lengths. A player should never feel that a short session is "wasted" or that a long session becomes a grind.

### 7.1 The 5-Minute Session

**Activity:** 1–2 practice 1v1 matches.
**Experience:** Warm up aim against a live opponent. Practice a specific movement tech (wall-run to mantle, slide-cancel timing). Test a new weapon's feel. Leave.
**Satisfaction source:** Combat loop only. The player practiced a skill and felt improvement, even marginal.

### 7.2 The 30-Minute Session

**Activity:** 3–4 casual 6v6 matches.
**Experience:** Quick-load into casual queue (target: <30 second queue time). Play through 3–4 complete matches. Try a new loadout combination. Experiment with different routes on the current map rotation.
**Satisfaction source:** Combat loop + match loop. The player experienced several complete match arcs, won some, lost some, and had fun throughout.

### 7.3 The 2-Hour Session

**Activity:** Warm-up (10 min) → Ranked 3v3 (90 min) → Cool-down (20 min).
**Experience:** 1v1 practice warm-up to calibrate aim and movement. Transition to ranked queue for focused competitive play. 90 minutes of ranked yields approximately 8–12 matches. End with a casual match or practice session to decompress. Review loadout performance and adjust for next session.
**Satisfaction source:** All three loops. The player engaged deeply with the combat loop, progressed (or defended) their rank in the match loop, and contributed to their seasonal standing.

### 7.4 Session Design Principles

- **No mandatory session length.** A player should never be penalized for leaving after one match. No "first win of the day" bonuses that punish short sessions.
- **Queue time target:** Under 30 seconds for all modes at all skill levels. This is a hard requirement, not an aspiration. If queue times exceed 30 seconds, the player population is insufficient and the mode/matchmaking requires adjustment.
- **Match duration target:** 8–12 minutes. Short enough to feel like a discrete unit. Long enough to develop narrative tension within the match.

---

## 8. Design Guardrails

The following rules must be maintained across all content and balance decisions to preserve the integrity of the core gameplay loops:

1. **Movement is never penalized.** No mechanic should make a stationary player more effective than a moving one. Accuracy penalties while moving must be minimal and recoverable.
2. **No gameplay-affecting unlocks.** All weapons, add-ons, and abilities are available from the start. Progression is cosmetic only. A day-one player has the same combat capability as a three-year veteran.
3. **No random mechanics in combat.** No critical hit chances, no recoil randomization (recoil patterns are fixed and learnable), no spread RNG within intended accuracy parameters. Combat outcomes must be deterministic based on player input.
4. **Engagement time is bounded.** No mechanic should allow a single engagement to last longer than 5 seconds without a resolution (kill, disengage, or reset). Prolonged chip-damage fights violate the combat loop timing.
5. **Spawn systems must be fair.** No player should be killed within 2 seconds of spawning. Spawn zones must be dynamic and randomized.
6. **Seasonal content must not be power-crept.** New weapons and abilities must be side-grades, not upgrades. The Season 1 weapon roster must remain viable in Season 10.

---

## 9. See Also

| Document | Path | Relevance |
|----------|------|-----------|
| Player Experience | `player_experience.md` | Emotional targets and player personas |
| Movement System | `movement_system.md` | Parkour mechanics that enable Loop 1 |
| Weapon System | `weapon_system.md` | Gunplay mechanics and TTK calibration |
| Game Modes | `game_modes.md` | Mode-specific rules and scoring |
| Seasonal Framework | `../06_cross_cutting/seasonal_framework.md` | Narrative structure and season planning |
