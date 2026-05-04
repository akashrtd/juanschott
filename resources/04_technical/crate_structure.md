# Crate Structure

> **Document version:** 1.0.0
> **Last updated:** 2026-05-05
> **Status:** Authoritative reference

---

## 1. Overview

JuanSchott is built as a single Rust crate using the Bevy engine. The module tree is organized so that every logical domain — player, weapons, health, add-ons, map loading, networking, rendering, UI, and game modes — is isolated behind its own `mod.rs` and exposed through a Bevy `Plugin`. This discipline serves two purposes:

1. **Immediate clarity.** Every system, component, and resource has exactly one home. Navigating the codebase is predictable; if you need the wall-run logic, you open `src/player/wall_run.rs`.

2. **Future workspace splitting.** When compilation times or team ownership boundaries demand it, any module directory can be promoted to its own crate in a Cargo workspace with minimal refactoring. The plugin-per-module pattern already defines the public API boundary each future crate would expose.

No module reaches into another module's private items. Cross-module communication happens exclusively through Bevy's ECS: components attached to entities, events, and shared resources defined in `constants.rs`. This is the single most important architectural rule in the crate.

---

## 2. Full File Tree

```
juanschott/
├── Cargo.toml                    # Project manifest, dependencies, features, profiles.
├── README.md                     # Project overview, quickstart, build instructions.
├── CONTRIBUTING.md               # Contribution guidelines, code style, PR process.
├── src/
│   ├── main.rs                   # App entry point. Plugin assembly. System scheduling.
│   ├── lib.rs                    # Library root. Re-exports. Plugin definitions.
│   ├── constants.rs              # All tuning constants as Bevy resources.
│   ├── player/
│   │   ├── mod.rs                # Player module. PlayerPlugin definition.
│   │   ├── components.rs         # Player, PlayerInput, MovementState, etc.
│   │   ├── spawn.rs              # Player entity spawning system.
│   │   ├── input.rs              # Input capture and mapping system.
│   │   ├── look.rs               # Mouse look (pitch/yaw) system.
│   │   ├── movement.rs           # Walk, sprint, gravity, collision system.
│   │   ├── jump.rs               # Jump, double-jump, ground detection systems.
│   │   ├── slide.rs              # Slide mechanic system.
│   │   ├── wall_run.rs           # Wall detection, wall-run, wall-jump (Phase 2).
│   │   └── camera.rs             # First-person camera sync, view model layer.
│   ├── weapons/
│   │   ├── mod.rs                # Weapons module. WeaponsPlugin definition.
│   │   ├── components.rs         # Weapon, Ammo, FireMode, WeaponType components.
│   │   ├── hitscan.rs            # Hitscan fire system (raycast).
│   │   ├── projectile.rs         # Projectile spawn and movement systems.
│   │   ├── beam.rs               # Beam weapon tick-damage system.
│   │   ├── reload.rs             # Reload timer and state management.
│   │   └── viewmodel.rs          # First-person weapon rendering.
│   ├── health/
│   │   ├── mod.rs                # Health module. HealthPlugin definition.
│   │   ├── components.rs         # Health, DamageEvent, RegenTimer components.
│   │   ├── damage.rs             # Damage application system.
│   │   ├── regen.rs              # Health regeneration system.
│   │   └── death.rs              # Death detection and respawn system.
│   ├── addons/
│   │   ├── mod.rs                # Add-ons module. AddonsPlugin definition.
│   │   ├── components.rs         # AddOnSlot, AddOnType, ActiveEffect components.
│   │   ├── equip.rs              # Add-on equipment and slot management.
│   │   ├── damage_addons.rs      # Damage category add-on effects.
│   │   ├── movement_addons.rs    # Movement category add-on effects.
│   │   ├── health_addons.rs      # Health category add-on effects.
│   │   └── perk_addons.rs        # Perk category add-on effects.
│   ├── map/
│   │   ├── mod.rs                # Map module. MapPlugin definition.
│   │   ├── loader.rs             # glTF scene loading and spawning.
│   │   ├── colliders.rs          # Collider generation from mesh geometry.
│   │   └── spawn_points.rs       # Player spawn point locations.
│   ├── network/                  # Phase 3
│   │   ├── mod.rs                # Network module. NetworkPlugin definition.
│   │   ├── shared.rs             # Shared protocol, channels, component registration.
│   │   ├── server.rs             # Server-specific systems and configuration.
│   │   ├── client.rs             # Client-specific systems, prediction setup.
│   │   └── input.rs              # Networked input handling (lightyear integration).
│   ├── render/                   # Phase 4
│   │   ├── mod.rs                # Render module. RenderPlugin definition.
│   │   ├── toon_material.rs      # Custom flat-color/stylized material (WGSL).
│   │   ├── outlines.rs           # Outline post-process effect.
│   │   └── post_processing.rs    # Custom post-processing chain.
│   ├── ui/
│   │   ├── mod.rs                # UI module. UIPlugin definition.
│   │   ├── hud.rs                # In-game HUD elements.
│   │   ├── crosshair.rs          # Dynamic crosshair system.
│   │   ├── menus.rs              # Main menu, loadout, settings screens.
│   │   └── score_screen.rs       # End-of-match score display.
│   └── modes/
│       ├── mod.rs                # Modes module. ModesPlugin definition.
│       ├── duel.rs               # 1v1 duel mode rules and scoring.
│       ├── skirmish.rs           # 3v3 skirmish mode rules and scoring.
│       └── clash.rs              # 6v6 clash mode rules and scoring.
├── assets/
│   ├── maps/
│   │   └── test_arena.glb        # Phase 1 test map.
│   ├── models/
│   │   ├── player/               # Player model variants (per species).
│   │   └── weapons/              # Weapon models.
│   ├── shaders/
│   │   ├── toon.wgsl             # Custom toon/flat-color shader.
│   │   └── outline.wgsl          # Outline post-process shader.
│   └── ui/
│       └── fonts/                # UI fonts.
├── blender/
│   ├── test_arena.blend          # Source Blender file for test map.
│   └── export_presets/           # Blender glTF export settings documentation.
└── resources/                    # This documentation folder (not in build).
```

