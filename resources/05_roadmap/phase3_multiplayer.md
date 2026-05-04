# Phase 3 — Multiplayer

**Version Target:** v0.3  
**Estimated Duration:** 8–10 weeks  
**Status:** Planning  
**Dependencies:** Phase 2 (v0.2) complete  

---

## 1. Overview

Phase 3 is the most technically challenging phase of the JuanSchott roadmap. It takes the fully functional local combat prototype from Phase 2 and makes it work **over the internet** between two players on different machines. This requires integrating the `lightyear` networking library, splitting the architecture into a server/client model, implementing client-side prediction with server reconciliation, building lag compensation for hitscan and projectile weapons, creating a loadout selection interface, completing the add-on system with full network replication, and establishing basic matchmaking capabilities.

By the end of Phase 3, two players on separate computers should be able to play a complete 1v1 duel with responsive controls, accurate hit registration, and no noticeable desync at up to 100ms of network latency. This is a high bar, but it is non-negotiable for a competitive arena shooter.

---

## 2. Goals

- Integrate lightyear for authoritative server networking
- Implement server/client architecture split
- Build client-side prediction with server reconciliation
- Implement lag compensation for all weapon types
- Create a loadout selection UI
- Complete the add-on system with full multiplayer support
- Establish basic matchmaking for 1v1 games

---

## 3. Task Breakdown

### 3.1 Lightyear Integration

**Task 3.1: Add lightyear dependency**

