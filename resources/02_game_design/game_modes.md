# Game Modes [SYS-005]

**Document ID:** SYS-005
**Category:** Core Systems
**Status:** Design Complete
**Last Updated:** 2026-05-05
**Dependencies:** SYS-001 (Core Gameplay Loop), SYS-004 (Health and Damage), SYS-008 (Map Design Philosophy)

---

## 1. Design Philosophy

Game modes are the structural containers that give combat context. Every mode in this document exists to serve a specific player intent — from deliberate practice to coordinated competition to chaotic relaxation. The decision to launch with exactly three modes is intentional: each targets a distinct player count, intensity band, and commitment level, and together they cover the full spectrum of competitive shooter engagement without fragmenting the player base.

Three principles govern all mode design:

1. **One mode, one job.** Duel is for skill expression. Skirmish is for competitive ranked play. Clash is for casual fun. Modes that try to be everything serve no one.
2. **Shared foundation, divergent parameters.** All modes run on the same combat system, the same weapons, the same movement. Modes differ in scale, pace, and stakes — never in core mechanics. A player who learns to fight in Duel applies those skills directly to Skirmish and Clash.
3. **Population-aware.** Three modes means three queues. Splitting further at launch risks wait times and matchmaking quality. Future modes are deferred until the player base supports them.

---

## 2. Mode 1 — Duel [MODE-001]

### 2.1 Identity

| Parameter | Value |
|---|---|
| Players | 1v1 |
| Purpose | Practice, aim training, skill expression |
| Format | First to 10 kills wins |
| Time limit | 15 minutes (hard cap) |
| Respawn delay | 3 seconds |
| Ranked | Yes — separate MMR ladder |

Duel is the laboratory. It strips away team dynamics, coordination, and chaos, leaving nothing but two players and their mechanical skill. It serves three populations: new players learning gunfeel, experienced players warming up before ranked, and dedicated duelists who treat 1v1 as its own competitive discipline.

### 2.2 Win Condition

The first player to reach **10 kills** wins the match. No time limit applies during normal play — the match ends when someone hits 10.

**Hard cap:** If 15 minutes of match time elapse and neither player has reached 10 kills, the match proceeds to sudden death (see §2.5).

### 2.3 Scoring

- **1 point per kill.** No assists — assists are meaningless in a 1v1 context.
- Score is displayed prominently at all times via the HUD. Both players always know the exact state of the match.

### 2.4 Map Requirements

Duel maps are **small arenas** designed for intimate combat:

- **Footprint:** Roughly 25–35% the size of a Skirmish map.
- **Sight lines:** Tight and deliberate. Long sight lines exist but are limited to one or two per map, creating known risk zones.
- **Parkour routes:** Present but limited. Vertical options reward movement skill without allowing players to disengage entirely. A duel should not become a chase.
- **Cover density:** High. Players should be able to break line of sight within 1–2 seconds of movement from any position.
- **Symmetry:** All duel maps are axially or rotationally symmetric. No side advantage.

Map design specifics are documented in `map_design_philosophy.md`.

### 2.5 Overtime and Sudden Death

**Match point rule:** If both players reach 9 kills, the next kill wins regardless of the score gap (the "match point" state). This is announced via HUD and audio callout.

**Stalemate breaker:** If 5 minutes of match point pass without a kill — indicating both players are playing excessively passive — a shrinking zone activates:

- A visible circle begins contracting from the map edges toward the center.
- Players outside the zone take 10 damage per second.
- The zone shrinks over 30 seconds to a final radius that forces face-to-face combat.
- The zone is visible on the minimap and as a physical boundary effect in the world.

### 2.6 Ranked Play

Duel maintains its own **independent MMR and ranking ladder**, separate from the 3v3 ranked system. Rationale:

- 1v1 skill does not directly translate to team performance. A mechanically gifted player with poor team coordination could dominate Duel while struggling in Skirmish.
- Separate ladders allow players to compete meaningfully in both without one polluting the other's matchmaking.
- Duel rankings use the same seasonal structure as Skirmish (see §3.5) but with adjusted percentile thresholds reflecting the smaller ranked population.

