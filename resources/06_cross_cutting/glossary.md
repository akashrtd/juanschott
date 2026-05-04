# Glossary

## Document Metadata

| Field | Value |
|---|---|
| Document ID | CCD-003 |
| Version | 1.0 |
| Status | Draft |
| Owner | Design Lead |
| Last Updated | 2026-05-05 |

---

## Purpose

This document defines every term used across all JuanSchott design documents, technical specifications, and communications. It is the single source of truth for terminology. If a term is used in any project document, it is defined here.

All terms are organized alphabetically. Related terms are cross-referenced.

---

## A

### Add-On

A modular equipment piece that modifies player capabilities. Each player has **3 add-on slots** in their loadout. Add-ons provide active abilities (triggered by the player) or passive effects (always active). Every add-on includes a trade-off — a negative effect that offsets the benefit. Add-ons are implemented as ECS components attached to the player entity.

See also: *Cooldown, Duration, Loadout, Trade-off*

### ADS (Aim Down Sights)

A weapon aiming mode where the player looks through the weapon's sight or scope. ADS reduces bullet spread, slightly reduces movement speed, and zooms the camera. All weapons support ADS. ADS is distinct from hip-fire, which is the default weapon state.

See also: *Spread (Hip), Spread (ADS), Hip-Fire*

### Avian

An ECS-native physics engine for Bevy. Avian provides collision detection, raycasting, kinematic body simulation, and spatial queries. JuanSchott uses Avian for all physics: player movement, projectile simulation, collision detection, and hit detection raycasts. Avian integrates directly with Bevy's ECS, meaning physics data is stored in components and physics logic runs in systems.

See also: *Bevy, ECS, Kinematic, Raycast*

---

## B

### Bevy

An open-source game engine written in Rust, using the Entity Component System (ECS) architecture. Bevy is the foundation of JuanSchott. All game logic, rendering, audio, networking, and UI are implemented as Bevy systems operating on ECS entities and components. Key Bevy features used: App, Commands, Query, Res, Resource, System, Schedule, States.

See also: *ECS, Avian, lightyear, wgpu*

### Beam

A continuous-damage weapon type. A beam weapon fires a visible line from the weapon to the target. Damage is applied in ticks (not continuously). The beam persists as long as the player holds the trigger and has ammunition. Beams are hitscan — they instantly connect to the first target in the line of fire.

See also: *Hitscan, Projectile, Tick Damage*

---

## C

### Cannonical Hierarchy

The ordering of what lore sources are most authoritative. Defines which sources override others when conflicts arise. The hierarchy from most to least authoritative: Core Canon, Established Canon, Soft Canon, Flexible Canon, Non-Canon. Governed by the Canon Hierarchy document (CCD-005).

See also: *Canon Hierarchy, Core Canon, Established Canon, Soft Canon, Flexible Canon*

### Cycle

One complete appearance and departure of the Vimana. A Cycle corresponds to one **season** in the game's live-service framework. When the Vimana arrives, a new Cycle begins. When it departs, the Cycle ends. The term "Cycle" is used in lore contexts; "Season" is used in operational contexts. They refer to the same period.

See also: *Season, Vimana, Invitation*

### Cybersymbiote

One of four playable species in JuanSchott. A human bonded with a symbiotic machine organism. The bonding process enhances physical capabilities and enables interfacing with certain Vimana systems. Cybersymbiotes are visually distinct from baseline humans due to visible cybernetic integration.

See also: *Species, Vimana*

---

## D

### Damage Falloff

A linear reduction in weapon damage beyond a certain range. Weapons deal full damage up to a `falloff_start` distance. Beyond that point, damage decreases linearly until the `falloff_end` distance, at which point the weapon deals its minimum damage (`min_damage`). Damage falloff ensures that weapons have effective range bands and encourages map-aware positioning.

Formula: `damage = max(min_damage, base_damage - (base_damage - min_damage) * (distance - falloff_start) / (falloff_end - falloff_start))` when `distance > falloff_start`.

See also: *Hitscan, Projectile, Range*

### Double Jump

A second jump available while airborne. The double jump propels the player upward with slightly less force than the initial jump. It resets when the player lands on a surface. Double jump is a core movement ability available to all players at all times — it is not tied to an add-on. The double jump has a distinctive energy-boost visual and audio effect.

See also: *Jump, JUMP_VELOCITY, DOUBLE_JUMP_VELOCITY, Wall Jump*

---

## E

