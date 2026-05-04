# Map Design Philosophy [SYS-006]

**Document ID:** SYS-006  
**Category:** Game Design — Systems  
**Status:** Draft  
**Version:** 0.1.0  
**Author(s):** Design Team  
**Last Updated:** 2026-05-05  

---

## Table of Contents

1. [Core Principle: Full 3D Utilization](#1-core-principle-full-3d-utilization)
2. [The Three Rules of JuanSchott Map Design](#2-the-three-rules-of-juanschott-map-design)
3. [Map Structure by Mode](#3-map-structure-by-mode)
4. [Vertical Design Language](#4-vertical-design-language)
5. [Cover Design Principles](#5-cover-design-principles)
6. [Environmental Storytelling in Maps](#6-environmental-storytelling-in-maps)
7. [Spatial Readability and Wayfinding](#7-spatial-readability-and-wayfinding)
8. [Dynamic Map Elements](#8-dynamic-map-elements)
9. [Map Flow and Pacing Analysis](#9-map-flow-and-pacing-analysis)
10. [Technical Constraints and Guidelines](#10-technical-constraints-and-guidelines)
11. [Map Naming Convention](#11-map-naming-convention)
12. [Design Verification Checklist](#12-design-verification-checklist)
13. [See Also](#13-see-also)

---

## 1. Core Principle: Full 3D Utilization

JuanSchott maps are **volumes**, not floors. This is the single most important concept every map designer must internalize. A traditional FPS map is a layout drawn on a 2D plane with height added as an afterthought — elevations, stairs, maybe a second floor. A JuanSchott map is a sculpted space where every surface — walls, ceilings, floating structures, even the underside of platforms — is potential combat space. The ground is just one of many options available to a player at any given moment.

This principle derives directly from the movement system (see `movement_system.md`). Players can double-jump, wall-run, dash, and chain these abilities into complex traversal sequences. Maps must be designed to reward and require this vocabulary. A player standing on the ground in JuanSchott should feel like they are swimming at the bottom of the ocean — they have an entire volume above and around them that they can and should be using. The map must make players think in three dimensions constantly, not as a novelty but as a survival requirement.

The practical implication is that map design begins with **volumetric thinking**. Designers should sketch maps as 3D spaces from the first concept, not as floor plans with vertical elements added later. Every design review should include questions like: "What is happening at the ceiling level?" and "Can a player traverse this space without ever touching the ground?" If the answer to the second question is no for any major corridor or arena, the map needs more vertical infrastructure.

### 1.1 The Volume, Not the Floor

Traditional level design workflow begins with a top-down floor plan. JuanSchott map design must begin with a volumetric blockout. Designers should think in terms of occupied and unoccupied space, density gradients, and vertical connectivity before considering any ground-level layout. The ground plane is the last surface to be detailed, not the first.

This inversion of the standard workflow has cascading effects on every aspect of design. Cover placement, sight line construction, spawn point selection, and objective positioning all must be evaluated from every possible approach angle — including from above and below, not just from the horizontal plane.

### 1.2 Movement Integration Maps

Maps must be built to serve the movement system, not the other way around. Every surface a player can interact with — walls, floors, ceilings, floating geometry — must be evaluated through the lens of movement verbs. Can a player wall-run here? Can they chain a wall-run into a double-jump to reach that platform? Is this gap dash-crossable? If a movement option exists in the player's kit, the map must create situations where that option is the correct choice. No movement ability should feel orphaned or unused.

---

## 2. The Three Rules of JuanSchott Map Design

These three rules are inviolable. Every map, regardless of mode or size, must satisfy all three. They form the design invariant against which every layout decision is measured.

### Rule 1: No Dead Ground

Every square meter of the map must serve a purpose — either as a combat position, a movement path, or a visual landmark. If a space has no reason to exist, it should not exist.

Dead ground is any area where nothing meaningful can happen. A corridor with no cover, no branching paths, and no vertical options is dead ground. A flat rooftop with no access points is dead ground. A wide-open arena floor with no floating cover is dead ground. These spaces waste the player's time, reduce the information density of the map, and create boring traversal sequences between actual gameplay moments.

Every space should answer at least one of these questions affirmatively:
- Can a fight happen here?
- Can a player meaningfully traverse through here (not just run through it)?
- Does this space provide navigation or orientation value?
- Does this space support a specific movement verb (wall-run, double-jump chain, dash)?

If a space answers "no" to all four, it must be redesigned or removed.

### Rule 2: Three Routes Minimum

Between any two significant points in the map, there must be at least three distinct routes:

- **Ground Route** — Direct, but exposed. The path of least resistance and greatest vulnerability. Ground routes are the "default" option that skilled players will avoid because exposure equals death.
- **Vertical Route** — Requires parkour skill. Involves wall-runs, double-jumps, or other movement abilities. Rewards spatial mastery and mechanical skill. Not every player will be able to execute these routes consistently, which creates a natural skill gradient.
- **Flank Route** — Longer, but safer. Takes the player out of direct engagement to approach from an unexpected angle. May also incorporate vertical elements. The cost is time; the reward is positional advantage.

This rule ensures players always have options and no single position is unassailable. It also creates natural information asymmetry — if there are three routes between points A and B, a defender at point B cannot watch all three simultaneously, which guarantees that attackers always have an unobserved approach.

The three-route minimum applies at every scale: between spawn and the nearest combat zone, between adjacent combat arenas, between a ground position and an elevated platform, between one side of the map and the other.

### Rule 3: Vertical Sight Lines

Every horizontal sight line must have a vertical counter-position. If a sniper can see down a corridor from ground level, there must be a position above that corridor from which the sniper can be flanked. If a player holds an angle from an elevated position, there must be a higher position or an approach path that bypasses that angle.

This rule enforces 3D thinking by making every dominant position contestable from above or below. It prevents the formation of impenetrable sight-line locks and ensures that map control is always fluid. A player who secures a powerful position must remain aware that their position is itself vulnerable to vertical flanking. This creates a recursive structure of advantage and counter-advantage that drives constant movement and engagement.

The practical test for this rule is the **Sight Line Audit**: for every long sight line in the map, the designer must identify at least one position that overlooks or flanks that sight line from a different vertical level. If none exists, the map fails the audit.

---

## 3. Map Structure by Mode

### 3.1 Duel Maps (1v1)

| Parameter | Value |
|---|---|
| Footprint | ~30m x 30m |
| Vertical Extent | ~20m |
| Players | 2 |
| Average Engagement Distance | 5–15m |
| Target Match Duration | 3–5 minutes |

**Layout Philosophy:** Central arena with 2–3 elevated positions, 1–2 wall-run surfaces, all interconnected via parkour. Duel maps are intimate. Every element must serve the duel directly — there is no room for decorative space.

**Pacing:** Fast, constant contact. Players should be able to find each other within 5 seconds of match start or after a kill. The map should never allow a player to "hide" — it encourages fighting, not camping.

**Required Features:**

- **1 central elevated platform** (the "throne") — highly contested, visible from everywhere on the map. Holding this position grants excellent sight lines but makes the holder visible to all approaches. It is the bait that draws players together.
- **2–3 wall-run routes** connecting different levels. These are the skill-expression paths. A player who can consistently execute wall-run routes has a significant movement advantage over one who cannot.
- **Ground-level cover** that is destructible or temporary. Ground cover should not provide permanent safe spots. It exists to create momentary respite, not to enable camping. Destructible cover also provides audio information — when cover breaks, both players know where the engagement happened.
- **No hiding spots.** Every position in a duel map should be reachable and contestable within seconds. If a player can sit in a corner unseen for more than 3 seconds, the map has failed its design intent.

**Design Anti-Patterns for Duels:**
- Long corridors with no vertical options (forces linear combat, eliminates 3D advantage)
- Permanent ground-level cover that can absorb unlimited damage (enables passive play)
- Spawn points visible from the throne position (gives spawn advantage to the player already at throne)
- More than 4 distinct levels (creates too much vertical space for 2 players, leads to searching rather than fighting)

### 3.2 Skirmish Maps (3v3)

| Parameter | Value |
|---|---|
| Footprint | ~60m x 60m |
| Vertical Extent | ~30m |
| Players | 6 |
| Average Engagement Distance | 8–25m |
| Target Match Duration | 8–12 minutes |

**Layout Philosophy:** Two halves with a contested center. Multiple vertical layers (3–4 levels of play). Team spawn zones on opposite ends. The two-halves structure creates natural front lines and contest zones, while multiple vertical layers ensure that "front line" is not a flat barrier but a volumetric engagement space.

**Pacing:** Rhythmic. Periods of rotation and positioning punctuated by team fights at contest points. The map should create natural "pull" toward contest areas without forcing players into them. Teams that coordinate vertical movement should have a measurable advantage over teams that play on the ground.

**Required Features:**

- **3–4 major combat arenas** connected by movement corridors. Each arena is a self-contained combat space with its own vertical options. Corridors between arenas are not dead space — they contain movement infrastructure (wall-run chains, floating platforms) that makes traversal itself a skill expression.
- **Vertical options in every arena** — wall-run surfaces, elevated positions, floating platforms. No arena should be playable entirely from the ground.
- **Mid-air cover structures** — geometric shapes suspended in space that break sight lines and create 3D engagements. These are the defining feature of skirmish-scale combat. A fight happening at mid-height between two floating platforms is the quintessential JuanSchott engagement.
- **Elevated and protected spawn zones.** Spawn zones must be positioned so that newly spawned players cannot be immediately engaged by opponents. Elevation provides natural protection — spawning above the combat zone gives the player a moment to orient before descending.
- **Sound cues built into architecture.** Certain surfaces produce louder footsteps. Certain paths are quieter (soft material, gravity-muted zones). This adds an information layer for attentive players and creates tactical decisions about route selection based on audio profile.

**Design Anti-Patterns for Skirmishes:**
- Single chokepoint connecting the two halves (creates a static meat-grinder rather than dynamic team fights)
- Spawn zones at ground level with direct sight lines from contest areas (enables spawn camping)
- More than 5 combat arenas (spreads 6 players too thin, leads to walking simulator between fights)
- Asymmetrical resource distribution (one team's half has more vertical options than the other)

### 3.3 Clash Maps (6v6)

| Parameter | Value |
|---|---|
| Footprint | ~100m x 100m |
| Vertical Extent | ~40m |
| Players | 12 |
| Average Engagement Distance | 10–40m |
| Target Match Duration | 12–18 minutes |

**Layout Philosophy:** Large arena with 4–5 distinct zones connected by parkour highways. Multiple fights happening simultaneously across the volume. Clash maps are where JuanSchott's design philosophy reaches its fullest expression — the entire map is alive with movement and combat at every level.

**Pacing:** Chaotic but readable. The minimap is essential. Hot zones should be visible on the HUD. Players must be able to quickly assess the state of the match and decide where to commit without needing to physically traverse the entire map to gather information.

**Required Features:**

- **4–5 "neighborhoods"** each with distinct character and combat style. A neighborhood might be a dense vertical maze favoring close-quarters combat, or an open aerial plaza favoring long-range engagements. This variety ensures that different weapon and playstyle loadouts have natural advantage zones.
- **Parkour highways** — long wall-run chains and double-jump paths connecting neighborhoods quickly. These are the express routes of the clash map. A player who knows the highway network can traverse the map in a fraction of the time it takes a ground-level player. Highway mastery is a meta-skill that develops over many matches.
- **Central mega-structure visible from everywhere** (navigation landmark). In a 100m x 100m map, orientation is a real challenge. A central landmark provides a constant reference point. Players should always be able to look up, see the mega-structure, and know roughly where they are relative to the map's center.
- **Multiple spawn zones per team** (randomized to prevent camping). Spawn selection should be randomized from a pool of valid positions. The pool should be large enough that opponents cannot predict spawn location with certainty.
- **Environmental storytelling elements** — Vimana architecture, Pantheon motifs, remnants of previous species' arenas. At clash scale, the map is large enough to tell a story through its architecture. Each neighborhood can reflect a different Pantheon host's aesthetic or a different era of the Vimana's history.

**Design Anti-Patterns for Clashes:**
- Fewer than 3 major routes between any two neighborhoods (creates chokepoint dependency)
- Neighborhoods without vertical infrastructure (becomes a traditional FPS space, breaks 3D philosophy)
- Ground-level-only connections between neighborhoods (forces linear traversal, eliminates highway gameplay)
- Central mega-structure that is purely decorative (wastes the most prominent position in the map — it should be playable space)

---

## 4. Vertical Design Language

### 4.1 Floating Platforms

**Purpose:** Mid-air cover and combat positions. The defining feature of JuanSchott maps.

**Placement:** Positioned to break long horizontal sight lines. Accessible via double-jump from below or wall-run from adjacent surfaces. Platforms should feel like they belong in the space — they are architectural elements of the Vimana, not arbitrary gameplay props.

**Size:** Large enough for 1–2 players. Not team-sized. A platform that holds an entire team eliminates the vulnerability that makes positioning meaningful.

**Geometry:** Low-poly geometric shapes — cubes, wedges, cylinders. Aligned with the Vimana's temple aesthetic. Clean geometry serves double duty: it reads clearly as gameplay space and it fits the established visual language.

**Variety:** Some platforms are static. Some are mobile (slowly rotating or shifting position on a timer). Mobile platforms add dynamism to sight lines and create time-pressure decisions — a player must judge not only whether they can reach a platform but whether it will be in the right position when they arrive. Mobile platform behavior should be predictable (cyclical) rather than random, so players can learn and anticipate patterns.

### 4.2 Wall-Run Surfaces

**Purpose:** Movement paths that reward spatial skill. Must be deliberately placed, not scattered.

**Placement:** Adjacent to high-traffic areas. Players should naturally discover wall-run shortcuts by moving through the map. A wall-run surface that is hidden away in a corner no one visits is wasted design.

**Length:** 8–15 meters of runnable surface. Long enough to be useful as traversal, short enough to require timing and precision. Wall-runs that are too long become trivial; too short become frustrating.

**Geometry:** Flat vertical surfaces. Must be clearly readable as wall-runnable through visual texture difference and a slight inward angle (2–5 degrees from vertical) that subtly communicates "this surface is for you." The visual distinction is critical — players must be able to identify wall-run surfaces at a glance during combat without stopping to examine geometry.

**Chaining:** Where possible, wall-run surfaces should chain — run one wall, jump to adjacent wall, run that wall. Chained wall-runs create "parkour highways" that serve as the rapid transit system of the map. Highway design should consider momentum: each wall in a chain should flow naturally into the next, with jump timing feeling intuitive rather than punishing.

### 4.3 Elevated Positions

**Purpose:** Sniper nests, recon spots, high-ground advantage.

**Placement:** Overlooking major combat areas but accessible via at least 2 routes to prevent camping. The elevated position must be strong enough to be worth fighting for but not so strong that holding it is trivial.

**Tradeoff:** Powerful position but exposed from multiple angles. You can see a lot, but a lot can see you. This is the fundamental balance of elevated positions in JuanSchott — the information advantage of height comes with an exposure cost. An elevated player who focuses on one target is vulnerable to approach from other angles.

**Counter Design:** Every elevated position must have a vertical counter — a higher position or a flank route that reaches it. This creates a positional hierarchy: ground → elevated → counter-elevated. No position sits at the top of this hierarchy permanently, because the counter-elevated position is itself accessible from another angle.

---

## 5. Cover Design Principles

### 5.1 Mid-Air Cover

Cover does not just exist on the ground. Floating geometric shapes at various heights create 3D cover throughout the map's volume. Players can duck behind floating platforms, use vertical pillars as cover, or hover near wall-run surfaces to break line of sight. This is the key differentiator — in most FPS games, cover is a ground-level concept. In JuanSchott, cover is volumetric. A player in mid-air should have cover options just as a player on the ground does.

### 5.2 Cover Must Be Readable

All cover elements must be visually distinct from non-cover geometry. Cover should have a consistent visual language — harder or different material from decorative surfaces. Low-poly style aids readability: clean geometry makes cover obvious. Players should never die because they misjudged whether a surface provided cover. If there is ambiguity, the design has failed.

The readability standard is the **Two-Second Test**: a player entering a space for the first time should be able to identify all major cover positions within two seconds. If cover is camouflaged or visually ambiguous, it violates this standard.

### 5.3 No Indestructible Safe Spots

Every position should be contestable. No "bunker" positions where a player can hide indefinitely. If a position is very strong (excellent cover, good sight lines, multiple escape routes), it should have limited access that requires parkour to reach and can be approached from multiple angles simultaneously. The strength of a position should be proportional to the skill required to reach and hold it.

---

## 6. Environmental Storytelling in Maps

Every map tells a story through its architecture. The Vimana is a living arena that has hosted combat for millennia. This history should be visible in the environment.

### 6.1 Pantheon Architecture

Each Pantheon member's aesthetic influence is visible in the arenas they host. The Architect's arenas are symmetrical, geometric, and precise — every surface placed with mathematical intention. The Flame's arenas are chaotic, hazardous, and aggressive — surfaces fractured, paths unpredictable, danger integrated into the geometry itself. These aesthetic differences are not purely cosmetic; they affect readability, flow, and combat character.

### 6.2 Historical Layers

Remnants of previous species' arenas exist as geological layers in the architecture — altered geometry where old passages were sealed, alien materials integrated into Vimana structures, incomprehensible inscriptions on surfaces. These elements add visual richness and reinforce the lore without requiring explicit exposition.

### 6.3 Combat Scarring

Weapon scars from past matches are visible as texture variation on surfaces. Heavily contested areas show more damage than quiet corridors. This is both narrative flavor and subtle gameplay information — a heavily scarred area signals to new players that this is a high-traffic combat zone.

### 6.4 The Living Vimana

The Vimana is not a static structure. Subtle changes between matches — new cracks in surfaces, slightly shifted geometry, different ambient lighting — reinforce the sense that the arena is alive. These changes should be cosmetic, not gameplay-altering, to maintain competitive fairness.

### 6.5 Gravity Anomaly Zones

Areas where physics behaves differently — visual distortion effects, floating debris, altered fall speed. These zones are both environmental storytelling (evidence of the Vimana's reality-manipulating nature) and gameplay modifiers that create unique combat conditions. Gravity anomaly zones should be clearly marked visually so players can account for their effects.

---

## 7. Spatial Readability and Wayfinding

### 7.1 Landmark Hierarchy

Every map must establish a clear hierarchy of visual landmarks to aid navigation:

- **Primary landmark:** The central mega-structure, visible from everywhere. Provides absolute orientation.
- **Secondary landmarks:** Major architectural features unique to each neighborhood or arena. Provide relative orientation within a zone.
- **Tertiary landmarks:** Cover elements, spawn markers, and detail geometry that provide local orientation.

### 7.2 Color and Material Language

Consistent use of color and material creates a subconscious navigation system. Spawn areas use a distinct palette. High-traffic combat zones use another. Movement corridors use a third. Players should eventually be able to identify their map position from visual cues alone, without consulting the minimap.

### 7.3 Vertical Orientation

Vertical orientation is a unique challenge in 3D maps. Players must be able to quickly determine their current height relative to the map's floor and ceiling. Techniques include: gradient lighting (darker at bottom, brighter at top), visible "floor" and "ceiling" elements that establish the vertical boundaries, and HUD indicators showing current elevation.

---

## 8. Dynamic Map Elements

### 8.1 Mobile Platforms

Platforms that move on predictable cycles add a time dimension to spatial reasoning. Players must account not only for position but for timing. Mobile platforms should move slowly enough that skilled players can use them consistently but fast enough that they create meaningful sight line changes.

### 8.2 Environmental Hazards

Selected maps may include environmental hazards — energy barriers, gravity wells, collapsing surfaces. These hazards must:
- Be clearly telegraphed visually and audibly
- Operate on predictable patterns
- Never feel random or unfair
- Create interesting combat decisions rather than arbitrary deaths

### 8.3 Match Progression

For longer game modes (Clash), maps may evolve over the course of a match — new paths open, existing paths close, environmental hazards activate or deactivate. These changes should be communicated clearly to all players and should create new tactical opportunities rather than punishing existing strategies.

---

## 9. Map Flow and Pacing Analysis

### 9.1 Encounter Density

Each map mode targets a specific encounter density — the frequency of meaningful combat interactions per minute. Duel maps target high density (combat every 5–10 seconds). Skirmish maps target medium density (team engagements every 30–60 seconds with individual skirmishes between). Clash maps target variable density (multiple simultaneous engagements with rotation periods between).

### 9.2 Rotation Time

The time it takes to rotate from one side of the map to the other must be balanced against the time-to-kill and respawn timer. A player who dies and respawns should be able to return to the fight within a reasonable window. For duel maps, rotation time should be under 5 seconds. For skirmish maps, under 8 seconds. For clash maps, under 12 seconds via parkour highways (longer via ground routes).

### 9.3 Flow Testing Protocol

Every map must undergo flow testing with the following metrics:

- **Contact Rate:** Time between encounters. Must fall within target range for the mode.
- **Route Distribution:** Percentage of player traversals using each route type. If one route is used more than 60% of the time, other routes need improvement.
- **Vertical Utilization:** Percentage of combat time spent above ground level. Target: 40% or higher for all modes.
- **Position Contention:** How often the highest-value positions change hands. If one position is held for more than 60% of a match, it needs more counter-routes.

---

## 10. Technical Constraints and Guidelines

### 10.1 Performance Budgets

- **Polycount:** Each map must stay within the established polycount budget per mode. Duel: 150k tris. Skirmish: 300k tris. Clash: 500k tris.
- **Draw Calls:** Maximum 800 draw calls per frame across all modes.
- **Collision Meshes:** Simplified collision geometry must be provided for all walkable and wall-runnable surfaces. Visual meshes must never be used directly for collision.

### 10.2 Modular Construction

Maps should be constructed from a shared library of modular architectural pieces to maintain visual consistency across the game and reduce asset creation overhead. Custom pieces are permitted for unique landmarks but must follow the established visual language defined in `../03_art/visual_language.md`.

### 10.3 Blocking and Iteration

All maps begin as greybox blockouts using primitive geometry. Greybox must pass all three design rules (No Dead Ground, Three Routes Minimum, Vertical Sight Lines) before any visual development begins. This ensures that spatial design drives aesthetic decisions, not the reverse.

---

## 11. Map Naming Convention

Maps are named after their architectural character or Pantheon host:

| Codename | Mode | Host/Character | Description |
|---|---|---|---|
| MAP-D01 | Duel | [Name TBD] — The Architect | Symmetrical, geometric, precise |
| MAP-D02 | Duel | [Name TBD] — The Flame | Chaotic, hazardous, aggressive |
| MAP-S01 | Skirmish | [Name TBD] | [Description TBD] |
| MAP-S02 | Skirmish | [Name TBD] | [Description TBD] |
| MAP-C01 | Clash | [Name TBD] | [Description TBD] |
| MAP-C02 | Clash | [Name TBD] | [Description TBD] |

Each map has a codename for internal use following the pattern `MAP-{Mode}{Number}` where Mode is D (Duel), S (Skirmish), or C (Clash).

---

## 12. Design Verification Checklist

Every map must pass the following checklist before advancing from greybox to art pass:

- [ ] **Rule 1 Audit:** Every square meter serves a purpose (combat, movement, or navigation).
- [ ] **Rule 2 Audit:** Three routes minimum between all significant point pairs.
- [ ] **Rule 3 Audit:** Every horizontal sight line has a vertical counter-position.
- [ ] **Sight Line Audit:** No uncounterable sight lines exist.
- [ ] **Spawn Safety:** Spawn points cannot be directly fired upon from any combat position.
- [ ] **Vertical Coverage:** At least 40% of playable space exists above ground level.
- [ ] **Wall-Run Readability:** All wall-run surfaces are visually distinguishable.
- [ ] **Cover Readability:** Two-Second Test passed for all combat areas.
- [ ] **Flow Metrics:** Contact rate, route distribution, and vertical utilization within target.
- [ ] **Performance Budget:** Polycount, draw calls, and collision meshes within limits.
- [ ] **Lore Integration:** Environmental storytelling elements present and consistent with established Vimana lore.

---

## 13. See Also

- `movement_system.md` — Player movement verbs that maps must support
- `weapon_system.md` — Weapon range and sight line requirements
- `game_modes.md` — Mode-specific rules affecting map design
- `../03_art/visual_language.md` — Visual style guide for map aesthetics
- `../01_lore/the_vimana.md` — Vimana lore informing environmental storytelling
