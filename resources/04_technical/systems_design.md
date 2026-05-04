# Systems Design

> **Audience:** Developers implementing or extending JuanSchott gameplay systems.
> **Prerequisite:** Familiarity with Bevy ECS, Avian Physics, and the project's component definitions (see `component_reference.md`).
> **Version:** 0.1 — reflects Phase 1 scope unless otherwise noted.

---

## 1. Overview

This document catalogs every Bevy system in JuanSchott, organized by plugin. Each system entry specifies its name, schedule placement, system set membership, query parameters, read/write access, and ordering constraints. The intent is unambiguous enough that a developer can implement each system from its specification without consulting additional sources.

Systems are grouped into four plugins:

| Plugin | Responsibility |
|---|---|
| `PlayerPlugin` | Input capture, movement, jumping, sliding, camera synchronization |
| `WeaponPlugin` | Weapon input, fire execution (hitscan / projectile / beam), reloading |
| `HealthPlugin` | Damage application, regeneration, death detection |
| `GamePlugin` | Top-level orchestration, HUD, crosshair, scoring, match flow |

All systems run within Bevy's default `MainSchedule`. No custom schedules are introduced; ordering is controlled entirely through **system sets** and explicit `before` / `after` constraints.

---

## 2. System Set Definitions

System sets are the primary mechanism for enforcing frame-consistent ordering without brittle per-system chaining. Each set is an enum variant implementing `SystemSet` (Bevy's derive macro). Sets are configured in their respective plugin's `build` method via `app.configure_sets`.

### 2.1 PlayerSet

```rust
#[derive(SystemSet, Debug, Hash, PartialEq, Eq, Clone)]
pub enum PlayerSet {
    InputCapture,
    MovementCalc,
    MovementApply,
    CameraSync,
}
```

| Set | Purpose | Invariant |
|---|---|---|
| `InputCapture` | Read raw OS input events into `PlayerInput` resource | Must complete before any system that reads `PlayerInput` |
| `MovementCalc` | Compute target velocity from input state | Must complete before Avian's physics step |
| `MovementApply` | (Reserved) Direct position writes if bypassing Avian | Currently unused — physics handles application |
| `CameraSync` | Align camera transform with player intent | Must run after `MovementCalc` so position is settled |

**Configuration (in `PlayerPlugin::build`):**

```rust
app.configure_sets(
    PreUpdate,
    PlayerSet::InputCapture,
);
app.configure_sets(
    Update,
    (
        PlayerSet::MovementCalc.before(PlayerSet::CameraSync),
        PlayerSet::CameraSync,
    ).chain(),
);
```

### 2.2 WeaponSet

```rust
#[derive(SystemSet, Debug, Hash, PartialEq, Eq, Clone)]
pub enum WeaponSet {
    InputCapture,
    FireExecution,
    Reload,
}
```

| Set | Purpose |
|---|---|
| `InputCapture` | Read fire / reload button events, populate weapon intent |
| `FireExecution` | Execute the weapon's fire logic (raycast, projectile spawn, beam tick) |
| `Reload` | Advance reload timer, restore ammo on completion |

**Configuration:**

```rust
app.configure_sets(
    PreUpdate,
    WeaponSet::InputCapture,
);
app.configure_sets(
    Update,
    (
        WeaponSet::FireExecution,
        WeaponSet::Reload.after(WeaponSet::FireExecution),
    ),
);
```

`FireExecution` precedes `Reload` so that a weapon that fires its last round during the current frame does not immediately begin reloading within the same frame — the reload starts on the *next* frame, giving the reload system a fresh `ReloadState` to observe.

### 2.3 HealthSet

```rust
#[derive(SystemSet, Debug, Hash, PartialEq, Eq, Clone)]
pub enum HealthSet {
    DamageApply,
    RegenCheck,
    DeathCheck,
}
```

| Set | Purpose |
|---|---|
| `DamageApply` | Consume `DamageEvent` queue, subtract HP, reset regen timers |
| `RegenCheck` | Advance regen delay timer, restore HP if eligible |
| `DeathCheck` | Detect HP ≤ 0, emit `DeathEvent` |

**Configuration:**

```rust
app.configure_sets(
    Update,
    (
        HealthSet::DamageApply,
        HealthSet::RegenCheck.after(HealthSet::DamageApply),
        HealthSet::DeathCheck.after(HealthSet::DamageApply),
    ),
);
```

`DeathCheck` and `RegenCheck` both follow `DamageApply` but are unordered relative to each other — they operate on different fields and never conflict. `RegenCheck` after `DamageApply` ensures the regen timer is already reset before we decide whether to regenerate this frame.

### 2.4 GameSet

```rust
#[derive(SystemSet, Debug, Hash, PartialEq, Eq, Clone)]
pub enum GameSet {
    Input,
    Simulate,
    Render,
}
```

| Set | Purpose |
|---|---|
| `Input` | Umbrella for all input capture — `PlayerSet::InputCapture` and `WeaponSet::InputCapture` are placed inside this |
| `Simulate` | Game logic: movement, weapons, health |
| `Render` | UI and visual updates |

**Configuration:**

```rust
app.configure_sets(
    Update,
    (GameSet::Input, GameSet::Simulate, GameSet::Render).chain(),
);
```

Cross-plugin ordering is established by making plugin-specific sets members of `GameSet`:

```rust
app.configure_sets(PreUpdate, PlayerSet::InputCapture.in_set(GameSet::Input));
app.configure_sets(PreUpdate, WeaponSet::InputCapture.in_set(GameSet::Input));
app.configure_sets(Update, PlayerSet::MovementCalc.in_set(GameSet::Simulate));
app.configure_sets(Update, WeaponSet::FireExecution.in_set(GameSet::Simulate));
app.configure_sets(Update, HealthSet::DamageApply.in_set(GameSet::Simulate));
app.configure_sets(PostUpdate, GameSet::Render);
```

---

## 3. System Schedule Diagram

The following shows the complete frame execution order. Indentation implies dependency ordering within the same schedule phase.

```
PreUpdate ─ GameSet::Input
│
├── input_capture_system          [PlayerSet::InputCapture]
└── weapon_input_capture_system   [WeaponSet::InputCapture]

Update ─ GameSet::Simulate
│
├── mouse_look_system             [PlayerSet::CameraSync]
│
├── movement_system               [PlayerSet::MovementCalc]
├── jump_system                   [PlayerSet::MovementCalc]
├── slide_system                  [PlayerSet::MovementCalc]
├── double_jump_system            [PlayerSet::MovementCalc]
├── wall_run_system               [PlayerSet::MovementCalc] ─── Phase 2
│
├── physics_step                  [Avian — external]
│
├── camera_sync_system            [PlayerSet::CameraSync]
│
├── hitscan_fire_system           [WeaponSet::FireExecution]
├── projectile_fire_system        [WeaponSet::FireExecution]
├── projectile_move_system        [WeaponSet::FireExecution]
├── beam_tick_system              [WeaponSet::FireExecution]
│
├── reload_system                 [WeaponSet::Reload]
│
├── damage_application_system     [HealthSet::DamageApply]
├── regen_system                  [HealthSet::RegenCheck]
└── death_system                  [HealthSet::DeathCheck]

PostUpdate ─ GameSet::Render
│
├── hud_update_system
├── crosshair_update_system
└── score_update_system
```

**Critical ordering rule:** `physics_step` (Avian) sits between `MovementCalc` and `CameraSync`. All velocity-writing systems must finish before Avian integrates; camera reads the post-physics position. This prevents the camera from lagging one frame behind the player body.

---

## 4. Detailed System Descriptions

Each system is documented with a fixed schema. Code sketches are illustrative — they omit imports and some boilerplate — but the logic is precise.

---

### 4.1 Player Systems

#### input_capture_system — SYS-PLAYER-001

| Field | Value |
|---|---|
| **Schedule** | `PreUpdate` |
| **System Set** | `PlayerSet::InputCapture` |
| **Queries** | None |
| **Reads** | `EventReader<KeyboardInput>`, `EventReader<MouseMotion>` |
| **Writes** | `ResMut<PlayerInput>` |
| **Dependencies** | None (runs first in frame) |

**Description:**

Reads all raw keyboard and mouse motion events accumulated since the last frame. Populates the `PlayerInput` resource with the current frame's aggregated input state. Each action (`Forward`, `Backward`, `Left`, `Right`, `Jump`, `Sprint`, `Slide`, `Reload`) stores a `ButtonState` enum: `Pressed`, `JustPressed`, `Released`, `JustReleased`.

Mouse delta is summed across all `MouseMotion` events and then multiplied by the sensitivity constant from `MovementConfig::mouse_sensitivity`. The result is stored as `PlayerInput::mouse_delta: Vec2`.

**Edge cases:**

- **Window focus lost:** A `WindowFocused` event with `focused: false` causes all button states to be forcibly set to `Released` and `mouse_delta` zeroed. This prevents "stuck keys" when the player alt-tabs.
- **Window focus gained:** Mouse capture is re-enabled via `Cursor::grab_mode = GrabMode::Locked` and `Cursor::visible = false`.
- **Multiple mouse events per frame:** Deltas are accumulated, not replaced — fast mouse movements produce correct total delta.

```rust
fn input_capture_system(
    mut keyboard_events: EventReader<KeyboardInput>,
    mut mouse_events: EventReader<MouseMotion>,
    mut player_input: ResMut<PlayerInput>,
    mut focus_events: EventReader<WindowFocused>,
    mut windows: Query<&mut Cursor, With<PrimaryWindow>>,
    config: Res<MovementConfig>,
) {
    if focus_events.read().any(|e| !e.focused) {
        player_input.reset_all();
    }
    if focus_events.read().any(|e| e.focused) {
        if let Ok(mut cursor) = windows.single_mut() {
            cursor.grab_mode = GrabMode::Locked;
            cursor.visible = false;
        }
    }

    player_input.mouse_delta = mouse_events
        .read()
        .map(|e| e.delta)
        .sum::<Vec2>()
        * config.mouse_sensitivity;

    for event in keyboard_events.read() {
        player_input.update_from_key(event.key_code, event.state);
    }
}
```

---

#### mouse_look_system — SYS-PLAYER-002

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `PlayerSet::CameraSync` |
| **Queries** | `Query<&mut Transform, With<PlayerCamera>>` |
| **Reads** | `Res<PlayerInput>`, `Res<MovementConfig>` |
| **Writes** | `PlayerCamera` `Transform` (rotation only) |
| **Dependencies** | `after(input_capture_system)` |

**Description:**

Applies accumulated mouse delta to camera orientation. Yaw (horizontal rotation) is applied to the **player body entity** — the camera parent — so that movement direction follows where the player looks. Pitch (vertical rotation) is applied directly to the camera entity and is clamped to the range `[-89°, 89°]` (in radians: `[-1.5533, 1.5533]`) to prevent full camera inversion.

Rotation uses quaternion composition (`Quat::from_rotation_y` for yaw, `Quat::from_rotation_x` for pitch`) to remain gimbal-lock-free. The system does **not** use Euler angles internally — pitch is tracked as a separate `f32` accumulator in a `CameraPitch` component to enable precise clamping.

```rust
fn mouse_look_system(
    player_input: Res<PlayerInput>,
    config: Res<MovementConfig>,
    mut camera_query: Query<&mut Transform, With<PlayerCamera>>,
    mut body_query: Query<(&mut Transform, &mut CameraPitch), (With<PlayerBody>, Without<PlayerCamera>)>,
) {
    let delta = player_input.mouse_delta;

    for (mut body_transform, mut pitch) in &mut body_query {
        body_transform.rotation *= Quat::from_rotation_y(-delta.x * config.mouse_sensitivity_y);
        pitch.0 = (pitch.0 - delta.y * config.mouse_sensitivity_x).clamp(-1.5533, 1.5533);

        for mut cam_transform in &mut camera_query {
            cam_transform.rotation = Quat::from_rotation_x(pitch.0);
        }
    }
}
```

**Note:** If the camera is parented to the player body in the hierarchy (recommended), only the body's Y-rotation needs updating — Bevy's transform propagation handles the rest.

---

#### movement_system — SYS-PLAYER-003

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `PlayerSet::MovementCalc` |
| **Queries** | `Query<(&PlayerInput, &mut LinearVelocity, &MovementState), With<Player>>` |
| **Reads** | `Res<Time>`, `Res<MovementConfig>` |
| **Writes** | `LinearVelocity.x`, `LinearVelocity.z` (Avian component) |
| **Dependencies** | `after(input_capture_system)`, `before(PhysicsStep)` |

**Description:**

Computes the player's desired horizontal movement direction from WASD input, projected onto the XZ plane relative to the camera's yaw. Applies acceleration toward the target speed when input is present, and deceleration toward zero when input is absent. Only modifies the X and Z components of `LinearVelocity` — the Y component is owned by gravity and jump systems.

The `approach` helper moves a value toward a target by a maximum delta per second, stopping exactly at the target (no overshoot):

```rust
fn approach(current: f32, target: f32, max_delta: f32) -> f32 {
    if (current - target).abs() < max_delta {
        target
    } else if current < target {
        current + max_delta
    } else {
        current - max_delta
    }
}
```

**MovementConfig fields consumed:**

| Field | Type | Default | Purpose |
|---|---|---|---|
| `walk_speed` | `f32` | `8.0` | Maximum speed when walking |
| `sprint_speed` | `f32` | `13.0` | Maximum speed when sprinting |
| `accel` | `f32` | `60.0` | Acceleration rate (units/s²) |
| `decel` | `f32` | `40.0` | Deceleration rate (units/s²) |

```rust
fn movement_system(
    time: Res<Time>,
    config: Res<MovementConfig>,
    camera_query: Query<&Transform, With<PlayerCamera>>,
    mut query: Query<(&PlayerInput, &mut LinearVelocity, &MovementState), With<Player>>,
) {
    let camera_yaw = camera_query.single().forward().xzy(); // XZ forward
    let forward = Vec3::new(camera_yaw.x, 0.0, camera_yaw.z).normalize_or_zero();
    let right = Vec3::new(-forward.z, 0.0, forward.x);

    for (input, mut velocity, _state) in &mut query {
        let mut direction = Vec3::ZERO;
        if input.forward { direction += forward; }
        if input.backward { direction -= forward; }
        if input.left { direction -= right; }
        if input.right { direction += right; }

        let target_speed = if input.sprint { config.sprint_speed } else { config.walk_speed };

        if direction.length_squared() > 0.0001 {
            let target_vel = direction.normalize() * target_speed;
            let dt = time.delta_seconds();
            velocity.x = approach(velocity.x, target_vel.x, config.accel * dt);
            velocity.z = approach(velocity.z, target_vel.z, config.accel * dt);
        } else {
            let dt = time.delta_seconds();
            velocity.x = approach(velocity.x, 0.0, config.decel * dt);
            velocity.z = approach(velocity.z, 0.0, config.decel * dt);
        }
    }
}
```

**Key invariant:** This system never touches `velocity.y`. Gravity is applied by Avian's built-in gravity. Jump sets `velocity.y` directly. This separation prevents movement from overriding jump impulses or gravitational acceleration.

---

#### jump_system — SYS-PLAYER-004

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `PlayerSet::MovementCalc` |
| **Queries** | `Query<(&PlayerInput, &mut LinearVelocity, &mut JumpState, &Grounded), With<Player>>` |
| **Reads** | `Res<MovementConfig>` |
| **Writes** | `LinearVelocity.y`, `JumpState` |
| **Dependencies** | `after(input_capture_system)`, `before(PhysicsStep)` |

**Description:**

Manages the jump state machine. On `JustPressed(Jump)`:

1. If `Grounded == true`: set `velocity.y = config.jump_velocity`, transition `JumpState` to `Airborne`.
2. If `Grounded == false` and `JumpState == Airborne` and `config.double_jump_enabled == true`: set `velocity.y = config.double_jump_velocity`, transition to `FullyAirborne`.

Ground detection is provided by a `Grounded` marker component that is updated by a separate ground detection system (see below) which runs on Avian collision events. This decoupling allows the same jump system to work with any ground detection strategy.

**JumpState machine:**

```
         ┌──────────────────────────┐
         │                          │
      jump (grounded)          land (Grounded)
         │                          │
         ▼                          │
  ┌─────────────┐    double jump    ┌──────────────┐
  │   Airborne  │──────────────────▶│ FullyAirborne │
  └─────────────┘                   └──────────────┘
         ▲                                │
         │          land (Grounded)        │
         └────────────────────────────────┘
```

**Ground detection system (unnumbered, utility):**

| Field | Value |
|---|---|
| **Schedule** | `PostUpdate` (after Avian collision resolution) |
| **Queries** | `Query<(Entity, &mut Grounded, &Transform), With<Player>>` |
| **Reads** | `EventReader<Collision>` (Avian) |
| **Writes** | `Grounded` component |

Performs a short downward raycast (0.2m) from the player's feet. If it hits a surface with a normal whose Y component > 0.6 (roughly 53° slope tolerance), sets `Grounded(true)`. Otherwise sets `Grounded(false)`. This approach is more reliable than Avian collision events for platforms with thin geometry.

---

#### slide_system — SYS-PLAYER-005

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `PlayerSet::MovementCalc` |
| **Queries** | `Query<(&PlayerInput, &mut LinearVelocity, &mut SlideState, &Grounded), With<Player>>` |
| **Reads** | `Res<Time>`, `Res<MovementConfig>` |
| **Writes** | `LinearVelocity`, `SlideState`, `Transform` (crouch scale) |
| **Dependencies** | `after(input_capture_system)`, `before(PhysicsStep)` |

**Description:**

Activates a slide when the player presses `Slide` while grounded and moving above a minimum speed threshold (`config.slide_min_entry_speed`, default 5.0 m/s). On activation:

1. Preserve current horizontal velocity magnitude.
2. Apply a directional boost in the movement direction (`config.slide_boost`, default 3.0 m/s additive).
3. Set `SlideState::Sliding { timer: Timer::from_seconds(config.slide_duration) }`.
4. Scale player transform Y to `config.slide_crouch_scale` (default 0.5) to visually crouch.

While sliding, the player's deceleration is reduced (`config.slide_friction`, default 8.0 — lower than normal `decel` of 40.0), creating a long slide. The slide ends when either the timer expires or the player's horizontal speed drops below `config.slide_end_speed` (default 2.0 m/s). On exit, restore transform scale and set `SlideState::Ready`.

**Edge cases:**

- **Air slide:** If the player leaves the ground during a slide (e.g., slides off a ledge), the slide continues but no new slide can be initiated until grounded.
- **Jump cancel:** Player can jump out of a slide (slide-jump), which cancels the slide and applies jump velocity. This is handled by `jump_system` checking for `SlideState::Sliding` and forcibly ending the slide.

---

#### double_jump_system — SYS-PLAYER-006

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `PlayerSet::MovementCalc` |
| **Queries** | `Query<(&PlayerInput, &mut LinearVelocity, &mut JumpState), (With<Player>, Without<Grounded>)>` |
| **Reads** | `Res<MovementConfig>` |
| **Writes** | `LinearVelocity.y`, `JumpState` |
| **Dependencies** | `after(input_capture_system)`, `before(PhysicsStep)` |

**Description:**

Handles the second jump in mid-air. Logically this is part of `jump_system` but is documented separately because it can be toggled independently via `config.double_jump_enabled`. When disabled, this system is removed from the schedule entirely (configured in `PlayerPlugin::build`). The double jump velocity is typically lower than the first jump (`config.double_jump_velocity`, default 7.0 vs `jump_velocity` default 9.0) to limit vertical reach.

---

#### wall_run_system — SYS-PLAYER-007 (Phase 2)

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `PlayerSet::MovementCalc` |
| **Queries** | `Query<(&PlayerInput, &mut LinearVelocity, &mut WallRunState, &Transform), With<Player>>` |
| **Reads** | `Res<Time>`, `Res<MovementConfig>` |
| **Writes** | `LinearVelocity`, `WallRunState`, camera roll (via `CameraRoll` component) |
| **Dependencies** | `after(input_capture_system)`, `before(PhysicsStep)` |

**Description (design spec, not yet implemented):**

Detects nearby walls via lateral raycasts from the player. If the player is airborne, moving laterally, and a wall is detected within `config.wall_run_detect_distance` (default 0.5m):

1. Cancel downward gravity for this frame (set `velocity.y` to a small upward drift or zero).
2. Preserve horizontal velocity, potentially boosting along the wall surface.
3. Apply a camera roll (±5°) toward the wall for visual feedback.
4. Track `WallRunState::Running { side, timer }`.

Exit conditions: timer expires (`config.wall_run_duration`, default 1.5s), player jumps, player moves away from wall, or player lands. On exit, restore gravity and camera roll.

---

#### camera_sync_system — SYS-PLAYER-008

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `PlayerSet::CameraSync` |
| **Queries** | `Query<&Transform, With<PlayerBody>>`, `Query<&mut Transform, With<PlayerCamera>>` |
| **Reads** | Player body position |
| **Writes** | Camera position (if not parented), camera rotation (pitch only) |
| **Dependencies** | `after(PhysicsStep)`, after `PlayerSet::MovementCalc` |

**Description:**

If the camera is not a child entity of the player body (detached hierarchy), this system copies the player body's world position to the camera entity. When the camera IS a child (default), this system only needs to update pitch rotation — Bevy's transform propagation handles position inheritance.

In both cases, the system applies a configurable **camera offset** (`config.camera_offset`, default `Vec3::new(0.0, 0.6, 0.0)`) to position the camera at eye level rather than at the player's origin (feet).

The system runs **after** Avian's physics step so it reads the final, post-collision position. This eliminates one-frame camera lag.

---

### 4.2 Weapon Systems

#### weapon_input_capture_system — SYS-WEAPON-000

| Field | Value |
|---|---|
| **Schedule** | `PreUpdate` |
| **System Set** | `WeaponSet::InputCapture` |
| **Queries** | None |
| **Reads** | `EventReader<MouseButtonInput>`, `Res<PlayerInput>` |
| **Writes** | `ResMut<WeaponInput>` |
| **Dependencies** | None |

**Description:**

Reads mouse button events and the `PlayerInput` resource to populate `WeaponInput` with `fire: ButtonState`, `reload: ButtonState`, and `alt_fire: ButtonState`. Fire is bound to left mouse button; alt-fire to right mouse button; reload to R key. If `PlayerInput` indicates a reload key press, the `reload` field is set accordingly regardless of mouse state.

---

#### hitscan_fire_system — SYS-WEAPON-001

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `WeaponSet::FireExecution` |
| **Queries** | `Query<(&WeaponInput, &ActiveWeapon, &Ammo, &mut FireCooldown, &GlobalTransform), With<Player>>`, spatial query for raycast |
| **Reads** | `Res<Time>`, `Res<WeaponConfig>`, `Query<&Health>` (enemy filter) |
| **Writes** | `EventWriter<DamageEvent>`, `EventWriter<ImpactEvent>`, `FireCooldown`, `Ammo`, visual effect spawning (`Commands`) |
| **Dependencies** | `after(weapon_input_capture_system)` |

**Description:**

Implements instant-hit weapons. On `fire: JustPressed` or `fire: Pressed` (for automatic weapons), if `FireCooldown` timer has elapsed and `Ammo::current > 0`:

1. Cast a ray from the camera's position in the camera's forward direction with max range `WeaponConfig::range`.
2. If the ray hits an entity with a `Health` component: emit `DamageEvent { target: hit_entity, damage: weapon.damage, source: player_entity, hit_point, hit_normal }`.
3. If the ray hits world geometry (any entity without `Health`): emit `ImpactEvent { point: hit_point, normal: hit_normal, material: SurfaceMaterial }` which spawns a decal and particle effect.
4. If the ray hits nothing: do nothing (no visual).
5. Decrement `Ammo::current` by 1.
6. Reset `FireCooldown` timer to `weapon.fire_rate`.

**Headshot detection:** If the ray hits an entity with a `Hitbox` component, check the hit point against `Hitbox::head_bounds`. If inside, multiply damage by `WeaponConfig::headshot_multiplier` (default 2.0).

**Accuracy / spread:** For non-perfect accuracy weapons, the ray direction is randomly deviated within a cone defined by `weapon.spread` (half-angle in degrees). The random offset is generated per-shot using `rng.gen_range(-spread..spread)` on both pitch and yaw offsets.

```rust
fn hitscan_fire_system(
    time: Res<Time>,
    config: Res<WeaponConfig>,
    mut player_query: Query<(
        &WeaponInput, &ActiveWeapon, &mut Ammo, &mut FireCooldown, &GlobalTransform,
    ), With<Player>>,
    health_query: Query<Entity, With<Health>>,
    hitbox_query: Query<&Hitbox>,
    spatial_query: SpatialQuery,
    mut damage_events: EventWriter<DamageEvent>,
    mut impact_events: EventWriter<ImpactEvent>,
    mut commands: Commands,
) {
    for (weapon_input, weapon, mut ammo, mut cooldown, transform) in &mut player_query {
        cooldown.tick(time.delta());
        if !weapon_input.fire.pressed() || !cooldown.finished() || ammo.current == 0 { continue; }

        let origin = transform.translation();
        let direction = transform.forward();
        let spread_direction = apply_spread(direction, weapon.spread);
        let max_range = weapon.range;

        if let Some(hit) = spatial_query.cast_ray(
            origin, spread_direction, max_range, true, &default_filters(),
        ) {
            if health_query.contains(hit.entity) {
                let damage = if let Some(hitbox) = hitbox_query.get(hit.entity).ok() {
                    if hitbox.is_headshot(hit.position) {
                        weapon.damage * config.headshot_multiplier
                    } else { weapon.damage }
                } else { weapon.damage };

                damage_events.send(DamageEvent {
                    target: hit.entity,
                    damage,
                    source: player_entity,
                    hit_point: hit.position,
                    hit_normal: hit.normal,
                });
            } else {
                impact_events.send(ImpactEvent {
                    point: hit.position,
                    normal: hit.normal,
                });
            }
        }

        ammo.current -= 1;
        cooldown.reset();
    }
}
```

---

#### projectile_fire_system — SYS-WEAPON-002

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `WeaponSet::FireExecution` |
| **Queries** | Same player query as hitscan |
| **Reads** | `Res<Time>`, `Res<WeaponConfig>` |
| **Writes** | `Ammo`, `FireCooldown`, `Commands` (spawns projectile entity) |
| **Dependencies** | `after(weapon_input_capture_system)` |

**Description:**

Spawns a projectile entity with `LinearVelocity`, a `Projectile` marker, `Lifetime` timer, and optional `ProjectileDamage`. The projectile's initial velocity is the camera forward direction multiplied by `weapon.projectile_speed`. The projectile entity also receives:

- `Collider::sphere(weapon.projectile_radius)` for Avian collision detection
- `Sensor` marker (projectiles should not physically push objects)
- `CollisionLayers::new(GameLayer::Projectile, [GameLayer::World, GameLayer::Enemy])`
- `Lifetime(Timer::from_seconds(weapon.projectile_lifetime))`

Collision events for projectiles are handled by a separate `projectile_collision_system` (unnumbered) that listens for `Collision` events involving a `Projectile` entity, applies damage to the hit target, and despawns the projectile.

---

#### projectile_move_system — SYS-WEAPON-003

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `WeaponSet::FireExecution` |
| **Queries** | `Query<(Entity, &mut Lifetime), With<Projectile>>` |
| **Reads** | `Res<Time>` |
| **Writes** | `Lifetime`, `Commands` (despawn) |
| **Dependencies** | None (projectile velocity is handled by Avian) |

**Description:**

Ticks the `Lifetime` timer on all projectile entities. When the timer finishes, despawns the projectile. This prevents projectiles from traveling infinitely if they never hit anything. Avian handles the actual movement of projectiles via `LinearVelocity` — this system only manages cleanup.

---

#### beam_tick_system — SYS-WEAPON-004

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `WeaponSet::FireExecution` |
| **Queries** | `Query<(&WeaponInput, &ActiveWeapon, &mut Ammo, &mut BeamState, &GlobalTransform), With<Player>>` |
| **Reads** | `Res<Time>`, `Res<WeaponConfig>`, spatial query for raycast |
| **Writes** | `Ammo`, `BeamState`, `EventWriter<DamageEvent>`, visual beam rendering |
| **Dependencies** | `after(weapon_input_capture_system)` |

**Description:**

Implements continuous-fire beam weapons (e.g., energy weapons). While fire is held (`fire: Pressed`) and `Ammo::current > 0`:

1. Consume ammo at `weapon.beam_ammo_drain` per second.
2. Cast a ray each tick (every `weapon.beam_tick_interval`, default 0.1s) from camera forward.
3. If the ray hits an entity with `Health`, emit `DamageEvent` with `weapon.beam_damage_per_tick`.
4. Update `BeamState::active = true` and `BeamState::end_point` for the visual renderer.

On fire release: set `BeamState::active = false`. The rendering system reads `BeamState` to draw or hide the beam visual.

---

#### reload_system — SYS-WEAPON-005

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `WeaponSet::Reload` |
| **Queries** | `Query<(&WeaponInput, &mut Ammo, &mut ReloadState), With<Player>>` |
| **Reads** | `Res<Time>`, `Res<WeaponConfig>` |
| **Writes** | `ReloadState`, `Ammo` (on completion) |
| **Dependencies** | `after(WeaponSet::FireExecution)` |

**Description:**

Manages the reload state machine. Three states:

| State | Meaning |
|---|---|
| `Ready` | Not reloading; can fire |
| `Reloading { timer }` | Reload in progress; cannot fire |
| `Empty` | Ammo depleted; auto-reload triggers |

Transitions:

- `Ready` → `Reloading`: On reload input AND `Ammo::current < Ammo::max`.
- `Empty` → `Reloading`: Automatically triggered when `Ammo::current == 0`.
- `Reloading` → `Ready`: When timer finishes. Sets `Ammo::current = Ammo::max`.

While `Reloading`, the `FireCooldown` is effectively infinite (fire systems check `ReloadState != Ready`). The reload timer duration comes from `ActiveWeapon::reload_time`.

```rust
fn reload_system(
    time: Res<Time>,
    mut query: Query<(&WeaponInput, &mut Ammo, &mut ReloadState), With<Player>>,
) {
    for (weapon_input, mut ammo, mut reload_state) in &mut query {
        match reload_state.as_ref() {
            ReloadState::Ready => {
                if weapon_input.reload.just_pressed() && ammo.current < ammo.max {
                    *reload_state = ReloadState::Reloading {
                        timer: Timer::from_seconds(weapon.reload_time, TimerMode::Once),
                    };
                }
                if ammo.current == 0 {
                    *reload_state = ReloadState::Reloading {
                        timer: Timer::from_seconds(weapon.reload_time, TimerMode::Once),
                    };
                }
            }
            ReloadState::Reloading { timer } => {
                let mut t = timer.clone();
                t.tick(time.delta());
                if t.finished() {
                    ammo.current = ammo.max;
                    *reload_state = ReloadState::Ready;
                } else {
                    *reload_state = ReloadState::Reloading { timer: t };
                }
            }
            ReloadState::Empty => {
                *reload_state = ReloadState::Reloading {
                    timer: Timer::from_seconds(weapon.reload_time, TimerMode::Once),
                };
            }
        }
    }
}
```

---

### 4.3 Health Systems

#### damage_application_system — SYS-HEALTH-001

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `HealthSet::DamageApply` |
| **Queries** | `Query<&mut Health, &mut RegenTimer>` |
| **Reads** | `EventReader<DamageEvent>` |
| **Writes** | `Health`, `RegenTimer` |
| **Dependencies** | After all fire execution systems |

**Description:**

The central damage pipeline. For each `DamageEvent` in the current frame's event queue:

1. Look up the target entity's `Health` component.
2. Subtract `event.damage` from `Health::current`. Floor at 0 (no negative HP).
3. Reset the target's `RegenTimer` to `HealthConfig::regen_delay` seconds.
4. If the entity has an `Armor` component, apply damage reduction *before* subtracting from HP:
   ```
   effective_damage = damage * (1.0 - armor.damage_reduction).clamp(0.0, 0.9)
   ```
   Armor reduction is capped at 90% to prevent invulnerability.

**Frame batching:** Multiple `DamageEvent`s targeting the same entity in one frame are all applied sequentially. This is correct behavior — a player hit by two sources in the same frame should take both instances of damage.

```rust
fn damage_application_system(
    mut damage_events: EventReader<DamageEvent>,
    mut query: Query<(&mut Health, &mut RegenTimer, Option<&Armor>)>,
    config: Res<HealthConfig>,
) {
    for event in damage_events.read() {
        if let Ok((mut health, mut regen_timer, armor)) = query.get_mut(event.target) {
            let reduction = armor.map(|a| a.damage_reduction).unwrap_or(0.0);
            let effective_damage = event.damage * (1.0 - reduction.clamp(0.0, 0.9));
            health.current = (health.current - effective_damage).max(0.0);
            regen_timer.0 = config.regen_delay;
        }
    }
}
```

---

#### regen_system — SYS-HEALTH-002

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `HealthSet::RegenCheck` |
| **Queries** | `Query<(&mut Health, &mut RegenTimer)>` |
| **Reads** | `Res<Time>`, `Res<HealthConfig>` |
| **Writes** | `Health::current`, `RegenTimer` |
| **Dependencies** | `after(HealthSet::DamageApply)` |

**Description:**

Counts down `RegenTimer` each frame. When it reaches zero, begins restoring `Health::current` at `HealthConfig::regen_rate` HP per second. Stops when `Health::current >= Health::max`. The timer is reset on any damage (see `damage_application_system`), so regen only begins after `regen_delay` seconds of not taking damage.

```rust
fn regen_system(
    time: Res<Time>,
    config: Res<HealthConfig>,
    mut query: Query<(&mut Health, &mut RegenTimer)>,
) {
    for (mut health, mut regen_timer) in &mut query {
        if health.current >= health.max { continue; }

        regen_timer.0 -= time.delta_seconds();
        if regen_timer.0 <= 0.0 {
            regen_timer.0 = 0.0;
            health.current = (health.current + config.regen_rate * time.delta_seconds())
                .min(health.max);
        }
    }
}
```

---

#### death_system — SYS-HEALTH-003

| Field | Value |
|---|---|
| **Schedule** | `Update` |
| **System Set** | `HealthSet::DeathCheck` |
| **Queries** | `Query<(Entity, &Health, &Player), With<Health>>` |
| **Reads** | None |
| **Writes** | `EventWriter<DeathEvent>`, `Commands` (respawn logic) |
| **Dependencies** | `after(HealthSet::DamageApply)` |

**Description:**

Checks all entities with `Health` for `Health::current <= 0.0`. For each dead entity:

1. Emit `DeathEvent { entity, killer }`. The `killer` field is populated from the most recent `DamageEvent` that brought HP to zero — this requires a `LastDamageSource` component that tracks the last attacker.
2. If the entity is a player: trigger respawn sequence (disable input, set `RespawnState::Dead`, start respawn timer).
3. If the entity is an AI enemy: despawn the entity and increment the score.

**DeathEvent consumers:**

- `score_update_system`: Increments killer's score, updates scoreboard.
- `hud_update_system`: Shows kill feed notification.
- `respawn_system` (unnumbered): Manages the respawn timer and repositions the player.

**Important:** This system checks `Health::current <= 0.0` after `DamageApply` has run, ensuring it catches deaths caused by this frame's damage. It runs concurrently with `RegenCheck` since regeneration cannot revive a dead entity (regen is skipped when HP is already at max, and death check runs on the same frame as damage).

---

### 4.4 Game/UI Systems (PostUpdate)

These systems run in `PostUpdate` within `GameSet::Render`. They are read-heavy — they observe game state and update UI elements. They must never modify gameplay state.

#### hud_update_system — SYS-GAME-001

| Field | Value |
|---|---|
| **Schedule** | `PostUpdate` |
| **System Set** | `GameSet::Render` |
| **Queries** | `Query<&Health, With<Player>>`, `Query<&Ammo, With<Player>>`, UI element queries |
| **Reads** | Player health, ammo, kill feed events |
| **Writes** | UI Text components |
| **Dependencies** | None |

**Description:**

Updates HUD text elements each frame:
- Health bar width: `health.current / health.max * bar_max_width`
- Ammo counter: `ammo.current / ammo.max`
- Kill feed: Reads `DeathEvent` queue (via a separate `EventReader`), appends kill notification with 5-second fade timer.

---

#### crosshair_update_system — SYS-GAME-002

| Field | Value |
|---|---|
| **Schedule** | `PostUpdate` |
| **System Set** | `GameSet::Render` |
| **Queries** | `Query<&FireCooldown, With<Player>>`, crosshair UI query |
| **Reads** | `FireCooldown`, `ReloadState` |
| **Writes** | Crosshair UI spread/opacity |
| **Dependencies** | None |

**Description:**

Adjusts crosshair spread based on player state:
- Default spread: `crosshair_base_spread`
- Shooting: Expand by `crosshair_fire_expansion` per shot, decay back over 0.3s
- Reloading: Hide crosshair entirely
- Sprinting: Expand to `crosshair_sprint_spread`

---

#### score_update_system — SYS-GAME-003

| Field | Value |
|---|---|
| **Schedule** | `PostUpdate` |
| **System Set** | `GameSet::Render` |
| **Queries** | `Query<&mut Score, With<Player>>` |
| **Reads** | `EventReader<DeathEvent>` |
| **Writes** | `Score` component |
| **Dependencies** | None |

**Description:**

For each `DeathEvent`, finds the killer's `Score` component and increments by the kill value (default 100 for standard kills, 150 for headshots). Updates the scoreboard UI element.

---

## 5. System Ordering Summary Table

| ID | System | Schedule | Set | Runs After |
|---|---|---|---|---|
| SYS-PLAYER-001 | `input_capture_system` | PreUpdate | `PlayerSet::InputCapture` | — |
| SYS-PLAYER-002 | `mouse_look_system` | Update | `PlayerSet::CameraSync` | SYS-PLAYER-001 |
| SYS-PLAYER-003 | `movement_system` | Update | `PlayerSet::MovementCalc` | SYS-PLAYER-001, before PhysicsStep |
| SYS-PLAYER-004 | `jump_system` | Update | `PlayerSet::MovementCalc` | SYS-PLAYER-001, before PhysicsStep |
| SYS-PLAYER-005 | `slide_system` | Update | `PlayerSet::MovementCalc` | SYS-PLAYER-001, before PhysicsStep |
| SYS-PLAYER-006 | `double_jump_system` | Update | `PlayerSet::MovementCalc` | SYS-PLAYER-001, before PhysicsStep |
| SYS-PLAYER-007 | `wall_run_system` | Update | `PlayerSet::MovementCalc` | SYS-PLAYER-001, before PhysicsStep |
| SYS-PLAYER-008 | `camera_sync_system` | Update | `PlayerSet::CameraSync` | PhysicsStep |
| SYS-WEAPON-000 | `weapon_input_capture_system` | PreUpdate | `WeaponSet::InputCapture` | — |
| SYS-WEAPON-001 | `hitscan_fire_system` | Update | `WeaponSet::FireExecution` | SYS-WEAPON-000 |
| SYS-WEAPON-002 | `projectile_fire_system` | Update | `WeaponSet::FireExecution` | SYS-WEAPON-000 |
| SYS-WEAPON-003 | `projectile_move_system` | Update | `WeaponSet::FireExecution` | — |
| SYS-WEAPON-004 | `beam_tick_system` | Update | `WeaponSet::FireExecution` | SYS-WEAPON-000 |
| SYS-WEAPON-005 | `reload_system` | Update | `WeaponSet::Reload` | `WeaponSet::FireExecution` |
| SYS-HEALTH-001 | `damage_application_system` | Update | `HealthSet::DamageApply` | `WeaponSet::FireExecution` |
| SYS-HEALTH-002 | `regen_system` | Update | `HealthSet::RegenCheck` | `HealthSet::DamageApply` |
| SYS-HEALTH-003 | `death_system` | Update | `HealthSet::DeathCheck` | `HealthSet::DamageApply` |
| SYS-GAME-001 | `hud_update_system` | PostUpdate | `GameSet::Render` | — |
| SYS-GAME-002 | `crosshair_update_system` | PostUpdate | `GameSet::Render` | — |
| SYS-GAME-003 | `score_update_system` | PostUpdate | `GameSet::Render` | — |

---

## 6. Resource Access Map

This table shows which resources are read and written by multiple systems, useful for identifying potential conflicts if ordering changes.

| Resource | Readers | Writers |
|---|---|---|
| `PlayerInput` | SYS-PLAYER-002, 003, 004, 005, 006, 007 | SYS-PLAYER-001 |
| `WeaponInput` | SYS-WEAPON-001, 002, 004, 005 | SYS-WEAPON-000 |
| `MovementConfig` | SYS-PLAYER-001, 002, 003, 004, 005 | — |
| `WeaponConfig` | SYS-WEAPON-001, 002, 004, 005 | — |
| `HealthConfig` | SYS-HEALTH-001, 002 | — |
| `Time` | SYS-PLAYER-003, 004, 005, 007, SYS-WEAPON-001, 003, 004, 005, SYS-HEALTH-002 | — |

---

## 7. See Also

| Document | Purpose |
|---|---|
| `component_reference.md` | Field-level documentation for every component type referenced above |
| `crate_structure.md` | Plugin organization, module boundaries, and file layout |
| `networking.md` | How systems are replicated or predicted over the network |