### 2.7 Special Rules

**Loadout visibility:** Both players can see each other's full loadout (primary weapon, secondary weapon, add-ons) during a 10-second pre-match phase. Counter-picking is explicit and intentional in 1v1 — players may swap loadouts during this window. Once the match begins, loadouts are locked.

This rule exists because 1v1 is a format where loadout advantage is magnified. Transparency removes hidden advantage and makes pre-match preparation part of the skill expression.

---

## 3. Mode 2 — Skirmish [MODE-002]

### 3.1 Identity

| Parameter | Value |
|---|---|
| Players | 3v3 |
| Purpose | Ranked, hardcore, competitive |
| Format | Team deathmatch |
| Kill target | 30 kills |
| Time limit | 10 minutes |
| Respawn delay | 4 seconds |
| Ranked | Yes — primary ranked mode |

Skirmish is the flagship mode. It is the mode the game is balanced around, the mode esports structures are built on, and the mode that defines the game's competitive identity. Every design decision in Skirmish prioritizes competitive integrity and depth.

### 3.2 Win Condition

The first team to reach **30 collective kills** wins the match. If the 10-minute time limit is reached before either team hits 30, the team with the higher score wins.

If scores are tied at time limit, overtime rules apply (see §5.3).

### 3.3 Scoring

| Event | Points |
|---|---|
| Kill | 1.0 |
| Assist | 0.5 |

**Assist qualification:** Dealing 40 or more damage to an enemy who dies within 3 seconds of the last damage instance qualifies as an assist. This threshold is documented in `health_and_damage.md`.

**Team score** is the sum of all individual player scores, displayed as a single integer (rounded down). Individual contributions are visible on the scoreboard but do not affect the match outcome independently.

### 3.4 Map Requirements

Skirmish maps are **medium arenas** designed for team coordination:

- **Footprint:** Baseline reference size. All other map sizes are defined relative to Skirmish.
- **Routes:** Multiple interconnected paths between any two points. No single choke should be the only option. Minimum three routes between major map zones.
- **Verticality:** Deliberate but not dominant. Elevated positions provide sight line advantage but have limited cover, creating risk-reward tradeoffs.
- **Sight lines:** A mix of long (25–40m), medium (10–25m), and close (<10m) engagement distances. No single range should dominate the map.
- **Team identity:** Asymmetric by design — sides are not mirrored, but statistical fairness is ensured through halftime side swaps (see §5.2).

### 3.5 Ranked System

Skirmish is the **primary ranked mode** and uses a full MMR/ELO system:

- **Placement:** New players complete 10 placement matches to establish initial rank.
- **Progression:** Wins increase MMR; losses decrease it. Performance-based adjustments (kill contribution, match impact) modulate the magnitude of change within a bounded range.
- **Seasons:** Rankings reset at the start of each season (6–8 week cycle). Previous season rank determines starting MMR for placement matches in the new season.
- **Rank tiers:** Specific tier names and thresholds are defined in `player_experience.md`.
- **Queue integrity:** Parties of 3 face parties of 3. Solo players are matched against solo players or small groups within tight MMR bands.

### 3.6 Team Composition

**No restrictions.** Any combination of loadouts, weapons, and add-ons is permitted. Three players running identical loadouts is valid. Three snipers is valid. The emergent strategy is the player's responsibility, not the system's.

Rationale: Restricting composition adds complexity without depth in a game without defined roles. Players discover optimal compositions organically through play, and the meta evolves naturally without designer intervention.

### 3.7 Special Rules

**Death camera:** After dying, the player sees a 2-second overhead camera showing teammate positions before entering respawn. This provides tactical context — the respawning player can see where the fight is and make informed decisions about re-engagement angles.

**Kill streak announcements:** Streaks of 3, 5, and 7 kills without dying trigger server-wide announcements (audio callout + kill feed highlight). These are **cosmetic only** — no gameplay bonuses, no power rewards, no score multipliers. Kill streaks are recognition, not reward.

