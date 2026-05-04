# Component Reference

This is the authoritative reference for every ECS component in JuanSchott. Follow Bevy's rustdoc pattern: summary line, detailed description, fields, spawn pattern, which systems read/write it.

Components are listed alphabetically within their category. Each entry includes the struct definition, field-level documentation, the systems that read or write the component, and the canonical spawn pattern used in production code.

---

## Table of Contents

- [Player Components](#player-components)
  - [Player](#player)
  - [PlayerInput](#playerinput)
  - [MovementState](#movementstate)
  - [JumpState](#jumpstate)
  - [Health](#health)
  - [RegenTimer](#regentimer)
  - [ActiveWeapon](#activeweapon)
  - [AddOnSlots](#addondlots)
  - [PlayerCamera](#playercamera)
- [Events](#events)
  - [DamageEvent](#damageevent)
- [Entity Spawn Patterns](#entity-spawn-patterns)
- [Resource Definitions](#resource-definitions)
  - [MovementConfig](#movementconfig)
  - [HealthConfig](#healthconfig)
- [See Also](#see-also)

---

## Player Components

### Player

Marker-struct identifying a player-controlled entity. Carries the network client identifier and the character species, both of which remain constant for the lifetime of the entity. In offline mode `client_id` is `None`; in multiplayer it is set to the `Lightyear` client ID on spawn and never mutated afterward.

```rust
#[derive(Component)]
pub struct Player {
    pub client_id: Option<u64>,
    pub species: Species,
}
```

**Fields**

| Field | Type | Description |
|---|---|---|
| `client_id` | `Option<u64>` | `None` in offline sessions. `Some(id)` maps to the Lightyear connection when running as a networked client. |
| `species` | `Species` | Determines base stats and animation set. Variants: `Human`, `Humanoid`, `Artificial`, `Cybersymbiote`. |

**Systems that read:** `movement_system`, `jump_system`, `camera_sync_system`, `weapon_fire_system`, `weapon_reload_system`, `addon_activation_system`, `damage_resolution_system`, `kill_feed_system`.

**Systems that write:** `spawn_player_system` (initialisation only — the component is never mutated after spawn).

**Spawned with:** `Transform`, `Visibility`, `RigidBody::Kinematic`, `Collider::capsule(0.3, 0.9)`, `LinearVelocity`, `PlayerInput`, `MovementState`, `JumpState`, `Health`, `RegenTimer`, `AddOnSlots`, `ActiveWeapon`.

---

### PlayerInput

Per-frame input snapshot attached to every `Player` entity. All fields are written every frame by `input_capture_system` and then consumed by downstream gameplay systems. Because the component is replaced in its entirety each tick, no partial-update races can occur.

```rust
#[derive(Component, Default)]
pub struct PlayerInput {
    pub forward: bool,
    pub backward: bool,
    pub left: bool,
    pub right: bool,
    pub sprint: bool,
    pub jump: bool,
    pub crouch: bool,
    pub fire: bool,
    pub reload: bool,
    pub aim_down_sights: bool,
    pub mouse_delta: Vec2,
    pub ability_1: bool,
    pub ability_2: bool,
    pub ability_3: bool,
}
```

**Fields**

| Field | Type | Description |
|---|---|---|
| `forward` | `bool` | W / left-stick up. |
| `backward` | `bool` | S / left-stick down. |
| `left` | `bool` | A / left-stick left. |
| `right` | `bool` | D / left-stick right. |
| `sprint` | `bool` | Left shift. Multiplies `walk_speed` by the sprint ratio defined in `MovementConfig`. |
| `jump` | `bool` | Space. Feeds into `JumpState.jump_buffer_timer`. |
| `crouch` | `bool` | Left ctrl. Transitions `MovementState` to `Crouching` or triggers a slide if the player is sprinting. |
| `fire` | `bool` | Left mouse. Read by weapon fire systems to begin or continue firing. |
| `reload` | `bool` | R key. Initiates the weapon reload sequence. |
| `aim_down_sights` | `bool` | Right mouse. Reduces FOV and recoil, increases accuracy. |
| `mouse_delta` | `Vec2` | Relative mouse movement since last frame, scaled by `MovementConfig::mouse_sensitivity`. |
| `ability_1` / `ability_2` / `ability_3` | `bool` | Bound to 1, 2, 3 keys. Each maps to a slot in `AddOnSlots`. |

**Written by:** `input_capture_system` (every frame, unconditional).

**Read by:** `movement_system`, `jump_system`, `slide_system`, `weapon_fire_system`, `weapon_reload_system`, `addon_activation_system`, `ads_system`.

---

### MovementState

Tracks the current locomotion state machine for a player. The state determines which physics forces and acceleration curves are applied each tick. Transitions are driven by `movement_system` and `slide_system` based on ground detection, wall proximity, and input.

```rust
#[derive(Component, Default)]
pub struct MovementState {
    pub state: MovementStateKind,
    pub slide_timer: Option<Timer>,
    pub wall_run_timer: Option<Timer>,
    pub last_wall_entity: Option<Entity>,
    pub double_jump_available: bool,
}

#[derive(Default)]
pub enum MovementStateKind {
    #[default]
    Grounded,
    Airborne,
    Sliding,
    WallRunning,
    Crouching,
}
```

**Fields**

| Field | Type | Description |
|---|---|---|
| `state` | `MovementStateKind` | The active movement state. See the enum variants below. |
| `slide_timer` | `Option<Timer>` | `Some` while sliding. Duration governed by `MovementConfig::slide_duration`. |
| `wall_run_timer` | `Option<Timer>` | `Some` while wall-running. Duration governed by `MovementConfig::wall_run_duration`. |
| `last_wall_entity` | `Option<Entity>` | The wall entity most recently wall-run on. Prevents the player from immediately re-attaching to the same wall. Reset to `None` on grounding. |
| `double_jump_available` | `bool` | `true` after grounding, consumed on the first mid-air jump. |

**State transitions**

```
Grounded  ──(leave ground)──►  Airborne
Airborne  ──(land)───────────►  Grounded
Grounded  ──(crouch+sprint)─►  Sliding
Sliding   ──(timer expire)──►  Crouching
Crouching ──(release crouch)►  Grounded
Airborne  ──(wall detect)───►  WallRunning
WallRunning──(timer expire)─►  Airborne
```

**Written by:** `movement_system`, `slide_system`, `wall_run_system`, `jump_system`.

**Read by:** `movement_system`, `animation_state_system`, `camera_sync_system`.

---

### JumpState

Manages ground detection and the coyote-time jump buffer. The `jump_buffer_timer` stores a 100 ms window so that a jump press slightly before landing is still honoured, providing forgiving platforming feel.

```rust
#[derive(Component)]
pub struct JumpState {
    pub grounded: bool,
    pub double_jump_available: bool,
    pub jump_buffer_timer: Timer,
}
```

**Fields**

| Field | Type | Description |
|---|---|---|
| `grounded` | `bool` | `true` when the `Collider` cast downward hits a surface within the ground tolerance. Updated by `ground_detection_system`. |
| `double_jump_available` | `bool` | Reset to `true` on landing, consumed when a mid-air jump fires. |
| `jump_buffer_timer` | `Timer` | 100 ms window. Started when `PlayerInput::jump` is `true` while airborne. Consumed by `jump_system` if the player lands before the timer expires. |

**Written by:** `ground_detection_system`, `jump_system`.

**Read by:** `jump_system`, `movement_system`, `animation_state_system`.

---

### Health

Tracks current and maximum hit-points. Maximum health is the base value (100) modified by any add-ons that grant bonus health. The component is never modified directly — all mutations flow through `DamageEvent` resolution and `regen_system`.

```rust
#[derive(Component)]
pub struct Health {
    pub current: f32,
    pub max: f32,
}
```

**Fields**

| Field | Type | Description |
|---|---|---|
| `current` | `f32` | Live hit-point value. Clamped to `[0.0, max]`. |
| `max` | `f32` | Base 100.0. Add-ons may raise this at spawn time. |

**Methods**

```rust
impl Health {
    pub fn new(max: f32) -> Self {
        Self { current: max, max }
    }

    pub fn is_dead(&self) -> bool {
        self.current <= 0.0
    }

    pub fn apply(&mut self, delta: f32) {
        self.current = (self.current + delta).clamp(0.0, self.max);
    }
}
```

**Written by:** `spawn_player_system` (initialisation), `damage_resolution_system` (decrement), `regen_system` (increment).

**Read by:** `death_system`, `hud_health_bar_system`, `regen_system`, `kill_feed_system`.

---

### RegenTimer

Counts time since the last damage event and drives passive health regeneration. Regen begins after `regen_delay` seconds of no incoming damage and proceeds at `regen_rate` HP per second until `Health::current` equals `Health::max`.

```rust
#[derive(Component)]
pub struct RegenTimer {
    pub timer: Timer,
    pub regen_delay: f32,
    pub regen_rate: f32,
    pub is_regening: bool,
}
```

**Fields**

| Field | Type | Description |
|---|---|---|
| `timer` | `Timer` | Counts upward from the last damage tick. Reset to zero by `damage_resolution_system`. |
| `regen_delay` | `f32` | Seconds of uninterrupted time required before regeneration begins. Default 3.0 s. |
| `regen_rate` | `f32` | HP restored per second while regenerating. Default 50.0 HP/s. |
| `is_regening` | `bool` | `true` once the delay has elapsed and the player is actively healing. |

**Written by:** `regen_system` (tick / reset), `damage_resolution_system` (reset timer on hit).

**Read by:** `regen_system`, `hud_regen_indicator_system`.

---

### ActiveWeapon

Dual-slot weapon container. The player carries a primary and secondary weapon and may swap between them with a short animation timer. Each `WeaponInstance` tracks its own ammo pool, fire cooldown, and reload state independently.

```rust
#[derive(Component)]
pub struct ActiveWeapon {
    pub primary: WeaponInstance,
    pub secondary: WeaponInstance,
    pub current_slot: WeaponSlot,
    pub swap_timer: Option<Timer>,
}

pub struct WeaponInstance {
    pub weapon_type: WeaponType,
    pub ammo: u32,
    pub max_ammo: u32,
    pub fire_cooldown: Timer,
    pub reload_timer: Option<Timer>,
}

pub enum WeaponSlot { Primary, Secondary }

pub enum WeaponType {
    Hitscan(HitscanType),
    Projectile(ProjectileType),
    Beam(BeamType),
}
```

**Fields (ActiveWeapon)**

| Field | Type | Description |
|---|---|---|
| `primary` | `WeaponInstance` | Slot 1 weapon. Always populated on spawn. |
| `secondary` | `WeaponInstance` | Slot 2 weapon. Always populated on spawn. |
| `current_slot` | `WeaponSlot` | Which slot is active. Determines which instance receives fire / reload input. |
| `swap_timer` | `Option<Timer>` | `Some` during the weapon-swap animation (~300 ms). Input is blocked while this timer is active. |

**Fields (WeaponInstance)**

| Field | Type | Description |
|---|---|---|
| `weapon_type` | `WeaponType` | Discriminant carrying the weapon variant and its static data (damage, range, spread, projectile speed, etc.). |
| `ammo` | `u32` | Current magazine count. Decremented on each shot. |
| `max_ammo` | `u32` | Magazine capacity. `ammo` is refilled to this value on reload completion. |
| `fire_cooldown` | `Timer` | Minimum interval between shots. Governed by the weapon's fire rate. |
| `reload_timer` | `Option<Timer>` | `Some` while a reload is in progress. Duration set by weapon definition. |

**Written by:** `spawn_player_system` (initialisation via `default_loadout()`), `weapon_fire_system` (ammo decrement, cooldown start), `weapon_reload_system` (timer start, ammo refill on completion), `weapon_swap_system` (slot switch, swap timer).

**Read by:** `weapon_fire_system`, `weapon_reload_system`, `hud_weapon_hud_system`, `crosshair_system`.

---

### AddOnSlots

Holds up to three equippable add-ons (abilities). Each slot has an independent cooldown timer. Add-ons are activated via `ability_1` / `ability_2` / `ability_3` in `PlayerInput`.

```rust
#[derive(Component)]
pub struct AddOnSlots {
    pub slots: [Option<AddOn>; 3],
    pub active_cooldowns: [Option<Timer>; 3],
}
```

**Fields**

| Field | Type | Description |
|---|---|---|
| `slots` | `[Option<AddOn>; 3]` | Fixed-size array. `None` means the slot is empty. `AddOn` is an enum with variants for each equippable ability (dash, shield, etc.). |
| `active_cooldowns` | `[Option<Timer>; 3]` | Parallel array. `Some(timer)` while the corresponding add-on is on cooldown. The timer is set from `AddOn::cooldown_duration()` on activation. |

**Written by:** `spawn_player_system` (initial loadout), `addon_activation_system` (start cooldown), `addon_cooldown_system` (tick timers).

**Read by:** `addon_activation_system`, `hud_ability_bar_system`.

---

### PlayerCamera

Zero-sized marker component attached to the camera child entity of a `Player`. The parent–child relationship splits rotation responsibilities:

- **Parent (`Player` entity) Transform:** controls yaw (horizontal look).
- **Child (`PlayerCamera` entity) Transform:** controls pitch (vertical look), clamped to ±89° to avoid gimbal lock.

```rust
#[derive(Component)]
pub struct PlayerCamera;
```

**Spawned as a child** at `Vec3::new(0.0, 1.6, 0.0)` relative to the player entity, representing eye height.

**Written by:** `camera_sync_system` (pitch rotation).

**Read by:** `camera_sync_system`, `weapon_fire_system` (ray origin), `hud_render_system`.

---

## Events

### DamageEvent

Fire-and-forget event emitted whenever an entity should take damage. All damage flows through this single event type to ensure consistent logging, kill-feed updates, and regen-timer resets.

```rust
#[derive(Event)]
pub struct DamageEvent {
    pub target: Entity,
    pub attacker: Entity,
    pub damage: f32,
    pub is_headshot: bool,
    pub damage_type: DamageType,
}
```

**Fields**

| Field | Type | Description |
|---|---|---|
| `target` | `Entity` | The entity receiving damage. Must have a `Health` component. |
| `attacker` | `Entity` | The entity that caused the damage (used for kill attribution). |
| `damage` | `f32` | Raw damage value before headshot multiplier. |
| `is_headshot` | `bool` | If `true`, `damage` is multiplied by `HealthConfig::headshot_multiplier` during resolution. |
| `damage_type` | `DamageType` | Categorises the source. Variants: `Kinetic`, `Energy`, `Explosive`, `Melee`. May affect add-on interactions (e.g., energy shields resist `Kinetic` only). |

**Emitted by:** `weapon_fire_system` (hitscan/projectile hit), `melee_system`, `explosion_system`.

**Read by:** `damage_resolution_system` (applies damage to `Health`, resets `RegenTimer`), `kill_feed_system`, `hit_marker_system`.

---

## Entity Spawn Patterns

### Player Entity Spawn

The canonical player spawn bundles every required component and parents a camera child entity. The pattern below is used by both `spawn_player_system` (single-player / listen-server host) and the networked-client spawn path.

```rust
fn spawn_player(mut commands: Commands, config: Res<GameConfig>) {
    commands.spawn((
        Player { client_id: None, species: Species::Human },
        Transform::from_translation(config.spawn_point),
        Visibility::default(),
        RigidBody::Kinematic,
        Collider::capsule(0.3, 0.9),
        LinearVelocity::default(),
        PlayerInput::default(),
        MovementState::default(),
        JumpState::new(0.1),
        Health::new(100.0),
        RegenTimer::new(3.0, 50.0),
        AddOnSlots::default(),
        ActiveWeapon::default_loadout(),
    )).with_children(|parent| {
        parent.spawn((
            PlayerCamera,
            Camera3d::default(),
            Transform::from_translation(Vec3::new(0.0, 1.6, 0.0)),
        ));
    });
}
```

**Notes on the spawn pattern:**

- `RigidBody::Kinematic` is chosen because player movement is driven by explicit velocity writes rather than forces. This gives deterministic, frame-rate-independent movement without fighting the physics solver.
- `Collider::capsule(0.3, 0.9)` defines a capsule with radius 0.3 m and half-height 0.9 m, yielding a total standing height of approximately 1.8 m.
- `JumpState::new(0.1)` creates a 100 ms jump buffer timer.
- `RegenTimer::new(3.0, 50.0)` sets a 3-second delay before regen begins, restoring 50 HP/s.
- `ActiveWeapon::default_loadout()` equips the starting primary and secondary weapons as defined by the current game mode configuration.

---

## Resource Definitions

Resources are singleton `Res` / `ResMut` values that parameterise gameplay systems. They are loaded once at startup from the tuning constants file and are not mutated during gameplay. This separation makes it trivial to iterate on game feel by editing a single TOML file without recompiling.

### MovementConfig

Tuning constants for all player locomotion. Values shown in comments are the current defaults and are the single source of truth — do not duplicate them in system code.

```rust
#[derive(Resource)]
pub struct MovementConfig {
    pub walk_speed: f32,           // 4.5
    pub sprint_speed: f32,         // 7.0
    pub ground_accel: f32,         // 20.0
    pub ground_decel: f32,         // 30.0
    pub air_control: f32,          // 0.3
    pub jump_velocity: f32,        // 5.5
    pub double_jump_velocity: f32, // 4.5
    pub gravity: f32,              // 9.81
    pub slide_speed: f32,          // 9.0
    pub slide_decel: f32,          // 3.0
    pub slide_duration: f32,       // 1.2
    pub wall_run_speed: f32,       // 7.5
    pub wall_run_duration: f32,    // 1.5
    pub wall_run_gravity: f32,     // 2.0
    pub mouse_sensitivity: f32,    // 0.002
}
```

**Field reference**

| Field | Default | Unit | Used by |
|---|---|---|---|
| `walk_speed` | 4.5 | m/s | `movement_system` |
| `sprint_speed` | 7.0 | m/s | `movement_system` |
| `ground_accel` | 20.0 | m/s² | `movement_system` |
| `ground_decel` | 30.0 | m/s² | `movement_system` |
| `air_control` | 0.3 | ratio | `movement_system` — multiplier on ground accel when airborne |
| `jump_velocity` | 5.5 | m/s | `jump_system` — initial upward velocity |
| `double_jump_velocity` | 4.5 | m/s | `jump_system` — upward velocity for double jump |
| `gravity` | 9.81 | m/s² | `movement_system` — applied to `LinearVelocity.y` each frame |
| `slide_speed` | 9.0 | m/s | `slide_system` — initial slide velocity |
| `slide_decel` | 3.0 | m/s² | `slide_system` — deceleration while sliding |
| `slide_duration` | 1.2 | s | `slide_system` — max slide time |
| `wall_run_speed` | 7.5 | m/s | `wall_run_system` — forward velocity while on wall |
| `wall_run_duration` | 1.5 | s | `wall_run_system` — max wall-run time |
| `wall_run_gravity` | 2.0 | m/s² | `wall_run_system` — reduced gravity while wall-running |
| `mouse_sensitivity` | 0.002 | — | `camera_sync_system` — multiplier on `mouse_delta` |

### HealthConfig

Tuning constants for the health and regeneration subsystem.

```rust
#[derive(Resource)]
pub struct HealthConfig {
    pub base_health: f32,          // 100.0
    pub regen_delay: f32,          // 3.0
    pub regen_rate: f32,           // 50.0
    pub headshot_multiplier: f32,  // 2.5
}
```

**Field reference**

| Field | Default | Unit | Used by |
|---|---|---|---|
| `base_health` | 100.0 | HP | `spawn_player_system` — initial `Health::max` |
| `regen_delay` | 3.0 | s | `regen_system` — time since last damage before regen starts |
| `regen_rate` | 50.0 | HP/s | `regen_system` — HP restored per second |
| `headshot_multiplier` | 2.5 | × | `damage_resolution_system` — multiplied into damage when `is_headshot` is true |

---

## System Access Matrix

Quick-reference table showing which systems read (R) or write (W) each component. Blank cells mean the system does not access that component.

| System \ Component | Player | PlayerInput | MovementState | JumpState | Health | RegenTimer | ActiveWeapon | AddOnSlots | PlayerCamera |
|---|---|---|---|---|---|---|---|---|---|
| `input_capture_system` | | W | | | | | | | |
| `movement_system` | R | R | RW | R | | | | | |
| `jump_system` | R | R | W | RW | | | | | |
| `slide_system` | | R | RW | | | | | | |
| `wall_run_system` | | R | RW | | | | | | |
| `ground_detection_system` | | | | W | | | | | |
| `camera_sync_system` | R | R | R | | | | | | RW |
| `weapon_fire_system` | R | R | | | | | RW | | R |
| `weapon_reload_system` | R | R | | | | | RW | | |
| `weapon_swap_system` | | R | | | | | RW | | |
| `addon_activation_system` | R | R | | | | | | RW | |
| `addon_cooldown_system` | | | | | | | | W | |
| `damage_resolution_system` | R | | | | W | W | | | |
| `regen_system` | | | | | RW | RW | | | |
| `spawn_player_system` | W | W | W | W | W | W | W | W | |
| `death_system` | R | | | | R | | | | |

---

## See Also

- [systems_design.md](systems_design.md) — full system scheduling graph, ordering constraints, and run conditions.
- [crate_structure.md](crate_structure.md) — which crate owns each component and the module visibility rules.
- [../06_cross_cutting/tuning_constants.md](../06_cross_cutting/tuning_constants.md) — the TOML file that feeds `MovementConfig` and `HealthConfig` defaults, and the hot-reload workflow.
