# Networking Architecture (Phase 3)

> **Status:** Draft
> **Last Updated:** 2026-05-05
> **Phase:** 3 — Multiplayer Foundation
> **Dependencies:** lightyear 0.26, Avian 3D, Bevy 0.16

---

## Table of Contents

1. [Overview](#overview)
2. [Network Model](#network-model)
3. [Server Architecture](#server-architecture)
4. [Client Architecture](#client-architecture)
5. [Replication Strategy](#replication-strategy)
6. [Prediction Model](#prediction-model)
7. [Lag Compensation for Hitscan](#lag-compensation-for-hitscan)
8. [Entity Types](#entity-types)
9. [Connection Flow](#connection-flow)
10. [Protocol Details](#protocol-details)
11. [Bandwidth Budget](#bandwidth-budget)
12. [Input System](#input-system)
13. [Physics Synchronization](#physics-synchronization)
14. [Combat Networking](#combat-networking)
15. [Error Handling and Recovery](#error-handling-and-recovery)
16. [Interest Management (Future)](#interest-management-future)
17. [Testing and Debugging](#testing-and-debugging)
18. [See Also](#see-also)

---

## Overview

Astra uses a **server-authoritative** multiplayer model built on [lightyear](https://github.com/cBournhonesque/lightyear) 0.26, integrated with Avian 3D physics. Every meaningful state change flows through the server. Clients never trust their own simulation results for anything that affects other players.

The architecture prioritizes three qualities in this order:

1. **Responsiveness** — the controlling player must feel zero input latency through client-side prediction.
2. **Consistency** — all clients must converge on the same game state, even if temporarily out of sync.
3. **Fairness** — lag compensation ensures players with higher ping are not penalized for the inherent delay in their connection.

lightyear provides the transport layer, replication machinery, prediction/rollback framework, and lag compensation out of the box. Our job is to configure it correctly, define the replication groups, and wire it into Avian and our gameplay systems.

---

## Network Model

### Authority Model

The server is the single source of truth. It owns every replicated component. Clients are thin: they capture input, render frames, and run a local predictive simulation that the server can override at any time.

```
Client                          Server
  |                               |
  |--- PlayerInput (tick N) ----->|   Client sends input
  |                               |   Server processes input
  |                               |   Server advances simulation
  |<-- Replicated State (tick N) -|   Server sends authoritative state
  |                               |
  |   Client compares predicted   |
  |   state vs server state.      |
  |   If mismatch: rollback and   |
  |   re-simulate from tick N.    |
```

### Tick Rates

| Rate | Value | Description |
|------|-------|-------------|
| Server simulation | 60 Hz | Physics step, input processing, game logic |
| Server→Client updates | 20 Hz | Replicated state snapshots sent to clients |
| Client input send | 60 Hz | One input packet per local simulation tick |
| Client rendering | Uncapped | Render as fast as the display allows |

The server simulates at 60 Hz but only sends network updates at 20 Hz. This means the server processes 3 simulation ticks for every network update. Clients receive a snapshot every 50ms and interpolate between them for non-predicted entities.

### Transport Layer

- **Primary:** UDP via lightyear's built-in transport. Uses a custom reliability layer on top of UDP — reliable-ordered for critical state changes, unreliable-sequenced for high-frequency transforms.
- **Web builds:** WebTransport (HTTP/3) as a fallback. Same logical protocol, different wire format. Configured via lightyear's transport abstraction so gameplay code is transport-agnostic.
- **Encryption:** Optional TLS for production deployments. Development builds run plaintext for easier debugging.

---

## Server Architecture

The server is a **headless Bevy application** — same ECS world, same systems, no rendering pipeline. Bevy's Cargo feature collections strip out `bevy_render`, `bevy_pbr`, `bevy_core_pipeline`, and all GPU-dependent crates. The server binary is small and fast because it never initializes a window or a Vulkan/DX12 context.

### Server Responsibilities

| Responsibility | System Group | Notes |
|---------------|-------------|-------|
| Full physics simulation | `FixedUpdate` | Avian 3D at 60 Hz |
| Input processing | `PreUpdate` | Validate and apply client inputs |
| Hit registration | `PostUpdate` | Raycasts against physics world |
| State replication | `PostUpdate` | Serialize and send to clients |
| Match logic | `Update` | Score, rounds, spawning |
| Anti-cheat validation | `PreUpdate` | Speed, position, input sanity checks |

### Server Does NOT Run

- Rendering pipeline (`bevy_render`)
- Camera systems
- UI layout or rendering
- Audio playback
- Input capture from hardware devices

### Server Configuration

```rust
// app_setup.rs (simplified)
App::new()
    .add_plugins(MinimalPlugins)
    .add_plugins(LightyearServerPlugin::new(server_config()))
    .add_plugins(AvianPhysicsPlugin::default())
    .add_plugins(AstraGameplayPlugin::server())
    // No rendering plugins
    ;
```

The `AstraGameplayPlugin` uses a `Side` enum to conditionally register systems:

```rust
pub enum Side {
    Server,
    Client,
}
```

Systems that only run on the server are gated behind `.run_if(resource_equals(Side::Server))`. Systems that only run on the client are similarly gated. Shared systems (physics stepping, component definitions) run on both.

---

## Client Architecture

The client is a full Bevy application with rendering, audio, and input. It also runs a local physics simulation for prediction purposes, but this simulation is strictly subordinate to the server's authoritative state.

### Client Responsibilities

| Responsibility | System Group | Notes |
|---------------|-------------|-------|
| Input capture | `PreUpdate` | Keyboard, mouse, gamepad |
| Rendering | `PostUpdate` | Camera, meshes, materials, VFX |
| Audio | `Update` | Spatial audio via `bevy_audio` |
| Local prediction | `FixedUpdate` | Run same physics as server |
| State correction | `PreUpdate` | Rollback if prediction was wrong |
| UI | `Update` | HUD, menus, crosshair |

### Prediction vs. Interpolation

The client treats entities differently depending on ownership:

- **Owned player entity:** The client predicts. It runs the full physics simulation locally, applying the player's inputs immediately. When the server's authoritative state arrives, the client compares. If there is a discrepancy, it rolls back and re-simulates.
- **Other player entities:** The client interpolates. It receives periodic snapshots from the server and smoothly interpolates between them. No local prediction is attempted for other players' movement.

This split is fundamental. Predicting other players would require guessing their inputs, which is unreliable and produces visible errors. Interpolating the owned player would introduce unacceptable input latency.

---

## Replication Strategy

Components are grouped by who needs them and how often they change. lightyear's `SyncChannel` system maps directly to these groups.

### Component Replication Table

| Component | Authority | Replicated To | Frequency | Channel |
|-----------|-----------|--------------|-----------|---------|
| `Transform` | Server | All clients | Every update tick (20 Hz) | Unreliable |
| `LinearVelocity` | Server | Owning client | Every update tick (20 Hz) | Unreliable |
| `AngularVelocity` | Server | Owning client | Every update tick (20 Hz) | Unreliable |
| `PlayerInput` | Client | Server | Every input tick (60 Hz) | Unreliable |
| `Health` | Server | All clients | On change | Reliable |
| `MaxHealth` | Server | All clients | On change | Reliable |
| `AmmoReserve` | Server | Owning client | On change | Reliable |
| `AmmoMagazine` | Server | Owning client | On change | Reliable |
| `ActiveWeapon` | Server | Owning client | On change | Reliable |
| `WeaponSlot` (x3) | Server | Owning client | On change | Reliable |
| `MovementState` | Server | Owning client | Every update tick | Unreliable |
| `AddOnSlots` | Server | Owning client | On change | Reliable |
| `IsGrounded` | Server | Owning client | Every update tick | Unreliable |
| `PlayerScore` | Server | All clients | On change | Reliable |
| `DamageEvent` | Server | All nearby clients | On event | Reliable |
| `KillFeedEvent` | Server | All clients | On event | Reliable |
| `PlayerClass` | Server | All clients | On spawn | Reliable |

### Replication Groups

Components are batched into replication groups to reduce overhead. A group is serialized and sent as a single unit.

**Group 1 — High-Frequency Transform (Unreliable, 20 Hz)**
- `Transform`
- `LinearVelocity` (owning client only)
- `AngularVelocity` (owning client only)
- `MovementState`
- `IsGrounded`

**Group 2 — Combat State (Reliable, On Change)**
- `Health`, `MaxHealth`
- `AmmoReserve`, `AmmoMagazine`
- `ActiveWeapon`, `WeaponSlot`
- `DamageEvent`, `KillFeedEvent`

**Group 3 — Loadout (Reliable, On Spawn/Change)**
- `PlayerClass`
- `AddOnSlots`
- `PlayerScore`

### Serialization Format

All components implement lightyear's `Serialize` and `Deserialize` traits. We use postcard for compact binary serialization. Components that contain `Vec3` or `Quat` values use half-precision floats (`f16`) for transform replication to reduce bandwidth:

- `Vec3` → 6 bytes (three `f16` values) instead of 12 bytes
- `Quat` → 8 bytes (four `f16` values) instead of 16 bytes

This halves the cost of the largest and most frequently replicated component. The precision loss (~0.001 per axis) is invisible to players.

---

## Prediction Model

Client-side prediction is the technique that makes a server-authoritative game feel responsive. Instead of waiting for the server to confirm each input, the client simulates the result locally and displays it immediately.

### What the Client Predicts

| Action | Predicted? | Scope |
|--------|-----------|-------|
| Movement (walk, sprint, slide) | Yes | Full physics simulation |
| Jump | Yes | Full physics simulation |
| Weapon fire (muzzle flash, tracers) | Yes | Visual only; damage is server-authoritative |
| Weapon switch animation | Yes | Animation only; actual slot change is server-authoritative |
| Reload animation | Yes | Animation only; ammo change is server-authoritative |
| Health change | No | Server-authoritative, applied when received |
| Death | No | Server-authoritative |
| Other players' movement | No | Interpolated from server snapshots |
| Projectile spawn | Partially | Client spawns cosmetic effect; server spawns actual projectile |

### Rollback and Re-simulation

When the server's authoritative state arrives for tick `N`, the client compares it against its own predicted state for tick `N`. If they match, nothing happens — the prediction was correct. If they differ, the client must rollback:

1. **Save the current predicted state** at the latest tick (e.g., tick `N+5`).
2. **Restore the server's authoritative state** at tick `N`.
3. **Re-simulate ticks N+1 through N+5**, applying the stored inputs for each tick in order.
4. **Compare the re-simulated state** at tick `N+5` with the saved predicted state.
5. If they still differ, apply the re-simulated state (this corrects accumulated drift).

### Rollback Window

- **Maximum rollback:** 10 ticks (~166ms at 60 Hz simulation rate).
- This limits the amount of stored input history the client must keep. Inputs older than 10 ticks are discarded.
- If the server is more than 10 ticks behind the client, the client performs a full snap to the server state (no re-simulation). This produces a visible hitch but prevents the client from drifting too far.

### Correction Smoothing

Rollback corrections can cause visible snapping if the difference between predicted and corrected state is large. To mitigate this:

- If the correction delta is small (< 0.1 units position, < 5 degrees rotation), blend from the old predicted position to the corrected position over **100ms**.
- If the correction delta is large (teleport-level), snap immediately. Large corrections indicate a fundamental desync that blending would only hide poorly.

```rust
// Pseudocode for correction smoothing
fn apply_correction(predicted: &Transform, corrected: &Transform) -> Transform {
    let delta_pos = corrected.translation.distance(predicted.translation);
    if delta_pos < 0.1 {
        // Smooth blend over 100ms
        lerp_transform(predicted, corrected, dt / 0.1)
    } else {
        // Snap immediately
        *corrected
    }
}
```

---

## Lag Compensation for Hitscan

Hitscan weapons (rifles, pistols, shotguns) require the server to evaluate whether a shot hit based on what the shooting player was seeing at the time they fired. Without lag compensation, a player with 100ms ping would need to aim 100ms ahead of their target — an unfair penalty.

### How It Works

lightyear provides built-in lag compensation. The mechanism:

1. **Client fires.** The client sends a `ShootInput` containing the tick at which the shot was taken and the aiming direction (as a ray or quaternion).
2. **Server receives the input** at a later tick (e.g., the client was at tick 100 when it fired; the server receives the message at tick 106).
3. **Server rewinds** its physics state to tick 100 — the tick the client was viewing.
4. **Server performs a raycast** against the rewound state using the client's reported aim direction.
5. **Server evaluates the hit:**
   - If the raycast intersects a hitbox: damage is applied. The shot is confirmed.
   - If the raycast misses: no damage. The client's view was outdated, or the aim was off.
6. **Server restores** the current physics state and continues simulation normally.

### Rewind Limits

- **Maximum rewind:** 200ms (12 ticks at 60 Hz). This prevents clients from exploiting high latency to "hit" targets that have long since moved.
- If a client's shot arrives with a tick older than the rewind limit, it is discarded. The client receives a `ShotRejected` event.
- This limit is configurable per-match via a server cvar.

### Hit Validation (Anti-Cheat)

The server does not blindly trust the client's aim direction. It performs sanity checks:

- **Aim direction** must be within a reasonable angular delta of the player's facing direction at the rewind tick. If the client claims to have aimed 90 degrees away from where they were looking, the shot is rejected.
- **Fire rate** must respect the weapon's cooldown. If the client sends two shots within 50ms for a weapon with a 100ms fire rate, the second shot is rejected.
- **Line of sight** at the rewind tick: if the shooter could not see the target from their reported position (e.g., a wall was between them), the shot is rejected.

### Projectile Weapons

Projectile weapons (grenades, rockets, ability projectiles) do not use lag compensation. The server spawns the projectile at the player's position on the tick it receives the fire input. The projectile's trajectory is fully server-authoritative and simulated forward from that point. Clients see a cosmetic spawn effect predicted locally, but the actual projectile physics run on the server.

---

## Entity Types

lightyear categorizes networked entities into three types based on how they are synchronized.

### Predicted Entities

Used for the player's own character. Marked with lightyear's `Predicted` component.

- The controlling client runs a full local simulation of this entity.
- The server also simulates this entity and sends authoritative state.
- The client compares and corrects.
- **Use case:** The player's character body, including transform, physics, and movement state.

```rust
#[derive(Component)]
pub struct PredictedPlayer;

// Server spawns with:
commands.entity(player_entity).insert(Predicted);
```

### Interpolated Entities

Used for other players and server-driven projectiles. Marked with lightyear's `Interpolated` component.

- The client does **not** simulate these entities locally.
- The client receives periodic snapshots and interpolates between the two most recent snapshots.
- Interpolation adds a fixed delay (the "interpolation delay", typically 2–3 ticks) to ensure smooth playback even if one snapshot arrives late.
- **Use case:** Other players' characters, server-spawned projectiles, destructible objects.

```rust
#[derive(Component)]
pub struct InterpolatedPlayer;

// Server spawns with:
commands.entity(other_player_entity).insert(Interpolated);
```

### Server-Only Entities

Entities that exist only on the server. They are never replicated to any client.

- **Use case:** Hit detection volumes (headshot colliders that are separate from the visual mesh), spawn zone markers, match boundary triggers, tactical area-of-effect zones used for game logic.
- These entities have no lightyear replication component. They are standard Bevy entities.

---

## Connection Flow

The connection sequence from client launch to gameplay-ready:

```
Client                                Server
  |                                     |
  |  1. UDP connect to server:port      |
  |------------------------------------>|
  |                                     |
  |  2. lightyear handshake             |
  |     - Protocol version check        |
  |     - Authentication token          |
  |<----------------------------------->|
  |                                     |
  |  3. Connection confirmed            |
  |<---- ConnectionAck -----------------|
  |                                     |
  |  4. Client sends LoadoutSelect      |
  |---- LoadoutSelect { class, addons } |
  |------------------------------------>|
  |                                     |
  |  5. Server spawns player entity     |
  |     - Creates entity with Predicted |
  |     - Applies loadout components    |
  |     - Places at spawn point         |
  |                                     |
  |  6. Server sends SpawnEvent         |
  |<---- SpawnEvent { entity, tick } ---|
  |                                     |
  |  7. Client begins prediction        |
  |     - Local entity mirrors server   |
  |     - Input capture starts          |
  |                                     |
  |  8. Game is live                    |
```

### Step Details

**Step 1 — UDP Connect:** The client opens a UDP socket and sends a connect request to the server's IP:port. If the connection fails (timeout after 5 seconds), the client retries up to 3 times before showing an error to the user.

**Step 2 — Handshake:** lightyear's handshake negotiates the protocol version and exchanges an authentication token. The token is a JWT containing the player's ID and a session key. The server validates the token against the matchmaker service. If validation fails, the connection is rejected with a `Denied` reason.

**Step 3 — Connection Confirmed:** The server allocates a player slot, assigns a client ID, and sends a `ConnectionAck` message.

**Step 4 — Loadout Selection:** The client sends its chosen class (Arbiter, Phantom, or Titan) and add-on selections. The server validates that the add-ons are legal for the chosen class.

**Step 5 — Entity Spawn:** The server creates the player entity with all gameplay components and the `Predicted` marker.

**Step 6 — Spawn Event:** The server notifies the client of the new entity. The client creates a local mirror entity and begins prediction.

**Step 7 — Prediction Starts:** From this point, the client captures input every tick, sends it to the server, and runs local prediction.

### Disconnection Handling

- **Client disconnect:** If the client's connection drops, the server waits 5 seconds (configurable) before removing the player entity. If the client reconnects within that window, it resumes where it left off.
- **Server disconnect:** If the server shuts down or the client is kicked, the client receives a `DisconnectReason` and returns to the main menu.
- **Timeout:** If no packets are received from a client for 10 seconds, the server considers the connection lost and initiates cleanup.

---

## Protocol Details

### Channel Configuration

lightyear channels define how messages are sent. Each channel has a reliability mode and an ordering guarantee.

| Channel | Reliability | Ordering | Usage |
|---------|------------|----------|-------|
| `INPUT` | Unreliable | Sequenced | Player input packets |
| `TRANSFORM` | Unreliable | Sequenced | High-frequency state updates |
| `GAME_EVENT` | Reliable | Ordered | Damage events, kill feed, score |
| `LOADOUT` | Reliable | Ordered | Class selection, add-on changes |
| `SYSTEM` | Reliable | Ordered | Connection, disconnection, errors |

### Message Types

**Client → Server:**

| Message | Fields | Size (approx.) |
|---------|--------|----------------|
| `PlayerInput` | tick, buttons (u16), mouse_delta (i16, i16), movement (Vec2 half-precision) | 32 bytes |
| `LoadoutSelect` | class (u8), add_on_0 (u8), add_on_1 (u8), add_on_2 (u8) | 8 bytes |
| `ShootInput` | tick, aim_direction (Vec3 f16), weapon_slot (u8) | 12 bytes |

**Server → Client:**

| Message | Fields | Size (approx.) |
|---------|--------|----------------|
| `StateUpdate` | tick, entity_count, per-entity: (entity_id, component_data) | Variable, ~200 bytes/entity |
| `DamageEvent` | target_entity, damage_amount, source_entity, hit_location | 16 bytes |
| `KillFeedEvent` | killer_name, victim_name, weapon_used, headshot (bool) | 64 bytes |
| `SpawnEvent` | entity_id, tick, transform, class, loadout | 48 bytes |
| `ConnectionAck` | client_id, tick | 12 bytes |

### Delta Compression

State updates use delta compression to avoid sending full component data every tick. The server maintains the last acknowledged tick per client per component. On each update, only the fields that changed since the last acknowledged tick are serialized.

For example, if a player's position changed but their health did not, only the position delta is sent. This reduces the average state update from ~200 bytes to ~80 bytes during normal gameplay.

---

## Bandwidth Budget

### Target Budgets

| Metric | Target | Hard Limit |
|--------|--------|------------|
| Server → Client (per client) | 50 KB/s | 80 KB/s |
| Client → Server | 5 KB/s | 10 KB/s |
| Total server bandwidth (12 players) | 600 KB/s | 960 KB/s |

### Breakdown — Server → Client (per client, 12-player match)

| Component | Updates/sec | Bytes/update | Total |
|-----------|------------|-------------|-------|
| Own transform + physics | 20 | 30 | 0.6 KB/s |
| Other 11 players' transforms | 20 | 6 × 11 = 66 | 1.3 KB/s |
| Own combat state (avg) | 2 | 40 | 0.08 KB/s |
| Events (damage, kills, etc.) | 1 | 64 | 0.06 KB/s |
| **Total** | | | **~2.0 KB/s** |

This is well under the 50 KB/s target, leaving headroom for projectile entities, VFX synchronization, and future features.

### Optimization Strategies

1. **Half-precision transforms:** `Vec3` → 6 bytes, `Quat` → 8 bytes. Saves 50% on transform data.
2. **Delta compression:** Only send changed fields. Reduces average state update by ~60%.
3. **Relevant-entity filtering:** Only replicate entities the client can see or interact with (future: interest management).
4. **Component grouping:** Batch small components into a single message to amortize packet overhead.
5. **Quantized inputs:** Mouse delta as `i16` (range ±32767) instead of `f32`. Sufficient precision for aiming.

---

## Input System

### Input Capture

The client captures raw input every frame but packages and sends it at the fixed simulation rate (60 Hz). This ensures prediction and server processing are aligned to the same tick grid.

```rust
#[derive(Component, Serialize, Deserialize)]
pub struct PlayerInput {
    pub tick: u32,
    pub buttons: u16,          // Bitfield: fire, jump, reload, ability1-3, etc.
    pub mouse_delta: (i16, i16), // Mouse movement since last tick
    pub move_axis: (f16, f16),  // WASD normalized direction
}
```

### Button Bitfield Layout

| Bit | Action |
|-----|--------|
| 0 | Fire |
| 1 | Jump |
| 2 | Crouch |
| 3 | Sprint |
| 4 | Reload |
| 5 | Switch Weapon Next |
| 6 | Switch Weapon Prev |
| 7 | Ability 1 |
| 8 | Ability 2 |
| 9 | Ability 3 |
| 10 | Interact |
| 11–15 | Reserved |

### Input Validation (Server-Side)

The server validates every input packet before applying it:

- **Tick sanity:** The input tick must be within ±10 ticks of the server's current tick. Stale or future inputs are rejected.
- **Movement speed:** The `move_axis` magnitude must not exceed 1.0. Values > 1.0 indicate a speed hack.
- **Fire rate:** The fire button must respect the weapon's cooldown. Rapid-fire inputs beyond the weapon's rate of fire are ignored.
- **Rate limiting:** No more than 60 input packets per second per client. Excess packets are dropped.

---

## Physics Synchronization

### Avian 3D Integration

Both client and server run the same Avian 3D physics simulation with identical configuration. The physics step runs at 60 Hz (`FixedUpdate`) on both sides. This ensures that given the same inputs, the simulation produces the same outputs (deterministic within floating-point tolerances).

### Determinism Challenges

Floating-point determinism is not guaranteed across different CPUs. Minor discrepancies accumulate over time. This is why prediction correction exists — it handles the inevitable drift between client and server physics.

To minimize drift:

- Both client and server use the same `TimestepMode::Fixed(Hz(60.0))`.
- Physics substeps are fixed at 1 (no variable substeps).
- Collision margins and solver iterations are identical.
- No random numbers in physics systems (use a seeded RNG stored in a resource).

### Physics Component Replication

| Component | Replicated? | Notes |
|-----------|------------|-------|
| `Transform` | Yes | Server-authoritative, predicted on owning client |
| `LinearVelocity` | Yes | Owning client only (for prediction accuracy) |
| `AngularVelocity` | Yes | Owning client only |
| `RigidBody` | No | Static configuration, set at spawn |
| `Collider` | No | Static configuration, set at spawn |
| `Mass` | No | Static configuration |
| `Friction` | No | Static configuration |
| `Restitution` | No | Static configuration |
| `ExternalForce` | No | Applied per-tick, not replicated |
| `ExternalImpulse` | No | Applied per-tick, not replicated |

Forces and impulses are not replicated because they are applied as a consequence of inputs, which both sides process independently. The server's authoritative transform/velocity state corrects any accumulated drift.

---

## Combat Networking

### Damage Flow

```
Shooter Client              Server                    Target Client
     |                        |                            |
     |-- ShootInput (tick N) >|                            |
     |                        |                            |
     |                        | Rewind to tick N            |
     |                        | Raycast against rewound     |
     |                        | target position             |
     |                        |                            |
     |                        |-- DamageEvent (target) --->|
     |                        |                            |
     |                        |-- KillFeedEvent (all) ---->|  (if lethal)
     |                        |                            |
     |<- Health update ------|                            |
     |    (shooter HUD)       |                            |
```

### Health Replication

`Health` is replicated to all clients so that nameplates and team UI can display health bars. However, the exact damage number is only sent to the owning client via `DamageEvent`. Observers see the health bar decrease but do not see the damage number.

### Death and Respawn

- **Death** is server-authoritative. When `Health` reaches 0, the server sets a `Dead` marker component and sends a `KillFeedEvent` to all clients.
- **Respawn** is triggered by the server after a configurable delay (default: 5 seconds). The server resets `Health` to `MaxHealth`, teleports the entity to a spawn point, and removes the `Dead` marker.
- The client cannot initiate respawn. This prevents spawn manipulation.

### Weapon Switching

- The client predicts the weapon switch animation for responsiveness.
- The actual `ActiveWeapon` component change is server-authoritative.
- If the server rejects the switch (e.g., the weapon slot is empty), the client plays a cancellation animation.

---

## Error Handling and Recovery

### Network Conditions

| Condition | Detection | Response |
|-----------|----------|----------|
| Packet loss | Sequence gaps in received packets | lightyear handles retransmission for reliable channels; unreliable channels skip missing data |
| High latency (> 200ms) | RTT measurement per packet | Increase interpolation delay; warn player via HUD indicator |
| Jitter (variance > 50ms) | Rolling standard deviation of RTT | Increase client-side buffer for interpolated entities |
| Connection lost | No packets for 10 seconds | Server removes player; client shows "Connection Lost" screen |
| Server crash | Connection reset | Client shows error and returns to main menu |

### State Recovery

If a client falls significantly behind the server (e.g., after a network hiccup), a full state sync is performed:

1. Server sends a `FullStateSnapshot` containing all replicated entities and their current component values.
2. Client clears all predicted and interpolated entity state.
3. Client reconstructs entities from the snapshot.
4. Client resumes normal prediction/interpolation from the snapshot tick.

This is expensive but ensures the client can recover from any desync without restarting the connection.

---

## Interest Management (Future)

Current implementation replicates all entities to all clients. For a 6v6 match on a medium-sized map, this is acceptable. For larger maps or higher player counts, we will implement interest management using lightyear's room system.

### Room-Based Interest

- The map is divided into **rooms** (regions/sections).
- Each room is a lightyear `Room` entity.
- Player entities are added to rooms based on their current position.
- Clients only receive replication updates for entities in rooms they are subscribed to.
- A client is subscribed to: the room they are in + adjacent rooms (so entities appear before they enter the client's field of view).

### Migration

When a player crosses a room boundary:

1. Server removes the player from the old room and adds them to the new room.
2. The client receives replication data for entities in the new adjacent rooms.
3. The client stops receiving updates for entities that are no longer in any subscribed room.
4. Entities leaving the client's interest set are smoothly faded out or transitioned (not snapped).

This system reduces per-client bandwidth from O(all players) to O(nearby players), which is critical for scaling beyond 12 players.

---

## Testing and Debugging

### Network Simulation Tools

lightyear provides a network condition simulator for testing. During development, we configure:

- **Latency:** Simulate 50ms, 100ms, 200ms round-trip times.
- **Packet loss:** Simulate 1%, 5%, 10% packet loss rates.
- **Jitter:** Simulate ±20ms, ±50ms, ±100ms jitter.

These are configured via the `LinkConditioner` in lightyear:

```rust
let conditioner = LinkConditioner::new()
    .with_latency(Duration::from_millis(100))
    .with_packet_loss(0.05)
    .with_jitter(Duration::from_millis(20));
```

### Debug Overlays

The client includes a debug overlay (toggled with F3) that displays:

- Current RTT (ping)
- Packet loss rate
- Tick difference (client vs server)
- Prediction error magnitude (how far off the prediction was on the last correction)
- Number of rollbacks in the last second
- Bandwidth usage (bytes/sec sent and received)

### Recording and Playback

The server can record all inputs and state changes to a file. This recording can be replayed to reproduce desyncs, verify hit registration, and debug anti-cheat false positives. This is critical for QA.

---

## See Also

| Document | Description |
|----------|-------------|
| [Architecture Overview](architecture_overview.md) | High-level system architecture, ECS structure, plugin organization |
| [Systems Design](systems_design.md) | Detailed system descriptions, scheduling, and dependencies |
| [Crate Structure](crate_structure.md) | Crate layout, dependency graph, feature flags |