### ECS (Entity Component System)

An architecture pattern used by Bevy. In ECS:

- **Entities** are lightweight IDs (just a number).
- **Components** are data structs attached to entities (position, health, weapon).
- **Systems** are functions that operate on entities with specific component combinations.

JuanSchott uses ECS exclusively. There are no object hierarchies, no inheritance chains. A player is an entity with components: `Transform`, `PlayerController`, `Health`, `Loadout`, `Velocity`, etc. A bullet is an entity with components: `Transform`, `Velocity`, `Projectile`, `Damage`.

See also: *Bevy, Entity, Component, System*

### ELO / MMR

The skill rating system used for ranked matchmaking. **ELO** (named after Arpad Elo) is the rating algorithm. **MMR** (Matchmaking Rating) is the numerical value representing a player's skill. Players are matched with and against players of similar MMR. MMR is adjusted after each match based on the outcome and the relative skill of opponents. MMR is per-season — each season starts with a soft reset from the previous season's rating.

See also: *Ranked, Season, Matchmaking*

---

## G

### glTF

GL Transmission Format. A 3D asset format for models, animations, and scenes. JuanSchott uses glTF for all 3D assets: player models, weapon models, map geometry, and environmental objects. Bevy has native glTF support. All assets are authored as `.glb` (binary glTF) files for optimal loading performance.

See also: *Bevy, Low-Poly High-Finish*

### Gravity

The acceleration applied to player entities when airborne. JuanSchott uses a fixed gravity value applied as a downward force each physics frame. The gravity value is tuned to produce satisfying jump arcs and fall speeds that match the game's fast-paced movement. In zero-g map sections, gravity is set to zero or near-zero.

See also: *GRAVITY, Zero-G, Jump, Wall Run*

---

## H

### Headshot

A hit to the top 20% of the player's capsule collider. Headshots deal 2.5x the weapon's base damage. The headshot zone is determined by the hit position relative to the capsule's vertical extent. Headshots are the primary mechanism for achieving low TTK kills and reward precise aiming.

See also: *HEADSHOT_MULTIPLIER, TTK, Hitscan, Capsule Collider*

### Hip-Fire

Discharging a weapon without entering ADS. Hip-fire has wider spread than ADS but allows full movement speed. Hip-fire is the default weapon state. The crosshair indicates current spread level.

See also: *ADS, Spread (Hip), Spread (ADS)*

### HP (Health Points)

The numerical value representing a player's current health. All players have a base HP of 100. When HP reaches 0, the player dies. HP regenerates after a damage-free period (see Regen). HP is stored as a component on the player entity.

See also: *BASE_HP, Regen, REGEN_DELAY, REGEN_RATE, Health*

### Hub

The central area of the Vimana. The inner surface of the cylindrical megastructure where the surface maps exist. The Hub is where most gameplay takes place. It is the "ground level" of the arena — a curved surface that wraps around the interior of the Vimana. The Hub has artificial gravity directed toward the inner surface.

See also: *Vimana, Zero-G, Map*

---

## I

### Invitation

The mental transmission sent by the Vimana to all humans when it arrives at the start of a new Cycle. The Invitation is a lore concept — it explains why players are compelled to enter the Vimana and compete. The nature and content of the Invitation varies by Pantheon host. The Invitation is experienced by all humans simultaneously and is impossible to ignore.

See also: *Vimana, Cycle, Pantheon, Season*

---

## J

### JuanSchott

The game's name. Also the protagonist title. The name is derived from "Juan" (a name suggesting singularity, "one") and "Schott" (a term combining connotations of a shot and a vessel/ship). Alternate reading: "One Shot" or "God's Blessing + Ship." The protagonist, JuanSchott, is a specific character in the lore — the individual who first entered the Vimana and returned. The name applies to both the game and the character.

See also: *Vimana, Pantheon, Cycle*

### Jump Buffer

A short input window (approximately 0.1-0.2 seconds) where a jump input is stored and executed when the player lands. If a player presses jump slightly before landing, the jump executes automatically on landing. This prevents frustration from timing-sensitive jump inputs and makes movement feel responsive.

See also: *JUMP_BUFFER, Jump, Double Jump*

---

## K

### Kinematic

A physics body type where movement is controlled directly by code, not by physics forces. Player characters in JuanSchott are kinematic bodies. They do not respond to external forces (explosions, collisions with other entities) through physics simulation. Instead, all player movement is explicitly calculated and applied by the movement system. This provides precise, predictable control essential for a competitive shooter.

