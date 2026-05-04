# Phase 1 — Movement + Test Map

**Version Target:** v0.1  
**Estimated Duration:** 4–6 weeks  
**Status:** Planning  
**Dependencies:** None (first phase)  

---

## 1. Overview

Phase 1 establishes the foundation of JuanSchott: **player movement feel.** This is the single most important phase in the entire project. If movement does not feel responsive, fast, and expressive, every subsequent phase will suffer. Arena shooters live or die by their movement systems. The player must be able to walk, run, jump, double-jump, and slide around a test arena using WASD + mouse look, and it must feel good from the first frame.

The phase delivers a complete movement prototype with a purpose-built test map. Upon completion, anyone should be able to launch the build and spend 5–10 minutes navigating the arena without encountering any physics bugs, input dead zones, or performance issues.

---

## 2. Prerequisites

Before beginning Phase 1, the following must be available:

- **Bevy 0.18** engine installed and compiling. Verify with a blank window rendering at 60+fps.
- **Avian 3D 0.6** physics plugin integrated. Verify with a falling cube test.
- **Blender** available for test map creation. Any recent version (3.x or 4.x) is acceptable.
- **Rust toolchain** stable, with `cargo build` and `cargo run` working.
- **Git repository** initialized with `.gitignore` configured for Rust/Bevy.

---

## 3. Goals

- Implement a first-person player controller with capsule physics body
- Support keyboard input (WASD + space + shift + ctrl) and mouse look
- Implement walk, sprint, jump, double-jump, and slide mechanics
- Build a test arena in Blender with varied geometry for testing movement
- Achieve 120fps on mid-range hardware (e.g., GTX 1060 / Ryzen 5 equivalent)

---

## 4. Task Breakdown

### 4.1 Project Setup

**Task 1.1: Initialize Cargo project**

Create the Rust project with Bevy and Avian dependencies. The `Cargo.toml` must include:
- `bevy` 0.18 with default features
- `avian3d` 0.6
- Any required feature flags for the rendering backend

**Task 1.2: Establish folder structure**

```
src/
  main.rs
  player/
    mod.rs
    spawn.rs
    input.rs
    movement.rs
    camera.rs
  map/
    mod.rs
    loader.rs
  constants.rs
assets/
  maps/
    test_arena.glb
```

**Task 1.3: Create constants.rs**

Define all tunable constants in a single module:
- Movement speeds (walk, sprint, slide initial, slide decay)
- Jump velocities (first jump, double jump)
- Gravity acceleration
- Mouse sensitivity (pitch, yaw)
- Camera height and offset
- Capsule dimensions (radius, height)
- Slide duration and hitbox shrink factor

All constants should be grouped logically and documented with units. These values will be tuned extensively during playtesting.

**Estimated effort:** 2–3 hours

---

### 4.2 Player Spawn

**Task 1.4: Spawn player entity**

Create a player entity with:
- `Transform` at spawn point (configurable position)
- `Visibility` component
- A capsule mesh for debug visualization (can be removed later)
- A `Name` component for editor identification

**Task 1.5: Attach physics body**

Add Avian 3D physics components:
- `RigidBody::Dynamic` with locked rotations (player should not tumble)
- `Collider::capsule(radius, height)` matching the capsule dimensions from constants
- `MassProperties` tuned for responsive movement (not too heavy, not too floaty)
- Linear damping and angular damping set to prevent drift

**Task 1.6: Spawn camera as child**

Spawn a `Camera3dBundle` as a child of the player entity at eye height offset. Configure:
- Field of view (recommended: 90–100 degrees for arena shooter feel)
- Near/far clip planes appropriate for arena scale
- Projection: Perspective
- No HDR or post-processing at this stage

**Estimated effort:** 3–4 hours

---

### 4.3 Input Capture

**Task 1.7: Define input bindings**

Use Bevy's input system to capture:
- **Keyboard:** W (forward), A (left), S (back), D (right), Space (jump), Left Shift (sprint), Left Ctrl (crouch/slide)
- **Mouse:** Delta movement for look. Capture raw mouse input for zero smoothing.
- **Mouse buttons:** Not needed this phase (reserved for combat in Phase 2)

**Task 1.8: Build input resource**

