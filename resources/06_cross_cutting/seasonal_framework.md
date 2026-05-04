# Seasonal Framework

## Document Metadata

| Field | Value |
|---|---|
| Document ID | CCD-002 |
| Version | 1.0 |
| Status | Draft |
| Owner | Live Ops Lead |
| Last Updated | 2026-05-05 |

---

## 1. Overview

JuanSchott operates on a seasonal content model. Each season corresponds to one **Cycle** in the game's lore — one complete appearance and departure of the Vimana satellite. The seasonal framework defines the structure, cadence, content delivery, and technical implementation of seasons.

This document covers the seasonal lifecycle from planning through execution. It is the authoritative reference for how content enters the game over time.

---

## 2. Season Structure

### 2.1 Duration

Each season runs for **8-10 weeks** of active content, followed by **1-2 weeks** of off-season. The total cadence is one season every **10-12 weeks** (approximately 4-5 seasons per year).

| Phase | Duration | Description |
|---|---|---|
| Off-season | 1-2 weeks | Transition period between seasons. Ranked queues inactive. Normal play available. |
| Season active | 8-10 weeks | Full seasonal content is live. Ranked queues active. Progression track available. |

### 2.2 Season Arc

Each season follows a four-phase narrative and gameplay arc:

#### Phase 1: Arrival (Week 1)

The Vimana arrives. The new Pantheon host is announced. The season's theme, modifiers, and new content are introduced. Narrative content (codex entries, environmental changes) is deployed. The first week is the most content-dense period of the season.

During Arrival:
- New map or map update goes live.
- New weapon or weapon variant becomes available.
- New add-ons are enabled.
- Seasonal progression track opens.
- Pantheon host announcement is delivered (text and imagery, not full cinematic video for MVP).
- Any season-specific gameplay modifiers are activated.

#### Phase 2: Competition (Weeks 2-7)

The core competitive period. Players engage with ranked matchmaking, climb the seasonal progression track, and experience the season's gameplay modifiers in full effect. This is the longest phase and the primary play period.

During Competition:
- Ranked queues are active with seasonal ELO/MMR tracking.
- Mid-season narrative content may be deployed (a new codex entry, an environmental change to the map, a message from the Pantheon host).
- Balance adjustments may be deployed based on data collection from weeks 1-2.
- No new gameplay content is introduced during this phase. Stability for competitive integrity.

#### Phase 3: Climax (Weeks 8-9)

The season reaches its narrative peak. The Pantheon host's judgment approaches. This phase may include:
- A limited-time game mode variant themed to the Pantheon host.
- Final narrative content drop before judgment.
- Increased XP/progression rewards to encourage seasonal track completion.
- Any final balance adjustments.

#### Phase 4: Judgment (Week 10)

The final week of the season. The Pantheon host delivers their judgment on the season's events. This is delivered as text and imagery (not full video for MVP). The judgment:
- Summarizes the season's competitive outcomes.
- Names top-performing players or factions (if applicable).
- Sets narrative hooks for the next season.
- Closes the seasonal progression track.

After Judgment, the season transitions to off-season.

#### Off-Season (1-2 Weeks)

The Vimana has departed. Ranked queues are disabled. Normal (unranked) play remains available. The off-season serves as:
- A break for players to avoid burnout.
- A technical window for the development team to deploy the next season's content.
- A narrative moment — the Vimana is gone, and the world waits for its return.

During off-season, all seasonal gameplay modifiers are removed. The game reverts to its baseline (Core Canon) state. Maps and weapons from the season remain available — content is not removed when a season ends. Only modifiers and progression tracks are retired.

---

## 3. Pantheon Host System

### 3.1 Concept

Each season features a different **Pantheon member** as the host. The Pantheon is the collective of godlike beings who operate the Vimana. The host for a given season influences:

- The season's theme and aesthetic direction.
- Gameplay modifiers unique to the host's personality/domain.
- Narrative content delivered throughout the season.
- Environmental changes to the active map.

### 3.2 Host Rotation

The five Pantheon members rotate as hosts across seasons. The rotation is not strictly sequential — narrative requirements may dictate which host appears in a given season. However, the intent is to feature each host at least once every 5-6 seasons.

The five Pantheon members (subject to revision in Established Canon):

| Host | Domain | Thematic Influence | Example Modifier |
|---|---|---|---|
| The Architect | Precision, structure, order | Clean geometric map elements, structured layouts | Precision bonus: reduced spread for all weapons when standing still |
| The Flame | Passion, chaos, transformation | Warm lighting, dynamic environmental hazards | Environmental hazards: random fire zones on the map that deal damage |
| The Void | Silence, mystery, the unknown | Dark environments, reduced visibility | Reduced HUD information: health bars hidden, minimap disabled |
| The Weaver | Connection, manipulation, fate | Interconnected map routes, intricate layouts | Bonus movement speed when near teammates |
| The Anvil | Strength, endurance, creation | Industrial environments, heavy machinery | Increased TTK: all players gain +25 HP for the season |