---

## 3. Entry Points

### 3.1 `main.rs`

`main.rs` is the binary entry point. Its responsibilities are narrow and deliberate:

1. Create the `App` with `MinimalPlugins` and `DefaultPlugins` (with the 3D render pipeline).
2. Add all game plugins in dependency order: `MapPlugin`, `ConstantsPlugin`, `PlayerPlugin`, `HealthPlugin`, `WeaponsPlugin`, `AddonsPlugin`, `ModesPlugin`, `UIPlugin`, and conditionally `NetworkPlugin` / `RenderPlugin`.
3. Set the initial game state (e.g. `AppState::MainMenu`).
4. Call `app.run()`.

`main.rs` never contains systems, components, or logic. It is pure assembly. If a line of code in `main.rs` does not register a plugin or configure the app builder, it probably does not belong there.

### 3.2 `lib.rs`

`lib.rs` is the library root and the public API surface of the crate. It:

1. Declares every module with `pub mod`.
2. Re-exports the plugin struct of every module so that `main.rs` (and future integration tests) can reference them without deep path imports.
3. Defines application-level enums and states: `AppState`, `GamePhase`, `Team`.
4. Contains no systems or component definitions — only type aliases, re-exports, and state enums.

The re-export pattern:

```rust
pub use player::PlayerPlugin;
pub use weapons::WeaponsPlugin;
pub use health::HealthPlugin;
pub use addons::AddonsPlugin;
pub use map::MapPlugin;
#[cfg(feature = "networked")]
pub use network::NetworkPlugin;
pub use render::RenderPlugin;
pub use ui::UIPlugin;
pub use modes::ModesPlugin;
```

