# Decisions Log

> Every major decision in JuanSchott's design and architecture is recorded here with context, alternatives considered, rationale, and date. This document is the institutional memory of the project.

## Format

Each entry follows the Architecture Decision Record (ADR) pattern:

```
### [ID] Decision Title
- **Date:** When the decision was made
- **Status:** Accepted / Superseded / Deprecated
- **Context:** Why this decision was needed
- **Alternatives Considered:** What options were evaluated
- **Decision:** What was chosen
- **Rationale:** Why this option was chosen over others
- **Consequences:** What this choice enables and constrains
```

---

### D-001: Pure Software Rendering Approach

- **Date:** Initial research phase
- **Status:** Superseded by D-002
- **Context:** The project began with the goal of running an FPS on any hardware, including machines without dedicated GPUs, by rendering entirely on the CPU using SIMD/AVX2 compute.
- **Alternatives Considered:**
  - olive.c as a rasterizer base
  - Custom SIMD renderer from scratch
  - Doom-style span-based rendering
  - Quake-style BSP + edge-list rasterization
- **Decision:** Pursue software rendering using compute shaders with SIMD optimization
- **Rationale:** The math showed 1080p@120fps was theoretically achievable with AVX2 on modern CPUs. Doom and Quake proved the approach worked at lower resolutions.
- **Consequences:** Research validated the compute budgets but revealed a hard visual ceiling equivalent to Quake 3 / CS 1.6. This was insufficient for the project's visual ambitions.

---

### D-002: Abandon Software Rendering, Evaluate Engines