### 3.3 Host Identity Per Season

The Pantheon host for each season is determined during pre-season planning. The host selection drives the season's content plan:

1. Host is selected.
2. Theme is derived from the host's domain.
3. Map content (new map or map update) is designed to match the theme.
4. Gameplay modifier is designed around the host's personality.
5. Narrative content is written from the host's perspective.

---

## 4. Season Content Delivery

### 4.1 Content Per Season

Each season delivers a defined set of content. This is the minimum viable season. Additional content may be added if development capacity allows, but the following is guaranteed:

| Content Type | Quantity Per Season | Notes |
|---|---|---|
| New map OR map update | 1 | Maps are not retired. New maps add to the pool. Map updates modify existing maps. |
| New weapon OR weapon variant | 1 | Weapon variants are existing weapons with modified stats (different trade-offs). |
| New add-ons | 1-2 | New modular equipment pieces that expand player options. |
| Narrative content | Variable | Codex entries, environmental changes, Pantheon host dialogue. Minimum 3 codex entries per season. |
| Seasonal gameplay modifier | 1 | Host-specific modifier active for the full season. |
| Progression track | 1 | Cosmetic-only reward track. See Section 7. |

### 4.2 Content Permanence

Content added in a season **remains in the game permanently** unless specifically designated as seasonal-exclusive. This applies to:

- Maps: remain in the map pool permanently.
- Weapons: remain available permanently.
- Add-ons: remain available permanently.
- Gameplay modifiers: removed at season end.

The only seasonal-exclusive content is:
- The seasonal progression track and its rewards.
- The seasonal gameplay modifier.
- Certain narrative events (specific codex entries may reference the season).

This policy ensures that the game grows over time. Each season expands the game's content without removing previous content. Players who join in Season 3 have access to all maps and weapons from Seasons 1 and 2.

---

## 5. Seasonal Modifiers

### 5.1 Purpose

Seasonal modifiers are gameplay changes introduced by the Pantheon host that affect all matches during the season. They serve two purposes:

1. **Narrative**: The modifier is an expression of the Pantheon host's will. It is a lore event — the host is reshaping the arena to reflect their domain.
2. **Gameplay variety**: The modifier introduces a strategic shift that keeps the game fresh across seasons without permanently changing the balance.

### 5.2 Design Constraints

Seasonal modifiers must:

- Be understandable in one sentence. ("Standing still reduces weapon spread." "Fire zones appear randomly on the map.")
- Not fundamentally break competitive integrity. The modifier must be a layer on top of the core game, not a replacement.
- Be testable. Each modifier must be playtested before the season launches.
- Be reversible. When the season ends, the modifier is removed and the game returns to baseline.
- Not stack with previous season modifiers. Only one modifier is active at a time.

### 5.3 Modifier Impact on Ranked Play

Seasonal modifiers apply to ranked play. This is intentional. Ranked play in JuanSchott is seasonal by nature — your rank is tied to a specific season and its modifier. The modifier is part of the competitive landscape for that season.

However, modifiers that prove destructive to competitive integrity during the season may be adjusted or disabled mid-season for ranked queues only. This decision requires sign-off from the design lead.

---

## 6. Technical Implementation

### 6.1 Feature Gating

Seasonal content is implemented as **Cargo features** in the Rust codebase. Each season is a feature flag that can be enabled or disabled at compile time or runtime.

```
[features]
season_01_architect = []
season_02_flame = []
season_03_void = []
```

This approach allows:

- **Clean separation**: Seasonal code is isolated. It does not interleave with core game code.
- **Testing**: Seasons can be tested independently by enabling only their feature.
- **Rollback**: If a season's content causes issues, it can be disabled without affecting the core game.
- **Archival**: Past seasons can be re-enabled for special events.

### 6.2 Content Loading

Seasonal content (maps, weapons, add-ons) is loaded from asset bundles. The active season's feature flag determines which assets are loaded at startup. Past seasons' assets are loaded on demand if players select maps or weapons from previous seasons.

### 6.3 Modifier Implementation

Seasonal modifiers are implemented as Bevy systems that are conditionally added to the app based on the active season's feature flag. The modifier system runs alongside core game systems and applies its effects through the ECS.

```
if feature_enabled("season_01_architect") {
    app.add_systems(Update, precision_bonus_system);
}
```