Create an `InputState` resource that aggregates all input each frame:
- `move_direction: Vec2` — normalized WASD direction
- `look_delta: Vec2` — mouse delta for the frame
- `jump_pressed: bool`
- `sprint_held: bool`
- `crouch_pressed: bool`

The resource is updated every frame in a `PreUpdate` system and consumed by movement systems in `Update`.

**Estimated effort:** 2–3 hours

---

### 4.4 Mouse Look

**Task 1.9: Implement pitch/yaw rotation**

Apply mouse delta to player rotation:
- **Yaw:** Rotate the player entity around the Y (up) axis. This is global — the player faces where the camera looks horizontally.
- **Pitch:** Rotate the camera child around its local X axis. Clamp pitch to prevent flipping (recommended: -89° to +89°).
- Apply sensitivity multipliers from `constants.rs`.
- Optionally support sensitivity scaling by FOV for consistent feel at different field-of-view values.

**Task 1.10: Handle edge cases**

- Capture and lock the cursor to the window when gameplay is active
- Handle window focus loss (release cursor)
- Reset mouse delta accumulation on focus regain to prevent yaw spikes
- Support configurable sensitivity via constants (no settings UI yet)

**Estimated effort:** 3–4 hours

---

### 4.5 Walk / Sprint

**Task 1.11: Implement walk movement**

Calculate movement direction from WASD input relative to the camera's yaw orientation:
- Project the camera's forward and right vectors onto the horizontal plane (zero out Y component, normalize)
- Multiply by `InputState.move_direction` to get the world-space movement vector
- Scale by walk speed from constants
- Apply as velocity or force to the physics body (prefer velocity-based for tighter control)

**Task 1.12: Implement sprint**

When `sprint_held` is true:
- Multiply movement speed by sprint multiplier (e.g., 1.5x)
- Optionally widen FOV slightly for speed sensation (bypass for v0.1 if complex)
- Sprint only works when the player is grounded and moving forward (not sideways/backward)

**Task 1.13: Acceleration and deceleration**

Movement should not be instant. Implement:
- **Acceleration:** Interpolate current velocity toward target velocity over time (e.g., 10–15 frames to reach full speed)
- **Deceleration:** Faster deceleration than acceleration for snappy stops (e.g., 5–8 frames to reach zero)
- **Air control:** Reduced acceleration factor when airborne (e.g., 0.3x ground acceleration)

**Estimated effort:** 4–6 hours

---

### 4.6 Jump

**Task 1.14: Ground detection**

Implement a ground check system:
- Cast a ray downward from the player's position
- If the ray hits a surface within a small distance (e.g., capsule radius + small tolerance), the player is grounded
- Store `is_grounded` state on a player component
- Consider edge cases: standing on the lip of a platform, near walls

**Task 1.15: Jump velocity**

When `jump_pressed` and `is_grounded`:
- Set the player's vertical velocity to the jump velocity from constants
- Set `is_grounded` to false
- Apply gravity continuously each frame when not grounded
- Gravity should feel fast — arena shooters have high gravity for snappy aerial arcs

**Task 1.16: Gravity implementation**

Apply a constant downward acceleration each physics step:
- `velocity.y -= gravity * delta_time`
- Ensure terminal velocity is capped to prevent physics explosions
- Verify that the player settles on the ground without bouncing (may require tuning linear damping)

**Estimated effort:** 4–5 hours

---

### 4.7 Double Jump

**Task 1.17: State tracking**

Add a `JumpState` component to the player:
- `Grounded` — can perform first jump
- `Airborne(jumps_remaining: u32)` — can perform additional jumps if `jumps_remaining > 0`
- Transition from `Airborne` back to `Grounded` when ground detection triggers

**Task 1.18: Second jump execution**

When `jump_pressed` and state is `Airborne(1)`:
- Set vertical velocity to double-jump velocity (may differ from first jump — typically slightly weaker or directional)
- Decrement `jumps_remaining` to 0
- Optionally add a small visual/audio cue (placeholder is fine for v0.1)

**Task 1.19: Coyote time (optional but recommended)**

Allow a brief window (e.g., 100–150ms) after leaving a ledge where the player can still perform a "grounded" jump. This is a quality-of-life feature that dramatically improves feel:
- Track `time_since_left_ground`
- If `jump_pressed` and `time_since_left_ground < coyote_time`, treat as grounded jump
- This is a high-priority polish item — implement it if time allows within the phase