### 3.3 `constants.rs`

Every tunable number in the game lives in `constants.rs` as a Bevy `Resource` struct. This includes movement speeds, jump heights, gravity, weapon damage values, health regeneration rates, and add-on effect magnitudes. Systems read these resources; they never hardcode values.

Rationale: designers and playtesters can tweak balance by editing a single file. When workspace splitting occurs, `constants.rs` becomes the first shared crate.

```rust
#[derive(Resource, Clone, Copy)]
pub struct MovementConstants {
    pub walk_speed: f32,
    pub sprint_speed: f32,
    pub slide_speed: f32,
    pub gravity: f32,
    pub jump_force: f32,
    pub double_jump_force: f32,
    pub ground_detect_distance: f32,
}
```

---

## 4. Module Responsibilities

### 4.1 Player (`src/player/`)

| Aspect | Detail |
|--------|--------|
| **Owns** | `Player`, `PlayerInput`, `MovementState`, `LookAngles`, `Grounded`, `Sliding`, `WallRunning` components; all player-related systems; the first-person camera entity. |
| **Depends on** | `constants` (movement tuning), `map` (collider queries for ground/wall detection via Avian3D). |
| **Exports** | `Player` marker component, `PlayerInput` resource, `MovementState` enum, `PlayerPlugin`. |
| **Plugin registers** | Input capture (`input.rs`), look (`look.rs`), movement (`movement.rs`), jump (`jump.rs`), slide (`slide.rs`), wall-run (`wall_run.rs` — Phase 2), camera sync (`camera.rs`), player spawn (`spawn.rs`). |

The player module is the largest module in Phase 1. Each movement mechanic is isolated in its own file so that mechanics can be enabled, disabled, or rewritten independently. `camera.rs` synchronises a separate camera entity with the player's transform and applies view-model bobbing, tilt, and field-of-view shifts.

**System ordering within PlayerPlugin:**

```
input_capture → look → movement → jump → slide → wall_run → camera_sync
```

All systems run in `PostUpdate` except `input_capture`, which runs in `PreUpdate` to ensure the latest frame's input is available before any consumption.

### 4.2 Weapons (`src/weapons/`)

| Aspect | Detail |
|--------|--------|
| **Owns** | `Weapon`, `Ammo`, `FireMode`, `WeaponType`, `ActiveWeapon`, `ReloadTimer` components; hitscan, projectile, and beam fire systems; reload logic; view-model rendering. |
| **Depends on** | `player` (camera transform for aiming direction, `PlayerInput` for fire/alt-fire/trigger), `health` (`DamageEvent` writer), `constants` (damage, fire rate, recoil). |
| **Exports** | `Weapon`, `ActiveWeapon`, `WeaponType`, `DamageEvent` (re-exported from health if needed), `WeaponsPlugin`. |
| **Plugin registers** | Hitscan fire, projectile fire, beam tick, reload timer, view-model sync. |

Three fire system variants exist because JuanSchott supports distinct weapon archetypes:

- **Hitscan** (`hitscan.rs`): Instant raycast. Used for rifles and pistols. Casts from the camera through the crosshair into the scene using Avian3D's `SpatialQuery`.
- **Projectile** (`projectile.rs`): Spawns a separate entity with a `Projectile` component and a velocity. Movement is handled by Avian3D physics. Used for rocket-type weapons.
- **Beam** (`beam.rs`): Continuous damage-over-time while the trigger is held. Uses a raycast per frame but applies fractional damage. Used for energy weapons.

`viewmodel.rs` manages the first-person weapon mesh: bobbing, recoil animation offsets, muzzle flash attachment points, and ADS (aim-down-sights) interpolation.

### 4.3 Health (`src/health/`)