### 6.4 Data Model

The season state is stored in a configuration file that specifies:

- Active season ID.
- Active Pantheon host.
- Seasonal modifier parameters.
- Content availability flags (which maps, weapons, add-ons are enabled).
- Progression track definition.
- Phase of the season arc (arrival, competition, climax, judgment, off-season).

This configuration is loaded at startup and can be hot-reloaded for testing.

---

## 7. Player Progression

### 7.1 Seasonal Progression Track

Each season includes a progression track that players advance through by playing matches. The track is **cosmetic-only**. No gameplay-affecting rewards are ever tied to seasonal progression.

Progression track rewards include:

- Player titles (text displayed next to the player's name).
- Weapon skins (visual modifications only — no stat changes).
- Profile portraits.
- Emotes or gestures (visual only, no gameplay effect).
- HUD customization options (visual themes for the UI).

### 7.2 Track Structure

The progression track has **50 tiers**. Each tier requires a fixed amount of XP to unlock. XP is earned from:

- Match completion.
- Kills, assists, and objectives.
- Win bonuses.
- Daily and weekly challenges.

The track is designed to be completable by a player who plays 3-5 matches per day, 4-5 days per week, over the 8-10 week season. Players who play more can complete the track faster. Players who play less can still make meaningful progress.

### 7.3 Track Reset

The progression track resets at the end of each season. Unclaimed rewards are forfeited. This creates urgency and engagement during the active season. However, cosmetic items from completed tiers are kept permanently once claimed.

---

## 8. Narrative Delivery

### 8.1 Delivery Methods

Narrative content is delivered through the following channels:

| Channel | Format | When |
|---|---|---|
| Pantheon host announcement | Text and imagery | Arrival (Week 1) |
| Codex entries | Text (in-game codex) | Arrival and Competition |
| Environmental changes | Visual changes to the active map | Arrival |
| Mid-season event | Text, possible gameplay variant | Competition (Week 4-5) |
| Judgment | Text and imagery | Judgment (Week 10) |

### 8.2 Narrative Tone

Narrative content is written from the perspective of the Pantheon host. Each host has a distinct voice:

- **The Architect**: Formal, precise, measured. Speaks in terms of structure and consequence.
- **The Flame**: Passionate, intense, dramatic. Speaks in terms of passion and transformation.
- **The Void**: Cryptic, minimal, unsettling. Speaks in riddles or near-silence.
- **The Weaver**: Clever, warm, manipulative. Speaks in terms of connection and fate.
- **The Anvil**: Gruff, direct, practical. Speaks in terms of strength and survival.

### 8.3 Narrative Scope Per Season

The narrative for each season is self-contained. It has a beginning (Arrival), middle (Competition, Climax), and end (Judgment). Players who join mid-season can read all released codex entries in the in-game codex. There is no "you had to be there" narrative content.

Season narratives may set hooks for future seasons, but they never end on cliffhangers. Each season's story resolves within that season.

---

## 9. Season Cadence Summary

```
Week 1:     Arrival — Content drop, host announcement, season begins
Weeks 2-3:  Competition — Core play period, data collection
Weeks 4-5:  Competition — Mid-season event, possible balance update
Weeks 6-7:  Competition — Stable competitive period
Weeks 8-9:  Climax — Limited-time content, increased rewards
Week 10:    Judgment — Season concludes, host judgment delivered
Weeks 11-12: Off-season — Ranked disabled, next season prep
```

---

## 10. Season Planning Timeline

The following timeline governs the preparation of each season:

| Milestone | Timing | Description |
|---|---|---|
| Concept approval | 12 weeks before launch | Season host and theme selected. Content plan finalized. |
| Design lock | 8 weeks before launch | All seasonal content (map, weapon, add-ons, modifier) design is finalized. |
| Content complete | 4 weeks before launch | All seasonal content is implemented and in internal testing. |
| Playtest complete | 2 weeks before launch | All seasonal content has been playtested and balanced. |
| Deployment | Launch day | Season goes live. |

---

## 11. Open Questions

| ID | Question | Status |
|---|---|---|
| SQ-1 | Should off-season have any unique gameplay (e.g., all modifiers active simultaneously)? | Deferred |
| SQ-2 | Should seasonal maps ever be retired from the pool? | Under discussion — current policy is no retirement |
| SQ-3 | What happens to a season's modifier if the community strongly dislikes it? | Mid-season disable policy needs formalization |
| SQ-4 | Should past season progression tracks be replayable? | Not in scope for launch |
| SQ-5 | How many past seasons of content should a new player see on first launch? | Needs UX design |