---

## 4. Mode 3 — Clash [MODE-003]

### 4.1 Identity

| Parameter | Value |
|---|---|
| Players | 6v6 |
| Purpose | Casual, unranked, chaotic fun |
| Format | Team deathmatch |
| Kill target | 75 kills |
| Time limit | 12 minutes |
| Respawn delay | 5 seconds |
| Ranked | No — casual queue only |

Clash is the relaxed entry point. It accommodates players who want the feel of the game without the pressure of ranked competition, as well as groups of friends with mixed skill levels who want to play together. The design prioritizes constant action and minimal friction over competitive fairness.

### 4.2 Win Condition

The first team to reach **75 collective kills** wins. If the 12-minute time limit is reached, the team with the higher score wins.

The higher kill target and longer time limit reflect the larger player count — individual deaths matter less, and the match has room for momentum swings.

### 4.3 Scoring

Identical to Skirmish: 1 point per kill, 0.5 per assist. Same assist qualification rules.

### 4.4 Map Requirements

Clash maps are **large arenas** designed for multi-front chaos:

- **Footprint:** Roughly 200–250% the size of a Skirmish map.
- **Routes:** Extensive. Multiple combat zones can be active simultaneously without players interfering with each other. Players should feel like they can choose where to fight.
- **Verticality:** More pronounced than Skirmish. Multi-level structures, rooftops, catwalks, and elevated platforms create a three-dimensional battlespace.
- **Hot zones:** Each Clash map designates 1–2 **hot zones** — areas with intentional design features (tight cover, central position, high traffic routing) that concentrate combat. Hot zones are marked on the minimap with a pulsing indicator to guide newer players toward action.

### 4.5 Matchmaking

Clash uses **loose skill-band matchmaking** — players are grouped into broad skill tiers rather than matched by precise MMR. This prioritizes fast queue times over tight competitive balance:

- **Queue timeout:** If a match is not found within 60 seconds, the skill band widens progressively.
- **Group size:** Parties of up to 6 are allowed. Mixed-skill groups are matched against similar mixed-skill groups when possible.
- **No rank impact:** Clash results do not affect MMR or any ranked metric.

### 4.6 Special Rules

**Auto-balance:** If a team loses enough players (disconnect, quit) that the disparity exceeds 2 players, auto-balance triggers between spawn waves:

- The system identifies the lowest-contributing player on the larger team (by score) and offers a voluntary switch. If no volunteer within 10 seconds, the system assigns the switch.
- Switched players retain their score contribution for their original team but future contributions count for the new team.
- Auto-balance is announced via HUD notification.

**Hot zone markers:** Designated hot zone areas pulse on the minimap and have a subtle world-space indicator (light column or environmental marker) visible from across the map. This serves as a navigational aid for players who are unsure where to go.

---

## 5. Shared Rules Across All Modes

### 5.1 Friendly Fire

**Friendly fire is OFF** in all modes. Players cannot damage or impede teammates with any weapon, ability, or environmental interaction.

Rationale: Friendly fire creates griefing vectors in casual modes and adds cognitive overhead that conflicts with the readability pillar. The tactical depth of friendly fire management does not justify the cost in a game focused on movement and gunskill.

### 5.2 Spawn System

All modes use a **zone-based randomized spawn system**:

- Each map has **fixed spawn zones** for each team (typically 2–3 zones per side).
- Within a spawn zone, the exact spawn point is **randomized** from a set of validated positions. This prevents spawn camping at specific points.
- Spawn zones **swap sides at halftime** — the midpoint of the time limit (5 minutes in Skirmish, 6 minutes in Clash). This ensures map asymmetry does not create lasting advantage.
- Spawn protection: 1 second of invulnerability after spawning, removed instantly upon firing a weapon. This prevents spawn killing without allowing offensive use of spawn protection.

### 5.3 Overtime

If scores are tied at the end of regulation time:

1. **60-second sudden death** begins. The first kill (or first team to gain a lead) wins.
2. If the 60-second sudden death ends with no score change, the match is declared a **draw**.
3. Draw handling in ranked: each team gains/loses 50% of the MMR they would have gained/lost for a win/loss. This prevents draws from being meaningless while ensuring they are always worse than winning.

### 5.4 Respawn Delays by Mode

| Mode | Respawn Delay | Rationale |
|---|---|---|
| Duel | 3 seconds | Quick return keeps 1v1 pace high. Down time is minimal. |
| Skirmish | 4 seconds | Slightly longer to create meaningful man-advantage windows for the opposing team. |
| Clash | 5 seconds | Longest delay to allow 6v6 fights to resolve before bodies return. |

The respawn delay includes the death camera (where applicable) and any pre-spawn UI. The player is actionable the instant the delay completes.

### 5.5 Matchmaking Architecture

- **MMR separation:** Duel MMR and Skirmish MMR are fully independent. Playing one mode does not affect the other's matchmaking.
- **Clash MMR:** Tracked internally for loose matchmaking but never displayed to players. No visible rank.
- **Queue priority:** Skirmish (ranked) receives priority in server allocation during peak hours to ensure competitive integrity.

---

## 6. Mode Comparison Matrix

| Property | Duel | Skirmish | Clash |
|---|---|---|---|
| Players | 1v1 | 3v3 | 6v6 |
| Kill target | 10 | 30 | 75 |
| Time limit | 15 min | 10 min | 12 min |
| Respawn | 3s | 4s | 5s |
| Assists | No | Yes (0.5) | Yes (0.5) |
| Ranked | Separate ladder | Primary ranked | Unranked |
| Map size | Small | Medium | Large |
| Friendly fire | OFF | OFF | OFF |
| Loadout visibility | Full pre-match | Hidden | Hidden |
| Auto-balance | N/A | No | Yes |
| Stalemate breaker | Shrinking zone | Overtime only | Overtime only |

---

## 7. Future Modes (Post-Launch Scope)

These modes are **not in scope for launch** and are documented here for planning purposes only. Development begins when population metrics indicate the player base can sustain additional queues without degrading match quality in existing modes.

| Mode | Players | Description |
|---|---|---|
| Capture Point Control | 3v3 / 6v6 | Static objectives. Teams fight over fixed capture zones. Introduces positional strategy beyond deathmatch. |
| Payload Escort | 6v6 | Asymmetric attack/defend. One team escorts a moving payload along a fixed path; the other team prevents progress. |
| Free-for-All Deathmatch | 8 players | Every player for themselves. No teams. Pure individual competition in a chaotic environment. |
| Custom Matches | Variable | Player-created matches with adjustable rules (score limits, time limits, weapon restrictions, gravity modifiers). Intended for community events, tournaments, and content creators. |

Each future mode will receive its own design document when approved for development.

---

## 8. Appendix — Parameter Quick Reference

### Respawn Timing

```
Duel:     ██████ (3s)
Skirmish: ████████ (4s)
Clash:    ██████████ (5s)
```

### Match Duration Target

```
Duel:     ~5–10 minutes (variable, 15 min hard cap)
Skirmish: ~8–10 minutes (10 min hard cap)
Clash:    ~10–12 minutes (12 min hard cap)
```

### Kill Target Scaled by Player Count

```
Duel:     10 kills  × 1 player = 10 kills per player (avg)
Skirmish: 30 kills  ÷ 3 players = 10 kills per player (avg)
Clash:    75 kills  ÷ 6 players = 12.5 kills per player (avg)
```

The per-player kill target is calibrated to keep average match durations within the target range. If actual match data deviates significantly, kill targets are the first tuning lever, followed by respawn delays.

---

## 9. See Also

- `core_gameplay_loop.md` — Combat loop and moment-to-moment design
- `health_and_damage.md` — HP, damage, regeneration, assist qualification
- `map_design_philosophy.md` — Arena design principles, size standards, spawn zone placement
- `player_experience.md` — Ranked progression, seasonal structure, MMR system
