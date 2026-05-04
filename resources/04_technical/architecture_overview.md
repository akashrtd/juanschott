# Technical Architecture Overview

> **Document Version:** 1.0
> **Last Updated:** 2026-05-05
> **Status:** Active
> **Audience:** Contributors, technical leads, onboarding engineers

---

## Table of Contents

1. [Technology Stack](#technology-stack)
2. [Architecture Principles](#architecture-principles)
3. [High-Level System Architecture](#high-level-system-architecture)
4. [System Communication Pattern](#system-communication-pattern)
5. [Data Flow](#data-flow)
6. [Plugin Decomposition](#plugin-decomposition)
7. [Entity-Component Inventory](#entity-component-inventory)
8. [Resource Registry](#resource-registry)
9. [Event Catalog](#event-catalog)
10. [Asset Pipeline](#asset-pipeline)
11. [Rendering Pipeline](#rendering-pipeline)
12. [Network Architecture (Phase 3)](#network-architecture-phase-3)
13. [Build and Run](#build-and-run)
14. [Version Compatibility Table](#version-compatibility-table)
15. [See Also](#see-also)

---

## Technology Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| Engine | Bevy | 0.18 | ECS framework, rendering, asset loading, scheduling |
| Physics | Avian 3D | 0.6 | Collision detection, raycasting, kinematic bodies, spatial queries |
| Networking | lightyear | 0.26 | Server-authoritative multiplayer, client-side prediction, rollback |
| Renderer | wgpu | (via Bevy) | Vulkan / DX12 / DX11 / Metal / OpenGL backend abstraction |
| Language | Rust | latest stable | All game code, no unsafe outside FFI boundaries |
| Assets | glTF 2.0 | — | 3D models, maps, skeletal animations exported from Blender |
| License | MIT / Apache 2.0 | — | Dual-licensed, matching Bevy's own licensing model |

### Why These Choices

**Bevy 0.18** provides a mature ECS with automatic parallelization, a compile-time checked schedule graph, and first-class support for hot-reloadable assets. It is the only Rust engine that combines a true ECS architecture with a capable 3D renderer and an active plugin ecosystem.

**Avian 3D** is the successor to Rapier-based physics in the Bevy ecosystem. Version 0.6 targets Bevy 0.18 and exposes Bevy-native ECS components for rigid bodies, colliders, and spatial queries. It is lighter than a full physics engine and gives us direct control over the character controller.

**lightyear** is the leading networking crate for Bevy. It provides server-authoritative replication, client-side prediction with rollback, lag compensation, and input buffering. Version 0.26 targets Bevy 0.18. Designing for lightyear from day one — even in an offline prototype — avoids costly architectural rewrites when multiplayer is introduced.

**glTF 2.0** is the industry-standard interchange format. Blender exports it natively with full PBR material and skeletal animation support. Bevy's `GltfAssetLoader` handles mesh, material, animation, and scene instantiation in a single load operation.

---

## Architecture Principles

### 1. ECS-First Design

Everything is an entity with components. Systems operate on queries. There are no singleton managers, no God objects, and no global mutable statics. If a piece of data needs to exist once, it is a Bevy `Resource`. If it needs to exist per-instance, it is a `Component`. This rule is non-negotiable.

### 2. Plugin Architecture

Each major feature is a Bevy `Plugin`. Plugins register their own components, resources, events, systems, and asset loaders. Plugins can be added or removed independently — disabling a plugin must not break compilation. Dependencies between plugins are expressed through shared components and resources, never through direct function calls.

### 3. Single Crate Start

The project begins as a single Rust crate (`juanschott`). Monorepo crate splitting is deferred until the codebase grows large enough to justify separate compilation units. When splitting occurs, the project will adopt a Cargo workspace structure following Bevy's own evolution pattern: `crates/` directory with `core/`, `gameplay/`, `rendering/`, `networking/`, and `assets/` sub-crates.

### 4. Data-Driven Configuration

All tuning constants — movement speed, jump impulse, fire rate, damage values, camera sensitivity — are Bevy `Resource` structs loaded from RON or TOML files. No magic numbers appear in system logic. Every numeric constant must have a name, a home in a resource struct, and ideally a serialized default that can be overridden by a designer.

### 5. Network-Aware From Start

Even in Phase 1 (offline prototype), the architecture does not assume single-player. Components that will need replication are marked with `#[cfg_attr(feature = "networked", derive(Replicate))]`. Systems that process player input read from a buffered `InputBuffer` resource rather than calling `Res<ButtonInput<KeyCode>>` directly. This discipline ensures that the transition to lightyear-based multiplayer in Phase 3 is a feature-flag activation, not an architectural rewrite.

---

## High-Level System Architecture

The engine is organized into five horizontal layers. Each layer depends only on the layers below it. Data flows upward through the ECS; control flows downward through the Bevy schedule.

```
┌─────────────────────────────────────────────────┐
│                  UI Layer                        │
│          Bevy UI, HUD, menus, crosshair          │
├─────────────────────────────────────────────────┤
│              Rendering Layer                     │
│    Camera, materials, post-processing, view      │
│    models, environment map, skybox               │
├─────────────────────────────────────────────────┤
│             Simulation Layer                     │
│  Movement, weapons, health, add-ons, AI,         │
│  projectiles, hit detection                      │
├─────────────────────────────────────────────────┤
│              Physics Layer                       │
│  Avian 3D: colliders, raycasts, kinematic        │
│  bodies, spatial queries, character controller   │
├─────────────────────────────────────────────────┤
│              Input Layer                         │
│  Raw input capture, action mapping, buffering    │
├─────────────────────────────────────────────────┤
│              Asset Layer                         │
│  glTF loading, material assignment, collider     │
│  generation, texture atlases, audio samples      │
├─────────────────────────────────────────────────┤
│           Network Layer (Phase 3)                │
│  Server authority, client prediction,            │
│  replication, lag compensation                   │
└─────────────────────────────────────────────────┘
```

### Input Layer

The input layer captures raw hardware input and translates it into game actions before the simulation runs. It runs in `PreUpdate`.

| Subsystem | Responsibility |
|-----------|---------------|
| `InputCaptureSystem` | Reads `ButtonInput<KeyCode>` and `ButtonInput<MouseButton>`, maps to game actions |
| `InputBufferResource` | Stores per-frame action state (pressed, just_pressed, just_released) for deterministic consumption |
| `ActionMap` | Resource mapping raw inputs to logical actions (`MoveForward`, `Jump`, `Fire`, `Reload`, `Ability1`, etc.) |
| `CursorLockSystem` | Manages mouse grab/release state for first-person camera control |

The input buffer is consumed by simulation systems via `Res<InputBuffer>`. It is never cleared mid-frame; it persists until the next `PreUpdate` cycle. This allows multiple systems to read the same input state within a single frame without race conditions.

### Simulation Layer

The simulation layer contains all gameplay logic. It runs in the `Update` schedule. Systems here are pure data transformations: they read components and resources, mutate components, and emit events.

| Subsystem | Key Components | Key Resources | Key Events |
|-----------|---------------|---------------|------------|
| **Movement** | `PlayerController`, `Velocity`, `Grounded` | `MovementConfig` | — |
| **Weapons** | `ActiveWeapon`, `WeaponState`, `AmmoPool` | `WeaponsConfig` | `FireEvent`, `ReloadEvent` |
| **Health** | `Health`, `DamageMultipliers`, `Shield` | `HealthConfig` | `DamageEvent`, `DeathEvent` |
| **Add-Ons** | `AddOnSlots`, `ActiveAddOn`, `PassiveEffects` | `AddOnsConfig` | `AddOnActivatedEvent`, `AddOnExpiredEvent` |
| **Projectiles** | `Projectile`, `Lifetime`, `ProjectileDamage` | `ProjectileConfig` | `ProjectileHitEvent` |
| **Hit Detection** | `Hitbox`, `HitResult` | `HitConfig` | `HitEvent` |

The movement system implements a custom kinematic character controller on top of Avian 3D. It does not use Avian's dynamic rigid body simulation for the player capsule. Instead, it performs manual shape-casting against the world geometry, resolves collisions iteratively, and writes the final position back to the `Transform`. This gives full control over slide-along-wall behavior, step-up, and slope handling.

### Rendering Layer

The rendering layer configures cameras, materials, and post-processing effects. It runs across `PostUpdate` and Bevy's render graph.

| Subsystem | Responsibility |
|-----------|---------------|
| **Camera System** | Spawns and configures the first-person camera with correct FOV, near/far planes, and sensitivity |
| **View Model Layer** | Renders the weapon/hands model on a separate camera with a different FOV and near-clip plane to prevent clipping |
| **Material System** | Applies custom flat-color or cel-shaded materials; manages material asset handles |
| **Post-Processing** | Configures bloom, tonemapping (AgX), FXAA, and color grading via Bevy's `Camera` component settings |
| **Environment** | Loads and applies HDRI skybox, configures ambient lighting and directional light |

The view model layer uses a second camera with `RenderLayer` filtering. The main camera renders world geometry on layers 0–3. The view model camera renders only layer 4. This prevents the view model from intersecting with world geometry and allows independent FOV control.

### Network Layer (Phase 3)

The network layer is architecturally prepared from Phase 1 but only activated in Phase 3. It wraps lightyear's transport, replication, and prediction systems.

| Role | Responsibility |
|------|---------------|
| **Server** | Authoritative simulation. Runs full physics and gameplay. Handles hit registration. Replicates component state to clients. |
| **Client** | Sends input to server. Runs local prediction of owned entities. Interpolates remote entities. Applies server corrections via rollback. |
| **Transport** | UDP via lightyear's configurable transport. Configurable tick rate (default: 64 Hz server, client sends input each frame). |

Components that require replication are annotated with lightyear's `#[derive(Replicate)]`. The replication rules define which components are predicted (player movement), interpolated (other players), or authoritative-only (server-side hit results).

### Asset Layer

The asset layer handles loading, processing, and hot-reloading of all game content.

| Subsystem | Responsibility |
|-----------|---------------|
| **glTF Loader** | Loads `.glb`/`.gltf` files via Bevy's built-in `GltfAssetLoader`. Extracts meshes, materials, animations, and scenes. |
| **Collider Generator** | Processes loaded meshes into Avian `Collider` components. Uses convex decomposition for complex geometry, convex hulls for props. |
| **Material Assignment** | Maps mesh primitives to game-specific materials (flat color, unlit, custom shader). Overrides glTF PBR when target aesthetic requires it. |
| **Hot Reload** | Bevy's `AssetServer` watches for file changes in development mode. Modified assets are reloaded without restarting the game. |

---

## System Communication Pattern

All inter-system communication flows through Bevy's ECS. No system directly calls another system's functions. This section defines the four communication mechanisms and when to use each.

| Mechanism | Use Case | Lifetime | Example |
|-----------|----------|----------|---------|
| **Components** | Persistent per-entity state | Until entity despawned | `Health`, `Velocity`, `WeaponState` |
| **Resources** | Singleton or shared configuration | Until app exits or overwritten | `MovementConfig`, `InputBuffer`, `Time` |
| **Events** | One-shot, point-to-point or broadcast | Consumed after one read cycle | `DamageEvent`, `DeathEvent`, `FireEvent` |
| **Queries** | Continuous read access to component data | Per-system, per-frame | `Query<&Transform, With<Player>>` |

### Communication Rules

1. **Systems must not hold references to entities across frames.** Use `Entity` identifiers as weak references. Always validate with `Query::get()` before access.
2. **Events are fire-and-forget.** An event is emitted by one system and consumed by any number of readers in the same or subsequent frame. Use `EventWriter` and `EventReader`.
3. **Resources are the configuration layer.** A system reads `Res<MovementConfig>` to get speed values. It never hardcodes `4.0`.
4. **Commands are deferred mutations.** Spawning, despawning, and component insertion/removal use `Commands` to avoid borrow conflicts within the same schedule stage.

---

## Data Flow

This section traces the path of a single frame from input capture to render submission.

```
Frame Start
  │
  ▼
PreUpdate
  ├── InputCaptureSystem: reads raw input → writes InputBuffer resource
  └── NetworkInputSystem (Phase 3): receives remote inputs → writes InputBuffer
  │
  ▼
Update (Physics step: Avian 3D)
  ├── CharacterControllerSystem: reads InputBuffer + MovementConfig
  │     → performs shape-casts → writes Transform, Velocity
  ├── WeaponFireSystem: reads InputBuffer + ActiveWeapon + WeaponsConfig
  │     → emits FireEvent, spawns Projectile entities
  ├── ProjectileSystem: reads Projectile + Velocity + Lifetime
  │     → advances position, decrements timer, emits ProjectileHitEvent
  ├── DamageSystem: reads DamageEvent + Health + DamageMultipliers
  │     → writes Health, emits DeathEvent if health ≤ 0
  ├── AddOnSystem: reads ActiveAddOn + PassiveEffects + InputBuffer
  │     → applies effects, emits AddOnActivatedEvent
  └── HitDetectionSystem: reads ProjectileHitEvent + Hitbox
        → emits HitEvent
  │
  ▼
PostUpdate
  ├── Transform propagation (Bevy built-in)
  ├── Camera follow system: syncs camera Transform to player Transform
  ├── View model animation: updates weapon animation state
  └── UI sync: reads Health, AmmoPool → updates HUD text/sprites
  │
  ▼
Render (Bevy render graph)
  ├── Extract phase: copies relevant ECS data into render world
  ├── Prepare phase: uploads GPU buffers, binds materials
  ├── Queue phase: submits draw calls per render pass
  └── Present: swaps framebuffers
  │
  ▼
Frame End
```

---

## Plugin Decomposition

Each row is a Bevy `Plugin` struct. The dependency column lists other plugins whose components or resources this plugin reads.

| Plugin | Registers | Depends On | Schedule Label |
|--------|-----------|-----------|---------------|
| `InputPlugin` | `InputBuffer`, `ActionMap`, `InputCaptureSystem` | None | `PreUpdate` |
| `MovementPlugin` | `PlayerController`, `MovementConfig`, `CharacterControllerSystem` | `InputPlugin`, `PhysicsPlugin` (Avian) | `Update` |
| `CombatPlugin` | `ActiveWeapon`, `AmmoPool`, `WeaponState`, weapon systems | `InputPlugin`, `MovementPlugin` | `Update` |
| `HealthPlugin` | `Health`, `Shield`, `DamageMultipliers`, damage/death systems | `CombatPlugin` | `Update` |
| `AddOnPlugin` | `AddOnSlots`, `ActiveAddOn`, `PassiveEffects`, add-on systems | `HealthPlugin`, `CombatPlugin` | `Update` |
| `ProjectilePlugin` | `Projectile`, `Lifetime`, projectile systems | `CombatPlugin`, `PhysicsPlugin` (Avian) | `Update` |
| `CameraPlugin` | `FpsCamera`, camera follow system | `MovementPlugin` | `PostUpdate` |
| `RenderingPlugin` | Material setup, post-processing config, view model camera | `CameraPlugin` | `PostUpdate` |
| `UiPlugin` | HUD, menus, crosshair, UI sync systems | `HealthPlugin`, `CombatPlugin` | `PostUpdate` |
| `AssetPlugin` | Asset loading, collider generation, material assignment | None | `Startup` + `Update` |
| `NetworkPlugin` (Phase 3) | lightyear client/server, replication rules, prediction | All gameplay plugins | All stages |

---

## Entity-Component Inventory

### Player Entity

| Component | Type | Purpose |
|-----------|------|---------|
| `Transform` | Bevy built-in | World position, rotation, scale |
| `Visibility` | Bevy built-in | Render visibility |
| `PlayerController` | Custom | Marks entity as player, stores controller config reference |
| `Velocity` | Avian | Linear and angular velocity |
| `Collider` | Avian | Capsule collider for player body |
| `Grounded` | Custom | Whether player is on a surface |
| `Health` | Custom | Current and max health points |
| `Shield` | Custom | Current and max shield points |
| `DamageMultipliers` | Custom | Per-damage-type multipliers |
| `ActiveWeapon` | Custom | Currently equipped weapon entity reference |
| `AmmoPool` | Custom | Ammo counts per weapon type |
| `AddOnSlots` | Custom | Active and passive add-on slots |

### Weapon Entity

| Component | Type | Purpose |
|-----------|------|---------|
| `Transform` | Bevy built-in | Position relative to player or world |
| `WeaponState` | Custom | Ready, firing, reloading, swapped |
| `FireRateTimer` | Custom | Cooldown between shots |
| `ReloadTimer` | Custom | Duration of reload animation |
| `DamageProfile` | Custom | Base damage, falloff range, damage type |

### Projectile Entity

| Component | Type | Purpose |
|-----------|------|---------|
| `Transform` | Bevy built-in | World position |
| `Velocity` | Avian or custom | Flight direction and speed |
| `Projectile` | Custom | Owner entity, damage profile, projectile type |
| `Lifetime` | Custom | Frames or seconds until auto-despawn |
| `ProjectileDamage` | Custom | Damage value carried from weapon at time of fire |

### Map Entity (spawned from glTF scene)

| Component | Type | Purpose |
|-----------|------|---------|
| `Transform` | Bevy built-in | World origin of map geometry |
| `Mesh3d` | Bevy | Mesh handle from glTF |
| `MeshMaterial3d` | Bevy | Material handle |
| `Collider` | Avian | Generated from mesh geometry |
| `RigidBody::Static` | Avian | Immovable world geometry |

---

## Resource Registry

Resources are singleton data stores. This table catalogs every game-defined resource.

| Resource | Type | Purpose | Serialized |
|----------|------|---------|-----------|
| `InputBuffer` | `Struct` | Per-frame action states | No |
| `ActionMap` | `Struct` | Key-to-action bindings | Yes (RON) |
| `MovementConfig` | `Struct` | Walk speed, sprint speed, jump impulse, gravity, friction | Yes (RON) |
| `WeaponsConfig` | `HashMap<WeaponType, WeaponParams>` | Per-weapon fire rate, damage, magazine size, reload time | Yes (RON) |
| `HealthConfig` | `Struct` | Max health, max shield, regen delay, regen rate | Yes (RON) |
| `AddOnsConfig` | `HashMap<AddOnType, AddOnParams>` | Per-add-on duration, cooldown, effect magnitudes | Yes (RON) |
| `ProjectileConfig` | `Struct` | Default speed, max lifetime, max bounces | Yes (RON) |
| `HitConfig` | `Struct` | Hitbox dimensions, headshot multiplier, friendly fire toggle | Yes (RON) |

All serialized resources are loaded from the `assets/config/` directory via Bevy's `AssetServer` with custom RON or TOML asset loaders. In development mode, file changes trigger automatic reload via Bevy's hot-reload infrastructure.

---

## Event Catalog

Events are one-shot messages. This table catalogs every game-defined event.

| Event | Emitted By | Consumed By | Payload |
|-------|-----------|-------------|---------|
| `FireEvent` | `WeaponFireSystem` | `ProjectileSystem`, `AudioSystem` | Entity (shooter), weapon type, fire origin, direction |
| `ReloadEvent` | `WeaponFireSystem` | `AudioSystem`, `AnimationSystem` | Entity (reloader), weapon type |
| `ProjectileHitEvent` | `ProjectileSystem` | `HitDetectionSystem` | Entity (projectile), Entity (hit target), hit point, normal |
| `DamageEvent` | `HitDetectionSystem` | `DamageSystem`, `UiPlugin` | Entity (target), damage amount, damage type, source entity |
| `DeathEvent` | `DamageSystem` | `GameModeSystem`, `UiPlugin` | Entity (killed), Entity (killer), damage type |
| `AddOnActivatedEvent` | `AddOnSystem` | `AudioSystem`, `FxSystem` | Entity (player), add-on type |
| `AddOnExpiredEvent` | `AddOnSystem` | `CombatPlugin`, `HealthPlugin` | Entity (player), add-on type |
| `HitEvent` | `HitDetectionSystem` | `AudioSystem`, `FxSystem`, `DamageSystem` | Entity (attacker), Entity (victim), hit point, hitbox type |

---

## Asset Pipeline

### Source to Runtime Flow

```
Blender (.blend)
  │
  ├── Export as glTF 2.0 (.glb)
  │     ├── Meshes (geometry)
  │     ├── Materials (base color, roughness, metallic)
  │     ├── Skeletons + Animations
  │     └── Scene hierarchy (empty objects as spawn points)
  │
  ▼
Bevy AssetServer (load at startup or on demand)
  │
  ├── GltfAssetLoader → spawns Mesh3d + MeshMaterial3d components
  ├── ColliderGenerator → processes mesh → spawns Avian Collider
  ├── MaterialOverride → replaces glTF materials with game materials
  └── AnimationPlayer → plays skeletal animations on weapon view models
```

### Asset Directory Layout

```
assets/
├── config/              # Serialized game parameters (RON/TOML)
│   ├── movement.ron
│   ├── weapons.ron
│   ├── health.ron
│   └── addons.ron
├── models/              # glTF files
│   ├── weapons/
│   ├── characters/
│   └── props/
├── maps/                # Level geometry (glTF scenes)
│   ├── arena_01.glb
│   └── arena_02.glb
├── textures/            # Raw textures (PNG, HDR)
├── materials/           # Custom material definitions
└── audio/               # Sound effects (WAV/OGG)
```

---

## Rendering Pipeline

The rendering pipeline leverages Bevy's built-in render graph with custom configuration.

1. **Main Camera** — FOV 90°, near 0.01m, far 1000m. Renders layers 0–3 (world geometry, props, projectiles, effects).
2. **View Model Camera** — FOV 70°, near 0.01m, far 5m. Renders layer 4 only (weapon model, hands). Clears depth before drawing to prevent z-fighting with world geometry.
3. **Post-Processing Chain** — AgX tonemapping → Bloom (threshold 0.8, intensity 0.2) → FXAA → Color grading (slight desaturation for target aesthetic).
4. **Environment** — HDRI cubemap loaded from assets, configured as ambient light. Single directional light for sun.

---

## Network Architecture (Phase 3)

> This section describes the target architecture for multiplayer. It is not implemented in Phase 1 or Phase 2.

### Topology

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│   Client A   │◄───────►│    Server    │◄───────►│   Client B   │
│              │  UDP    │              │  UDP    │              │
│  - Send      │         │  - Simulate  │         │  - Send      │
│    inputs    │         │  - Validate  │         │    inputs    │
│  - Predict   │         │  - Hit reg   │         │  - Predict   │
│  - Interp    │         │  - Replicate │         │  - Interp    │
│    remote    │         │  - Authority │         │    remote    │
└──────────────┘         └──────────────┘         └──────────────┘
```

### Replication Model

| Component | Replication Mode | Authority | Notes |
|-----------|-----------------|-----------|-------|
| `Transform` (player) | Predicted | Server | Client predicts; server corrects |
| `Transform` (remote player) | Interpolated | Server | Smoothed between snapshots |
| `Health` | Replicated | Server | Client displays only |
| `WeaponState` | Predicted | Server | Client predicts fire/reload |
| `InputBuffer` | Input | Client | Sent to server each frame |
| `Projectile` | Replicated | Server | Server spawns, client renders |

### Tick Rate

- Server simulation: **64 Hz** (fixed timestep)
- Client input send: **every frame** (variable, buffered)
- Client render: **uncapped** (vsync or max FPS)
- Snapshot send: **20 Hz** (replication to clients)

---

## Build and Run

```bash
# Clone and build
git clone https://github.com/[org]/juanschott.git
cd juanschott
cargo run --release

# Run with specific features
cargo run --release --features "3d"

# Run tests
cargo test

# Run with fast compiles (development)
cargo run --features "bevy/dynamic_linking"

# Build for release distribution
cargo build --release --features "3d"
```

### Development Workflow

1. Run with `dynamic_linking` for sub-second incremental compiles after the first build.
2. Asset hot-reload is enabled by default in debug mode. Modify any file under `assets/` and see changes in real-time.
3. Use `RUST_LOG=debug cargo run` for verbose logging. Use `RUST_LOG=juanschott=trace` for game-specific trace output.

---

## Version Compatibility Table

| Component | Version | Bevy Compatible | Notes |
|-----------|---------|----------------|-------|
| Bevy | 0.18 | — | Foundation |
| Avian 3D | 0.6 | 0.18 | Physics and spatial queries |
| lightyear | 0.26 | 0.18 | Networking (Phase 3) |
| Rust MSRV | latest stable | — | Pin to Bevy's MSRV if it differs |
| Blender | 4.x+ | — | glTF 2.0 export pipeline |

### Upgrade Strategy

When Bevy releases a new breaking version (0.19, etc.):
1. Check Avian and lightyear release notes for compatible versions.
2. Run Bevy's migration guide.
3. Update `Cargo.toml` versions in lockstep.
4. Run full test suite before merging.

---

## See Also

| Document | Path | Description |
|----------|------|-------------|
| Crate Structure | [crate_structure.md](./crate_structure.md) | Workspace layout, module boundaries, dependency graph |
| Systems Design | [systems_design.md](./systems_design.md) | Detailed system specifications, query patterns, schedule ordering |
| Networking | [networking.md](./networking.md) | lightyear integration, replication rules, prediction strategy |
| Rendering Pipeline | [rendering_pipeline.md](./rendering_pipeline.md) | Camera setup, materials, shaders, post-processing configuration |