Add the `lightyear` networking crate to the project. Configure:
- Transport layer (WebSocket or UDP, based on lightyear's recommended transport for Bevy)
- Protocol versioning for future compatibility
- Connection configuration: tick rate, send rate, buffer sizes

**Task 3.2: Define network protocol**

Define the network message types using lightyear's protocol macros:
- **Input messages:** Player input state (WASD, mouse delta, jump, fire, etc.) sent from client to server every tick
- **State messages:** Entity state updates (position, rotation, velocity, health, weapon state) sent from server to client
- **Event messages:** Damage events, death events, respawn events, weapon switch events, add-on equip events
- **RPC messages:** Chat, ready-up, matchmaking commands

All messages must be serializable and have defined size limits. Large messages (like map data) should use separate channels.

**Task 3.3: Connection lifecycle**

Implement the connection lifecycle:
- Client connects to a known server address
- Server accepts connections and assigns player IDs
- Handshake: client sends player info, server sends initial game state
- Disconnection: clean disconnect handling, entity cleanup on disconnect
- Reconnection: basic reconnection support (stretch goal for v0.3)

**Estimated effort:** 10–14 hours

---

### 3.2 Server / Client Split

**Task 3.4: Architecture refactoring**

Refactor the codebase to support separate server and client execution paths:
- **Shared module:** Components, resources, constants, utility functions used by both server and client
- **Server module:** Game simulation, physics, damage validation, entity spawning
- **Client module:** Input capture, prediction, rendering, audio, UI

The `main.rs` entry point should support running as server-only, client-only, or listen-server (both simultaneously for local testing).

**Task 3.5: Server-authoritative game state**

The server is the single source of truth for:
- All entity positions and velocities
- All health and damage state
- All weapon firing and hit validation
- All game mode state (scores, timers, win conditions)

Clients never modify game state directly. Clients send inputs and render the state provided by the server (with prediction applied locally).

**Task 3.6: Entity replication**

Configure entity replication from server to client:
- Mark components that should be replicated (Transform, Health, WeaponState, etc.)
- Define replication frequency (every tick for critical state, less frequent for non-critical)
- Handle entity spawning and despawning (player joins/leaves, projectile creation/removal)

**Task 3.7: Client-side input sending**

Every tick, the client sends its current `InputState` to the server:
- Serialize the input efficiently (bit-packing for boolean inputs, float compression for mouse delta)
- Include a sequence number for acknowledgment
- Buffer sent inputs locally for prediction reconciliation

**Estimated effort:** 12–16 hours

---

### 3.3 Client-Side Prediction

**Task 3.8: Prediction framework**

Implement client-side prediction so that the player's own character responds immediately to input, even with network latency:
- The client simulates the player entity locally using the same movement and physics code as the server
- When a state update arrives from the server, compare the server's state with the predicted state
- If they match: prediction was correct, continue
- If they differ: rewind to the server state and re-simulate from the corrected state using the buffered input history

**Task 3.9: Input history buffer**

Maintain a ring buffer of recent inputs (last 60–120 ticks):
- Each entry: sequence number + full input state
- When server acknowledges a sequence number, discard older entries
- Use the buffer for re-simulation after server corrections

**Task 3.10: Re-simulation**

When a correction arrives:
- Restore the entity state to the server's corrected state
- Re-apply all buffered inputs from the correction point forward
- The result should closely match what the server computed
- Minimize visual jitter during corrections (smooth the difference rather than snapping)

**Task 3.11: Prediction for other players**

Other players' entities are NOT predicted using input (we do not have their inputs). Instead:
- Use entity interpolation: render other players at a slightly delayed position interpolated between received state updates
- The interpolation delay should be configurable (default: 2–3 ticks of buffer)
- This means other players appear slightly behind their actual server state, which is acceptable and standard for FPS games

**Estimated effort:** 15–20 hours

---

### 3.4 Lag Compensation

**Task 3.12: Server-side history buffer**

Maintain a history of entity transforms on the server:
- Store the last 30–60 ticks of transform data for all entities
- Each history entry is indexed by tick number
- This allows the server to "rewind" to see where entities were in the past

**Task 3.13: Hitscan lag compensation**

When a client fires a hitscan weapon:
- The fire command includes the client's current tick number (or the tick the client was on when they fired)
- The server calculates the latency: `current_server_tick - client_tick_at_fire = latency_in_ticks`
- The server rewinds entity positions by that many ticks
- The server performs the raycast against the rewound positions
- This ensures the hit test matches what the client saw on their screen

**Task 3.14: Projectile lag compensation**

Projectiles are simpler because they exist in the game world and their position is authoritative:
- Spawn the projectile on the server when the fire command is received
- The projectile's initial position is the server-authoritative muzzle position
- There will be a slight delay between when the client sees the shot and when the server spawns it — this is acceptable for projectile weapons
- Optionally, spawn the projectile on the client immediately (with a visual-only predicted projectile) and correct to the server projectile when the state update arrives

**Task 3.15: Beam lag compensation**

Beam weapons use continuous damage, so lag compensation works differently:
- The server receives "beam active" input from the client each tick
- The server applies the beam damage using the client's view direction at the corresponding historical tick
- Rewind the target entity positions for the damage check, similar to hitscan
- The beam visual on the client is predicted: the client renders the beam locally and the server validates damage

**Estimated effort:** 12–16 hours

---

### 3.5 Loadout Selection UI

**Task 3.16: Loadout data structure**

Define the loadout data model:
- A loadout specifies: primary weapon, secondary weapon, add-ons per weapon
- Players can define multiple loadout presets (e.g., 3 loadout slots)
- Loadouts are validated (no invalid weapon/add-on combinations)

**Task 3.17: Loadout selection screen**

Create a Bevy UI-based loadout selection screen:
- Display available weapons with stats
- Display available add-ons organized by slot type
- Drag-and-drop or click-to-equip interface
- Preview of weapon stats with current add-on configuration
- Confirm button to lock in the loadout

**Task 3.18: Loadout networking**

When a player confirms their loadout:
- Send the loadout to the server
- Server validates the loadout (prevents cheating by sending invalid loadouts)
- Server distributes loadout selections to both players
- Once both players confirm, the match begins (or enters a countdown)

**Estimated effort:** 10–14 hours

---

### 3.6 Add-On System (Full Implementation)

**Task 3.19: Add-on data completion**

Expand the add-on system from Phase 2's basic implementation:
- Define the full add-on roster (target: 8–12 add-ons covering all slot types)
- Each add-on has: name, description, slot type, stat modifications, visual indicator
- Stat modifications are expressed as multipliers or deltas on base weapon stats

**Task 3.20: Add-on replication**

Ensure add-on state is replicated over the network:
- When a player equips an add-on, the server validates and acknowledges it
- The add-on state is part of the replicated player entity
- Other players can see equipped add-ons on opponent weapons (visual indicator in Phase 4)

**Task 3.21: Add-on balance validation**

With the full add-on roster, conduct balance testing:
- Verify that no add-on combination makes a weapon overpowered
- Verify that all add-ons have meaningful gameplay impact
- Adjust stat multipliers based on playtest feedback

**Estimated effort:** 8–10 hours

---

### 3.7 Basic Matchmaking

**Task 3.22: Matchmaking server**

Create a simple matchmaking system:
- A lobby server that tracks available players
- Players connect to the lobby and enter a queue
- When two players are in the queue, the lobby assigns them to a match
- The lobby informs both players of the server address to connect to

**Task 3.23: Match flow**

Implement the full match flow:
1. Player connects to lobby
2. Player enters matchmaking queue
3. Match found → both players redirected to game server
4. Players connect to game server
5. Loadout selection phase
6. Countdown and match start
7. Match plays until win condition is met
8. Score screen (basic — log to console or simple UI)
9. Return to lobby

**Task 3.24: Listen server support**

For development and casual play, support a listen-server mode:
- One player hosts (runs server + client)
- Other players connect to the host's IP address
- This bypasses the matchmaking server for direct-play scenarios
- The host's client has a local connection with zero simulated latency for testing

**Estimated effort:** 8–12 hours

---

## 4. Task Summary

| Task Group | Tasks | Estimated Hours |
|-----------|-------|----------------|
| Lightyear Integration | 3.1, 3.2, 3.3 | 10–14 |
| Server/Client Split | 3.4, 3.5, 3.6, 3.7 | 12–16 |
| Client-Side Prediction | 3.8, 3.9, 3.10, 3.11 | 15–20 |
| Lag Compensation | 3.12, 3.13, 3.14, 3.15 | 12–16 |
| Loadout Selection UI | 3.16, 3.17, 3.18 | 10–14 |
| Add-On System Full | 3.19, 3.20, 3.21 | 8–10 |
| Basic Matchmaking | 3.22, 3.23, 3.24 | 8–12 |
| **Total** | **24 tasks** | **75–102 hours** |

At 8–12 productive hours per week, this maps to the 8–10 week estimate. This is the most time-intensive phase. The estimate includes buffer for the inherent complexity of multiplayer networking.

---

## 5. Acceptance Criteria

Phase 3 is complete when **all** of the following are true:

1. **Network Connection:** Two clients can connect to a server and maintain a stable connection for the duration of a match. No unexpected disconnects under normal conditions.

2. **Player Replication:** Both players can see each other moving in real-time. Movement appears smooth with no visible stuttering at up to 100ms ping. Entity interpolation for the remote player is functional.

3. **Prediction:** The local player's movement feels identical to Phase 2 (single-player). No input lag. Server corrections are infrequent and subtle when they occur.

4. **Hitscan Lag Compensation:** At 100ms ping, hitscan shots register correctly. If the client's crosshair is on an opponent on their screen, the shot hits on the server. False negatives (missed hits that should have connected) are minimal.

5. **Projectile Weapons:** Projectiles spawn and travel correctly in multiplayer. Projectile state is replicated. Collision detection works for all projectile types.

6. **Beam Weapons:** Beam damage applies correctly in multiplayer. Lag compensation ensures continuous beam tracking is accurate.

7. **Health/Damage:** All damage is server-authoritative. Health regeneration works. Death and respawn work in multiplayer. Kill attribution is correct (the correct player gets credit for kills).

8. **Loadout Selection:** Players can select weapons and add-ons from the loadout screen. Selected loadouts are reflected in gameplay. Invalid loadouts are rejected.

9. **Add-On System:** All add-ons function correctly in multiplayer. Stat modifications are applied consistently on both client and server. Add-on state is replicated.

10. **Matchmaking:** Players can find each other through the matchmaking system and enter a match. Listen-server mode works for direct connections. The match plays to completion with correct scoring.

11. **Latency Tolerance:** At 100ms ping, the game is playable and competitive. At 50ms ping, the experience is indistinguishable from local play. At 150ms ping, the game is degraded but functional.

---

## 6. Test Protocol

### 6.1 Local Network Test

1. Start a listen server on the development machine.
2. Connect a second client from another machine on the same LAN.
3. Play a full 1v1 match (first to 10 kills or 5 minutes).
4. Verify smooth gameplay, correct hit registration, and no desync.

### 6.2 Simulated Latency Test

1. Add artificial latency to the connection (50ms, 100ms, 150ms).
2. At each latency level:
   - Verify movement prediction feels responsive
   - Fire hitscan weapons at a stationary target — verify hit registration
   - Fire projectile weapons — verify projectile spawning and collision
   - Fire beam weapons — verify continuous damage tracking
   - Engage in combat — verify TTK is consistent with local play

### 6.3 Stress Test

1. Play continuously for 15 minutes.
2. Monitor: memory usage (no leaks), network bandwidth (reasonable), tick rate stability (server maintains consistent tick rate).
3. Verify no desync accumulates over time.

---

## 7. Architecture Decisions

### Authoritative Server

The server is fully authoritative. Clients never modify game state. All damage, physics, and game logic run on the server. The client only predicts its own movement and renders the server's state. This architecture prevents cheating and ensures consistency.

### Tick Rate

The server runs at a fixed tick rate (recommended: 60 ticks/second). Client input is sent every tick. State updates are sent at tick rate or a fraction thereof (e.g., every other tick for non-critical state). The tick rate should be configurable for server performance tuning.

### Prediction Model

The game uses a client-side prediction + server reconciliation model:
- Client predicts its own entity's state based on local input
- Server validates and sends corrections
- Client re-simulates from corrections using buffered input history

This is the standard model for FPS games and is well-supported by lightyear.

### Interpolation Model for Remote Players

Remote players are rendered using entity interpolation:
- The client maintains a buffer of the last N received state updates for each remote entity
- The client renders the entity at a position interpolated between two buffered states
- This introduces a small additional delay (typically 20–50ms) but provides smooth visual movement

---

## 8. Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| lightyear API instability or missing features | Medium | High | Begin integration early. Maintain a fork with patches if needed. Engage with the lightyear community. |
| Prediction re-simulation is computationally expensive | Medium | Medium | Profile re-simulation cost. Optimize by only re-simulating the player entity, not the entire world. |
| Hitscan lag compensation produces unfair-feeling results | Medium | High | Tune the compensation algorithm. Add a maximum rewind limit (e.g., 200ms) to prevent extreme cases. Playtest extensively. |
| Server performance at target tick rate | Low | High | Profile the server simulation. Optimize physics and damage checks. Consider reducing tick rate if necessary. |
| Desync under packet loss | Medium | High | Implement reliable channels for critical state. Use delta compression for state updates. Test with simulated packet loss. |

---

## 9. Exit Criteria

Upon completing Phase 3 and meeting all acceptance criteria:

1. Tag the repository as `v0.3`.
2. Conduct a multiplayer playtest session between two remote machines.
3. Document latency compensation accuracy with measured data at 50ms, 100ms, and 150ms.
4. Review the networking architecture for performance and extensibility.
5. Proceed to Phase 4 (Rendering).