See also: *Avian, Physics, Dynamic Body*

---

## L

### Lag Compensation

A server-side technique that rewinds the game state to validate hitscan shots from a client's perspective. When a client fires a hitscan weapon, the server receives the fire command with a timestamp. The server rewinds all player positions to what they were at that timestamp (from the firing client's network perspective) and performs the raycast. This ensures that players can hit what they see on their screen, even with network latency.

See also: *lightyear, Hitscan, Ping, Prediction, Rollback*

### lightyear

A networking library for Bevy. lightyear handles client-server architecture, state replication, client-side prediction, server reconciliation, and lag compensation. JuanSchott uses lightyear for all multiplayer networking. lightyear integrates with Bevy's ECS, treating networked state as replicated components.

See also: *Bevy, Prediction, Rollback, Lag Compensation, Replication*

### Loadout

The player's selected equipment configuration. A loadout consists of: 1 primary weapon, 1 secondary weapon, and 3 add-on slots. Players configure their loadout before a match or during respawn. Loadouts can be saved as presets.

See also: *Add-On, Weapon, Primary Weapon, Secondary Weapon*

### Low-Poly High-Finish

The art style of JuanSchott. Models use minimal geometry (low polygon count) with high-quality materials, textures, and lighting. The result is a clean, readable aesthetic where silhouettes are clear, performance is high, and the visual identity is distinctive. This style is chosen for both aesthetic and practical reasons: it is faster to produce, easier to read during fast gameplay, and performs well on a wide range of hardware.

See also: *Silhouette, glTF*

---

## M

### Mantle

Pulling up onto a ledge. A future movement mechanic not planned for launch. Mantling would allow players to grab the edge of a surface above them and pull themselves up, extending vertical mobility options. Currently deferred.

See also: *Wall Run, Wall Jump, Jump*

---

## P

### Pantheon

The collective of godlike beings who operate the Vimana. The Pantheon members are the closest thing to deities in the JuanSchott lore. They are not omnipotent — they are powerful, ancient entities with distinct personalities and domains. Each season, one Pantheon member serves as the host, influencing the season's theme and modifiers. The five Pantheon members are part of Established Canon.

See also: *Vimana, Season, Cycle, Canon Hierarchy*

### Ping

Network round-trip time in milliseconds (ms). Ping measures the time for a message to travel from the client to the server and back. Lower ping means more responsive gameplay. JuanSchott's lag compensation system is designed to provide fair gameplay up to approximately 100ms ping. The ping value is displayed in the HUD during gameplay.

See also: *Lag Compensation, lightyear, Prediction*

### Prediction

Client-side simulation of the player's own actions ahead of server confirmation. When a player presses jump, the client immediately simulates the jump locally without waiting for the server to confirm. The server later validates the action. If the server disagrees with the client's prediction, a rollback occurs.

See also: *Rollback, lightyear, Lag Compensation*

### Projectile

A weapon type that spawns a physical entity with velocity and travel time. Unlike hitscan weapons, projectiles do not instantly hit their target. They travel through space and can be dodged. Projectiles are simulated as ECS entities with Transform, Velocity, and Projectile components. They are affected by physics (collision with surfaces and players).

See also: *Hitscan, Beam, Splash Damage, Velocity*

---

## R

### Regen (Health Regeneration)

The automatic recovery of HP after a damage-free period. Regen starts **3 seconds** after the player last took damage. Once started, HP recovers at the REGEN_RATE per second until full (100 HP). Full regeneration from 1 HP to 100 HP takes approximately 2 seconds. Taking damage during regen immediately stops regeneration and resets the delay timer.

See also: *BASE_HP, REGEN_DELAY, REGEN_RATE, HP*

### Rollback

The process of reverting client-side prediction to match the authoritative server state when predictions differ. When the server sends a state update that contradicts the client's predicted state, the client rolls back to the server's state and re-simulates forward. This happens quickly and is designed to be imperceptible to the player under normal network conditions.

See also: *Prediction, lightyear, Lag Compensation*

---

## S

### Season

One Cycle of the Vimana. Corresponds to a content season in the live-service framework. A season is 8-10 weeks of active content followed by 1-2 weeks of off-season. Each season features a Pantheon host, new content (map, weapon, add-ons), a gameplay modifier, and a cosmetic progression track.

See also: *Cycle, Pantheon, Seasonal Framework*

### Silhouette