**Estimated effort:** 3–4 hours

---

### 4.8 Slide

**Task 1.20: Slide activation**

A slide triggers when:
- Player is sprinting (sprint_held + moving forward)
- Player presses crouch
- Player is grounded

Upon slide activation:
- Apply an immediate speed burst in the current movement direction (e.g., 1.3x current sprint speed)
- Shrink the capsule collider height (player is lower to the ground)
- Enter `Sliding` state with a timer

**Task 1.21: Slide deceleration**

While sliding:
- Gradually reduce speed over the slide duration (e.g., 0.8–1.2 seconds)
- Player cannot change direction significantly while sliding (reduced steer authority)
- Player cannot jump until the slide ends (or allow slide-jump as an advanced tech — design decision)
- At the end of the slide duration, return to walk/sprint state and restore capsule height

**Task 1.22: Slide hitbox adjustment**

- Interpolate the capsule collider from standing height to crouch height over a few frames
- Ensure the camera lowers with the hitbox change
- On slide end, interpolate back to standing height
- Verify the player does not clip through floors or walls during hitbox transitions

**Estimated effort:** 4–6 hours

---

### 4.9 Test Map

**Task 1.23: Design test arena layout**

The test arena is a purpose-built environment for validating movement mechanics. It should include:
- A large flat floor area for basic walking and sprinting
- Walls around the perimeter to test collision
- Several ramps of varying angles (15°, 30°, 45°) to test slope movement
- Platforms at different heights (requiring single jump, double jump, and creative movement to reach)
- A long straight corridor for sprint and slide testing
- Low overhead geometry to test crouch/slide hitbox changes
- Gaps of varying width to test jump distance

**Task 1.24: Build map in Blender**

Model the test arena in Blender:
- Use simple geometry (no textures needed — flat colors are fine)
- Scale everything in meters (1 Blender unit = 1 game meter)
- The player capsule is approximately 0.5m radius, 1.8m tall — design gaps and platforms accordingly
- Apply basic materials with distinct colors per surface type for visual clarity

**Task 1.25: Export and load glTF**

- Export the Blender scene as `.glb` (binary glTF)
- Load the model in Bevy using the standard asset loader
- Verify all geometry appears correctly in the game
- Check scale, orientation (Blender Z-up vs. Bevy Y-up), and materials

**Task 1.26: Collider generation**

Generate physics colliders for the test map:
- Use Avian's collider generation from mesh, or manually place simplified colliders
- For flat surfaces, use box colliders (more performant than mesh colliders)
- For ramps, angled box colliders or mesh colliders are acceptable
- Verify the player cannot fall through the floor or walk through walls

**Estimated effort:** 5–8 hours

---

### 4.10 Camera Sync

**Task 1.27: Frame-accurate camera positioning**

Ensure the camera is positioned at the correct eye height relative to the physics body every frame:
- After physics simulation, read the player entity's `Transform`
- Set the camera's local position to `(0.0, eye_height, 0.0)` relative to the player
- During slide, adjust eye height to match the reduced capsule
- During jump, the camera follows the physics body naturally (no additional logic needed if camera is a child entity)

**Task 1.28: Camera smoothing (optional)**

Consider adding subtle camera effects for juice:
- View bob during walk/sprint (very subtle — can cause motion sickness if overdone)
- Camera tilt during slide (slight roll toward the slide direction)
- Camera shake on landing from a high jump (single frame, very subtle)

These are stretch goals for v0.1. Core camera positioning must work perfectly before any effects are added.

**Estimated effort:** 2–3 hours

---

## 5. Task Summary

| Task Group | Tasks | Estimated Hours |
|-----------|-------|----------------|
| Project Setup | 1.1, 1.2, 1.3 | 2–3 |
| Player Spawn | 1.4, 1.5, 1.6 | 3–4 |
| Input Capture | 1.7, 1.8 | 2–3 |
| Mouse Look | 1.9, 1.10 | 3–4 |
| Walk/Sprint | 1.11, 1.12, 1.13 | 4–6 |
| Jump | 1.14, 1.15, 1.16 | 4–5 |
| Double Jump | 1.17, 1.18, 1.19 | 3–4 |
| Slide | 1.20, 1.21, 1.22 | 4–6 |
| Test Map | 1.23, 1.24, 1.25, 1.26 | 5–8 |
| Camera Sync | 1.27, 1.28 | 2–3 |
| **Total** | **28 tasks** | **32–46 hours** |