| Aspect | Detail |
|--------|--------|
| **Owns** | `Health`, `MaxHealth`, `DamageEvent` (Bevy event), `RegenTimer`, `Alive` components; damage application, regeneration, and death systems. |
| **Depends on** | `player` (for respawn location, `Player` marker), `constants` (base HP, regen delay, regen rate). |
| **Exports** | `Health`, `DamageEvent`, `MaxHealth`, `Alive`, `HealthPlugin`. |
| **Plugin registers** | Damage application (`damage.rs`), regeneration (`regen.rs`), death detection (`death.rs`). |

`DamageEvent` is the universal damage interface. Any system — weapon hitscan, projectile impact, beam tick, environmental hazard, or future ability — writes a `DamageEvent`. The damage application system reads these events, applies add-on modifiers (by querying the target's add-on components), and clamps health. This indirection means new damage sources never need to know about add-ons or health internals.

```rust
pub struct DamageEvent {
    pub target: Entity,
    pub amount: f32,
    pub source: Option<Entity>,
    pub damage_type: DamageType,
}
```

### 4.4 Add-ons (`src/addons/`)

| Aspect | Detail |
|--------|--------|
| **Owns** | `AddOnSlot`, `AddOnType`, `ActiveEffect`, `AddOnInventory` components; equip logic; per-category effect systems. |
| **Depends on** | `player` (movement modification), `weapons` (damage modification), `health` (health modification), `constants` (add-on magnitudes). |
| **Exports** | `AddOnSlot`, `AddOnType`, `ActiveEffect`, `AddonsPlugin`. |
| **Plugin registers** | Equip system (`equip.rs`), damage add-on effects, movement add-on effects, health add-on effects, perk add-on effects. |

Add-ons are the loadout customization system. Each player has four `AddOnSlot` components (one per category: damage, movement, health, perk). The equip system validates and attaches an add-on entity to a slot. Effect systems read the active add-on and apply passive modifiers during their respective domain's update cycle.

The add-on module is a cross-cutting concern: it reads from player, weapons, and health but does not contain game logic itself — only modifiers. This keeps the core modules testable in isolation without add-ons.

### 4.5 Map (`src/map/`)

| Aspect | Detail |
|--------|--------|
| **Owns** | `MapHandle`, `SpawnPoint` components; glTF loading, collider generation, spawn point extraction systems. |
| **Depends on** | `constants` (physics material properties). |
| **Exports** | `SpawnPoint`, `MapPlugin`. |
| **Plugin registers** | Map loader (`loader.rs`), collider generator (`colliders.rs`), spawn point reader (`spawn_points.rs`). |

`loader.rs` uses Bevy's `AssetServer` to load a `.glb` file, spawns it into the scene, and triggers collider generation. `colliders.rs` iterates over loaded mesh entities and attaches Avian3D `Collider` components — either trivially from convex decomposition or from manually authored collision meshes tagged in the Blender file with a naming convention (e.g. `col_` prefix).

`spawn_points.rs` reads named empty objects from the glTF scene whose names begin with `spawn_` and creates `SpawnPoint` components for use by the player spawn system.

### 4.6 Network (`src/network/`) — Phase 3

| Aspect | Detail |
|--------|--------|
| **Owns** | `NetworkConfig` resource, `ClientState`, `ServerState` components; lightyear protocol definition, channel setup, prediction/rollback config. |
| **Depends on** | Every game module (replicates all game state). |
| **Exports** | `NetworkPlugin` (behind `#[cfg(feature = "networked")]`). |
| **Plugin registers** | Shared protocol registration, server authority systems, client prediction and interpolation, networked input relay. |

The network module is gated behind the `networked` feature flag. In Phase 1 and 2, it compiles to a no-op. `shared.rs` defines the lightyear protocol: which components are replicated, which channels exist (reliable, unreliable, tick-based), and component registration macros. `server.rs` runs authority systems that validate moves and broadcast state. `client.rs` sets up prediction with rollback using lightyear's built-in rollback framework. `input.rs` captures input on the client and serialises it for transmission.

### 4.7 Render (`src/render/`) — Phase 4

| Aspect | Detail |
|--------|--------|
| **Owns** | `ToonMaterial` (custom Bevy material), outline post-process node, custom render graph modifications. |
| **Depends on** | Bevy render APIs; reads `Player` and `Weapon` transforms for view-model layering. |
| **Exports** | `ToonMaterial`, `RenderPlugin`. |
| **Plugin registers** | Toon material asset loader, outline render pass, post-processing chain. |

`toon_material.rs` implements Bevy's `Material` trait with a custom WGSL shader that produces flat-shaded, hard-edge lighting — the stylised look described in the art direction. `outlines.rs` adds an inverted-hull or screen-space outline pass. `post_processing.rs` chains any additional effects (bloom, colour grading overrides, vignette).

### 4.8 UI (`src/ui/`)

| Aspect | Detail |
|--------|--------|
| **Owns** | All Bevy UI node trees, UI state resources, HUD layout data. |
| **Depends on** | `health` (HP display), `weapons` (ammo display), `addons` (loadout display), `modes` (score display), `player` (crosshair aim state). |
| **Exports** | `UIPlugin`. |
| **Plugin registers** | HUD update, crosshair update, menu systems, score screen. |

The UI module is a pure consumer. It reads ECS state and writes to Bevy's UI framework. It never modifies game components. This separation means UI can be redesigned without touching any game logic.

`crosshair.rs` is notable: it dynamically adjusts crosshair spread based on player movement state (walking, sprinting, sliding, airborne) and weapon recoil recovery, providing visual feedback for accuracy.

### 4.9 Modes (`src/modes/`)

| Aspect | Detail |
|--------|--------|
| **Owns** | `ActiveMode` resource, `Score`, `MatchTimer` components; per-mode rulesets and win conditions. |
| **Depends on** | `health` (death events), `player` (player identity and team). |
| **Exports** | `ActiveMode`, `ModesPlugin`. |
| **Plugin registers** | Duel rules, skirmish rules, clash rules. |

Each mode file implements the same interface pattern: a system that reads death events, updates scores, checks win conditions, and transitions `AppState` when the match ends. Modes are selected at match start and the active mode resource controls which rule system runs.

---

## 5. Dependency Graph

```
                  ┌──────────┐
                  │constants │
                  └────┬─────┘
                       │
         ┌─────────────┼─────────────┐
         │             │             │
    ┌────▼────┐   ┌────▼────┐       │
    │  map    │   │ player  │◄──────┤
    └─────────┘   └──┬─┬─┬─┘       │
                     │ │ │         │
          ┌──────────┘ │ └──────┐  │
          │            │        │  │
    ┌─────▼───┐  ┌─────▼───┐  ┌▼──┴─────┐
    │ weapons │  │  health │  │  addons  │
    └────┬────┘  └────┬────┘  └────┬─────┘
         │            │            │
         └─────┬──────┘────────────┘
               │
         ┌─────▼──────┐
         │   modes    │
         └─────┬──────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐ ┌───▼───┐ ┌────▼──────┐
│  ui   │ │render │ │ network   │
│       │ │(Ph.4) │ │ (Ph.3)    │
└───────┘ └───────┘ └───────────┘
```

**Edge descriptions:**

| From | To | Nature of dependency |
|------|----|----------------------|
| `player` | `constants` | Reads movement speed, gravity, jump force resources. |
| `player` | `map` | Queries Avian3D spatial query API for ground/wall detection against map colliders. |
| `weapons` | `player` | Reads `PlayerInput` for fire trigger; reads camera `Transform` for aim direction. |
| `weapons` | `health` | Sends `DamageEvent` on hit. |
| `health` | `player` | Reads `Player` marker for respawn; queries `Transform` for spawn positioning. |
| `health` | `addons` | Queries active add-ons to modify incoming damage or regeneration rate. |
| `addons` | `player` | Modifies movement constants at runtime via `MovementAddOn` effect. |
| `addons` | `weapons` | Modifies damage output via `DamageAddOn` effect. |
| `addons` | `health` | Modifies max HP or regen via `HealthAddOn` effect. |
| `modes` | `health` | Reads death events for scoring. |
| `modes` | `player` | Reads player identity and team for scoreboard. |
| `ui` | `health`, `weapons`, `addons`, `modes` | Reads components for display. Never writes game state. |
| `network` | all | Replicates component state; sends/receives input. |
| `render` | all | Reads transforms and materials for rendering pipeline. |

**Cycle note:** The `addons` module depends on `player`, `weapons`, and `health`, while `health` depends on `addons`. This is not a circular dependency at the code level because `health` only *queries* add-on components on entities — it does not import add-on systems or call add-on functions. The coupling is data-level through the ECS, which is the intended pattern. Module import graphs remain acyclic.

---

## 6. Cargo.toml Configuration

```toml
[package]
name = "juanschott"
version = "0.1.0"
edition = "2021"
license = "MIT OR Apache-2.0"

[dependencies]
bevy = { version = "0.18", features = ["3d"] }
avian3d = "0.6"

[dev-dependencies]
# Testing utilities

[features]
default = ["3d"]
3d = ["bevy/3d"]
networked = ["lightyear"]
lightyear = { version = "0.26", optional = true }

[profile.dev]
opt-level = 1

[profile.dev.package."*"]
opt-level = 3

[profile.release]
lto = "thin"
codegen-units = 1
strip = true
```

### Profile rationale

- **`profile.dev`** with `opt-level = 1` keeps dev builds fast while avoiding the severe slowdown of fully unoptimised Bevy systems.
- **`profile.dev.package."*"`** at `opt-level = 3` compiles all dependencies (Bevy, Avian3d, etc.) with full optimisation even in dev mode. This eliminates the majority of frame time caused by unoptimised dependency code without slowing the rebuild cycle for game code.
- **`profile.release`** uses thin LTO and single codegen unit for maximum runtime performance in distributed builds.

### Feature flags

The `networked` feature gate controls compilation of the entire `src/network/` module. In Phase 1 and 2, the game compiles without lightyear, keeping build times down and the dependency tree minimal. When Phase 3 begins, `cargo build --features networked` enables multiplayer.

---

## 7. Future Workspace Splitting

When the crate grows beyond ~20,000 lines or when separate compilation is warranted by CI caching or team boundaries, the following workspace structure is recommended:

```
juanschott/
├── crates/
│   ├── juanschott_core/       # constants.rs, lib.rs types (AppState, Team)
│   ├── juanschott_player/     # src/player/
│   ├── juanschott_weapons/    # src/weapons/
│   ├── juanschott_health/     # src/health/
│   ├── juanschott_addons/     # src/addons/
│   ├── juanschott_map/        # src/map/
│   ├── juanschott_network/    # src/network/ (optional)
│   ├── juanschott_render/     # src/render/ (optional)
│   ├── juanschott_ui/         # src/ui/
│   └── juanschott_modes/      # src/modes/
├── crates/game/               # Binary crate: main.rs, plugin assembly
└── Cargo.toml                 # Workspace root
```

The plugin-per-module pattern makes this split mechanical: each crate exports its plugin struct and public types, and the `game` crate depends on all of them. No system code changes are required — only `Cargo.toml` wiring and `use` path adjustments.

---

## 8. See Also

| Document | Description |
|----------|-------------|
| [Architecture Overview](architecture_overview.md) | High-level system architecture, data flow, and design principles. |
| [Systems Design](systems_design.md) | Detailed system specifications, ordering constraints, and frame budgets. |
| [Component Reference](component_reference.md) | Complete listing of every ECS component, its fields, and invariants. |