The outline shape of a player or object. In JuanSchott's art direction, silhouettes must be distinctive and readable at a glance. A player should be able to identify an opponent's species, current movement state, and equipped weapon from the silhouette alone. This is achieved through the low-poly high-finish art style and careful character design.

See also: *Low-Poly High-Finish, Species, Art Direction*

### Slide

A crouch-while-sprinting speed burst. The player initiates a slide by crouching while sprinting. The slide grants an initial speed boost (SLIDE_INITIAL_SPEED) followed by deceleration (SLIDE_DECEL) over a maximum duration (SLIDE_MAX_DURATION). During a slide, the player's hitbox is reduced (crouch height). The slide is loud and alerts nearby enemies.

See also: *SLIDE_INITIAL_SPEED, SLIDE_DECEL, SLIDE_MAX_DURATION, Crouch, Sprint*

### Splash Damage

Area damage from projectile weapons. When a projectile impacts a surface or player, it deals full damage at the impact point and reduced damage in a radius around it. Splash damage decreases linearly from the impact point to the splash radius edge.

See also: *Projectile, Damage Falloff*

### Species

One of four playable character types in JuanSchott. The four species are: Human (baseline), Cybersymbiote (human bonded with a machine organism), and two others to be defined in lore. Species are a visual and narrative distinction — they do not affect gameplay stats. All species have identical HP, movement speed, and hitbox dimensions.

See also: *Cybersymbiote, Silhouette*

### Spread

The angular variance of weapon fire. When a weapon fires, each shot deviates randomly within a cone defined by the spread value. Spread is lower when ADS and higher when hip-firing. Spread increases during sustained fire (for automatic weapons) and resets when the player stops firing.

See also: *ADS, Hip-Fire, SPREAD_HIP, SPREAD_ADS, Recoil*

---

## T

### TTK (Time to Kill)

How long it takes to eliminate a target from full HP (100). JuanSchott has a low TTK design — clean kills take 0.3-0.8 seconds. This means that a player who lands accurate shots (especially headshots) can eliminate an opponent very quickly. The low TTK emphasizes positioning, aim, and reaction time over sustained damage trades.

See also: *Headshot, Damage, HP, Weapon Balance*

---

## V

### Vimana

The satellite arena where JuanSchott takes place. Sanskrit for "divine flying machine." The Vimana is a cylindrical megastructure — a massive satellite that appears in Earth's orbit cyclically. Its interior surface contains the combat arenas (maps). The Vimana is operated by the Pantheon. It appears, issues the Invitation, hosts the competition (a season/Cycle), delivers judgment, and departs.

See also: *Cycle, Pantheon, Invitation, Hub, Zero-G, Season*

---

## W

### Wall Jump

Launching off a wall surface during or after wall contact. The wall jump propels the player laterally away from the wall with a slight upward component. Wall jumps do not require a preceding wall-run — a player can simply jump at a wall and bounce off. The wall jump's lateral force (WALL_JUMP_LATERAL) and vertical force (WALL_JUMP_VERTICAL) are tuned separately.

See also: *Wall Run, WALL_JUMP_LATERAL, WALL_JUMP_VERTICAL, Double Jump*

### Wall Run

Running along a vertical wall surface. A player moving toward a wall at sufficient speed can begin a wall-run, traveling along the wall for up to WALL_RUN_DURATION (1.5 seconds). During a wall-run, gravity is reduced (WALL_RUN_GRAVITY). The wall-run ends when the duration expires, the player jumps, or the wall surface ends. After a wall-run ends, the player can wall jump or fall.

See also: *Wall Jump, WALL_RUN_SPEED, WALL_RUN_DURATION, WALL_RUN_GRAVITY, Movement System*

### wgpu

Graphics API abstraction layer used by Bevy. wgpu provides a cross-platform interface to GPU APIs including Vulkan, DirectX 12, DirectX 11, Metal, and OpenGL/WebGL. JuanSchott renders through wgpu, which means the game can run on Windows, macOS, Linux, and (theoretically) web browsers without platform-specific rendering code.

See also: *Bevy, Rendering*

---

## Z

### Zero-G

Areas inside the Vimana with no gravity. Zero-g sections are interior map areas where gravity is disabled or near-zero. Players in zero-g can move in all three dimensions using their movement abilities. Zero-g sections are part of specific maps and are not present on every map. They provide a distinct gameplay experience compared to the standard gravity-based Hub surface.

See also: *Vimana, Hub, Gravity, Map*