At 10–15 productive hours per week, this maps to the 4–6 week estimate. The task count exceeds the initial estimate of 15–20 because each high-level task has been decomposed into specific implementation steps.

---

## 6. Acceptance Criteria

Phase 1 is complete when **all** of the following are true:

1. **Walk:** Player moves at a consistent walk speed using WASD. Movement direction is relative to camera facing. Acceleration feels smooth.
2. **Sprint:** Player moves faster when holding shift. Sprint only works while grounded and moving forward. Speed transition is perceptible but not jarring.
3. **Jump:** Player jumps when pressing space while grounded. Jump arc feels snappy — high initial velocity, strong gravity pull. Player lands cleanly without bouncing.
4. **Double Jump:** Player can jump once more while airborne. Double jump provides a second upward impulse. No more than two jumps per grounding.
5. **Slide:** Player slides when pressing crouch while sprinting. Slide provides a speed burst, then decelerates. Hitbox visibly shrinks. Player can slide under low geometry.
6. **Mouse Look:** Camera rotates smoothly with mouse movement. Pitch is clamped. No cursor visible during gameplay. No yaw spikes on window refocus.
7. **Test Map:** All geometry is present and collidable. Player cannot escape the arena. All platforms are reachable using movement mechanics. Ramps are traversable.
8. **Performance:** Game runs at 120fps on mid-range hardware. Frame time is consistent (no stuttering spikes above 12ms). No memory leaks over 10 minutes of continuous play.

---

## 7. Test Protocol

### 7.1 Manual Movement Test

1. Launch the build.
2. Walk around the flat floor area for 60 seconds. Verify no stuttering, no clipping, no jitter.
3. Sprint around the perimeter. Verify speed gain is immediate and consistent.
4. Jump onto each platform. Verify landing detection works at all heights.
5. Double jump to reach the highest platform. Verify the second jump activates reliably.
6. Sprint-slide through the corridor. Verify the speed burst, deceleration, and hitbox shrink all work.
7. Slide under a low obstacle. Verify the player fits and does not clip.
8. Walk up and down each ramp. Verify the player does not slide down ramps unintentionally at low angles and can climb at all angles.

### 7.2 Frame Time Measurement

1. Enable Bevy's diagnostic frame time display.
2. Navigate the arena continuously for 5 minutes.
3. Record minimum, maximum, and average frame times.
4. Target: average ≤ 8.3ms (120fps), maximum ≤ 16.6ms (60fps floor).

### 7.3 Edge Case Testing

1. Walk into a corner — verify no physics jitter or tunneling.
2. Jump into a ceiling — verify the player is pushed down without physics explosion.
3. Slide into a wall — verify the player stops cleanly.
4. Double jump near a wall — verify no wall-clipping or physics anomaly.
5. Alt-tab away and back — verify no mouse delta spike.

---

## 8. Known Design Decisions

- **Kinematic vs. Dynamic body:** The player uses a dynamic rigid body with locked rotations. Forces and velocities are applied directly rather than through kinematic movement. This allows Avian's collision response to handle wall sliding and ramp behavior naturally.
- **Ground detection via raycast:** A downward raycast is preferred over collision events because it is more predictable and does not depend on the physics step's contact manifold, which can be unreliable for capsule-terrain contact.
- **No wall-run in Phase 1:** Wall-run/wall-jump is a Phase 2 feature that depends on wall detection systems that are better developed alongside combat movement tech.

---

## 9. Exit Criteria

Upon completing Phase 1 and meeting all acceptance criteria:

1. Tag the repository as `v0.1`.
2. Distribute the build to all contributors for playtesting.
3. Collect movement feel feedback and log any adjustments needed.
4. Proceed to Phase 2 (Combat).

Any movement feel adjustments identified during Phase 2 playtesting should be backported to the movement systems, not patched into combat code. The movement layer must remain clean and independent.