- **Date:** After software rendering research concluded
- **Status:** Accepted
- **Context:** Software rendering's visual ceiling was too low. The core values (optimization, old hardware support, open-source) remained, but the delivery method needed to change.
- **Alternatives Considered:**
  - Continue with software rendering (lower visual quality)
  - Godot 4 (110K stars, MIT, OpenGL Compatibility mode)
  - Bevy (45.9K stars, MIT/Apache, pre-1.0 Rust ECS)
  - O3DE (Apache 2.0, AAA lineage, AWS-dependent)
  - Flax Engine (BSD-3, C#, capable but small)
  - Custom wgpu-based engine (maximum control, maximum work)
- **Decision:** Evaluate Godot 4 and Bevy as final candidates
- **Rationale:** O3DE was too cumbersome with AWS dependencies. Flax was proprietary-company-controlled. Custom wgpu was reinventing Bevy. The real choice was between Godot's stability and Bevy's architecture.
- **Consequences:** Led to deep evaluation of both engines.

---

### D-003: Choose Bevy Over Godot

- **Date:** After engine evaluation
- **Status:** Accepted
- **Context:** Both Godot 4 and Bevy 0.18 were viable. Godot offered stability, a full editor, and a massive community. Bevy offered native ECS, Rust safety, automatic parallelism, and superior architecture for FPS games — but was pre-1.0 with breaking changes every 3-5 months.
- **Alternatives Considered:**
  - Godot 4 (stable, proven, massive community, OpenGL compat for old hardware)
  - Bevy 0.18 (pre-1.0, native ECS, Rust, better architecture)
- **Decision:** Bevy 0.18
- **Rationale:** The project philosophy is open-source growth. Breaking changes become community migration PRs, not private suffering. The ECS architecture is fundamentally better for FPS games — cache-friendly data access, automatic system parallelism, clean entity-component composition. Rust's memory safety prevents entire classes of bugs in netcode. The project grows with Bevy.
- **Consequences:**
  - Must pin to specific Bevy versions and migrate on schedule
  - No visual editor — maps built in Blender, exported as glTF
  - Smaller contributor pool (Rust developers) but more engaged
  - Compile times are slower than Godot's GDScript iteration
  - Architecture is cleaner and more maintainable long-term

---

### D-004: Avian 3D for Physics

- **Date:** Engine selection phase
- **Status:** Accepted
- **Context:** FPS movement requires collision detection, raycasting, and kinematic character controllers. A physics engine is needed.
- **Alternatives Considered:**
  - bevy_rapier (Rapier physics wrapper for Bevy)
  - Avian 3D (formerly bevy_xpbd — ECS-native physics for Bevy)
  - Custom collision detection (rolling our own)
- **Decision:** Avian 3D v0.6
- **Rationale:** Avian is ECS-native — it uses Bevy's entity-component system directly, not a separate physics world. It supports kinematic rigid bodies (required for FPS character controllers), CCD for fast-moving projectiles, raycasting for hitscan weapons, and collider generation from mesh geometry for map loading. Version 0.6 is compatible with Bevy 0.18. The API is ergonomic and well-documented.
- **Consequences:**
  - Kinematic character controller must be built on top of Avian's kinematic body support
  - Avian is also pre-1.0 but follows Bevy's release cadence
  - Collider generation from complex meshes may need optimization for large maps

---

### D-005: lightyear for Networking

- **Date:** Engine selection phase
- **Status:** Accepted
- **Context:** Competitive FPS requires server-authoritative netcode with client-side prediction, rollback, lag compensation, and snapshot interpolation. No engine ships this out of the box — it must be built or integrated.
- **Alternatives Considered:**
  - lightyear 0.26 (Bevy-native networking library)
  - bevy_replicon (replication-focused plugin)
  - naia (networking library with Bevy support)
  - Custom netcode from scratch (maximum control)
  - bevy_ggrs (rollback-focused, better for fighting games)
- **Decision:** lightyear 0.26
- **Rationale:** lightyear has the most comprehensive FPS-relevant feature set: client-side prediction with rollback, snapshot interpolation, **built-in lag compensation for hitscan** (explicitly designed for FPS), bandwidth management, interest management via rooms, and dedicated `lightyear_avian3d` integration for physics replication. Version 0.26 is compatible with Bevy 0.18.
- **Consequences:**
  - Server-authoritative architecture from the start
  - Must design around lightyear's prediction/rollback model
  - lightyear is community-maintained (not Bevy official)
  - Network protocol is UDP with optional WebTransport for web builds

---

### D-006: Blender for All Art Assets

- **Date:** Engine selection phase
- **Status:** Accepted
- **Context:** Maps, character models, weapon models, and animations need to be created and exported to Bevy. Bevy has no editor, so an external tool is required.
- **Alternatives Considered:**
  - Blender for everything (maps, models, animations)
  - TrenchBroom for maps + Blender for models
  - Procedural generation only (no manual art tools)
- **Decision:** Blender for all art assets
- **Rationale:** Blender is free, open-source, and has excellent glTF 2.0 export. Using one tool for everything simplifies the art pipeline and reduces contributor onboarding friction. TrenchBroom is excellent for Quake-style BSP maps but produces brush-based geometry that doesn't align with JuanSchott's organic, vertical arena aesthetic.
- **Consequences:**
  - All art must be exported as glTF 2.0 (.glb binary format)
  - Avian's `Collider::trimesh_from_mesh` can auto-generate collision from exported geometry
  - No in-engine preview — artists must export and load in-game to see results
  - Blender files (.blend) are tracked in the repository for source assets

---

### D-007: Tactical Precision Over Speed

- **Date:** Game design phase
- **Status:** Accepted
- **Context:** Titanfall 2's movement inspired the project, but the game feel needed to be distinct. The spectrum ranges from slow and tactical (CS) to fast and chaotic (Quake).
- **Alternatives Considered:**
  - Arena shooter speed (Quake-level movement, bunny hopping, air strafing)
  - Titanfall 2 speed (fast but fluid)
  - Tactical precision (CS-like positioning, moderate movement speed)
  - Realistic slow (Squad/Insurgency pace)
- **Decision:** Tactical precision with parkour movement
- **Rationale:** The game's identity is "positioning is king." Low TTK means positioning advantage usually wins. Parkour movement (wall-run, double-jump, slide) makes positioning itself a skill. The combination — fast enough to be exciting, precise enough that every movement matters — creates a unique feel distinct from both arena shooters and military sims.
- **Consequences:**
  - Movement speeds are moderate: walk 4.5 m/s, sprint 7.0 m/s
  - Air control is limited (30%) — you commit to your jump trajectory
  - No bunny hopping, no air-strafe acceleration
  - Wall-running is time-limited (1.5 seconds max) to prevent it from becoming dominant

---

### D-008: Low TTK Design

- **Date:** Game design phase
- **Status:** Accepted
- **Context:** Time-to-kill defines the entire feel of an FPS. High TTK rewards tracking aim and sustained fire. Low TTK rewards positioning, first-shot advantage, and precision.
- **Alternatives Considered:**
  - High TTK (Quake/Halo — 1-2 seconds to kill, tracking aim matters)
  - Medium TTK (Overwatch/Valorant — 0.5-1.5 seconds, balanced tracking and precision)
  - Low TTK (CS/Valorant sharp — 0.2-0.8 seconds, positioning dominates)
- **Decision:** Low TTK (CS/Valorant sharp)
- **Rationale:** Low TTK reinforces Pillar 1 (Positioning Is King). When a single well-placed burst kills, the player who has the better angle wins. This makes movement and map knowledge more important than raw aim speed. It also creates high tension — every peek matters, every rotation is a risk.
- **Consequences:**
  - Weapons must be tuned so that a clean burst/headshot kills in 2-4 hits
  - Body-shot time-to-kill should be 0.5-1.0 seconds
  - Headshot multiplier is critical for rewarding precision
  - Regen means players reset quickly between fights, keeping pace high

---

### D-009: Regenerating Health

- **Date:** Game design phase
- **Status:** Accepted
- **Context:** Health systems define match pacing. Pickups create map control dynamics. Fixed HP creates simplicity. Regen keeps the pace moving.
- **Alternatives Considered:**
  - Map pickups (Quake-style — creates map control metagame)
  - Fixed HP per life (CS-style — resets on death)
  - Regenerating health (CoD/Halo — regens after damage-free period)
  - Loadout-based armor (variable max HP based on selection)
- **Decision:** Regenerating health
- **Rationale:** JuanSchott's pace is fast and fluid. Players are constantly rotating, flanking, and repositioning. Forcing players to hunt for health pickups would slow the game down and incentivize camping. Regen after 3 seconds of no damage means players can disengage, recover, and re-engage quickly. Combined with low TTK, this creates a rhythm: peek, fight, disengage or die, recover, re-engage.
- **Consequences:**
  - Base HP: 100
  - Regen delay: 3 seconds of no incoming damage
  - Regen rate: full HP over 2 seconds
  - Add-ons can modify regen delay, max HP, or damage reduction
  - No health pickups needed on maps

---

### D-010: Seasonal Stakes Over Per-Match Stakes

- **Date:** Lore integration phase
- **Status:** Accepted
- **Context:** The lore states that the Pantheon judges humanity's fighters and punishes or rewards Earth accordingly. The question was whether this judgment applies per match or per season.
- **Alternatives Considered:**
  - Per-match stakes (every match determines if Earth survives — too dramatic for a multiplayer game)
  - Per-season stakes (collective performance over a season determines Earth's fate)
  - No stakes (lore is pure flavor, no gameplay connection)
- **Decision:** Per-season stakes
- **Rationale:** Per-match existential stakes create a tonal contradiction — you lose a match but the planet is fine. This trivializes the lore. Per-season stakes create a narrative arc: the season begins, fighters compete over weeks, and the Pantheon renders judgment at the end. The orbit shift (reward or punishment) is cumulative over many seasons. This matches how live-service games actually work and gives the lore a natural delivery cadence.
- **Consequences:**
  - Each season has a narrative arc with a Pantheon host
  - Season end triggers a judgment event (lore/cinematic, not gameplay-affecting)
  - Earth's condition evolves over time based on seasonal outcomes
  - Matches themselves are sport, not survival

---

### D-011: Modular Add-On System (3 Slots)

- **Date:** Game design phase
- **Status:** Accepted
- **Context:** Players need customization for strategic variety, but the game is not a hero shooter. The system must allow meaningful choices without creating power creep or meta builds.
- **Alternatives Considered:**
  - No customization (pure gunplay only)
  - Class-based system (predefined kits)
  - Unlimited add-ons with tradeoffs
  - Limited slots (pick 3 from a pool)
- **Decision:** Limited slots — 3 add-on slots from a shared pool
- **Rationale:** 3 slots force meaningful choices. You cannot take every useful add-on. Each slot is a tradeoff: do you stack damage, or balance damage with a movement add-on? Do you take an active ability that has a cooldown, or a passive that is always on? The limited system makes counter-picking readable — if you see an opponent using a specific add-on's visual cue, you can adjust your own loadout.
- **Consequences:**
  - All add-ons must be roughly equal in power (different, not better)
  - Every add-on needs a visual or audio tell so opponents can read it
  - Counter-play must exist for every add-on
  - Categories: damage, movement, health, perks

---

### D-012: Eastern Temple x Alien Art Direction

- **Date:** Art direction phase
- **Status:** Accepted
- **Context:** The Vimana is a god-built arena. Its visual identity needs to communicate both sacred reverence and alien unknowability. The low-poly high-finish style must have a strong thematic anchor.
- **Alternatives Considered:**
  - Clean sci-fi (white walls, neon accents — Portal/Overwatch style)
  - Brutalist concrete (raw geometric spaces)
  - Gothic cathedral (tall arches, dramatic lighting)
  - Eastern temple x alien (Angkor Wat geometry at cosmic scale, Hindu temple proportions, carved surfaces in unknown languages)
- **Decision:** Eastern temple x alien
- **Rationale:** The Vimana is named after the Sanskrit concept of divine flying machines. Hindu and Buddhist temple architecture provides sweeping geometry, layered surfaces, and a sense of sacred enormity — exactly the feeling of fighting inside something built by gods. Making it *alien* rather than *accurately Hindu* prevents cultural appropriation while preserving the aesthetic power. The geometry is inspired by, not copied from, real temples.
- **Consequences:**
  - Map geometry uses vertical pillars, tiered platforms, sweeping curves reminiscent of temple architecture
  - Surfaces have carved patterns that are NOT any real script — they are alien inscriptions
  - Scale is deliberately wrong for human proportions (doorways too tall, steps too wide)
  - Color palette draws from warm stone, deep reds, golds, and alien bioluminescent accents
  - This is a reference and inspiration, NOT direct cultural replication

---

## Changelog

| Date | Change |
|------|--------|
| 2026-05-05 | Initial decisions log created (D-001 through D-012) |

## See Also

- [Vision and Origin](./vision_and_origin.md) — how the project started
- [Design Pillars](./design_pillars.md) — the rules governing all decisions
- [Anti-Goals](./anti_goals.md) — what we explicitly decided NOT to do
