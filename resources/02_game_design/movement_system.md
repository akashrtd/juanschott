# Movement System [SYS-001]

**Document ID:** GDD-SYS-001
**Version:** 1.4.0
**Status:** Design Review
**Author:** Systems Design Team
**Last Updated:** 2026-05-05
**Reviewers:** Lead Gameplay Programmer, Lead Level Designer, Creative Director

---

## Table of Contents

1. [Overview](#1-overview)
2. [Design Philosophy](#2-design-philosophy)
3. [Movement State Machine](#3-movement-state-machine)
4. [Core Mechanics](#4-core-mechanics)
5. [Advanced Mechanics (Phase 2)](#5-advanced-mechanics-phase-2)
6. [Environmental Interactions](#6-environmental-interactions)
7. [Movement Constants Reference](#7-movement-constants-reference)
8. [Game Feel Targets](#8-game-feel-targets)
9. [Technical Implementation Notes](#9-technical-implementation-notes)
10. [Edge Case Registry](#10-edge-case-registry)
11. [Appendix](#11-appendix)

---

## 1. Overview

This document defines every player-facing movement mechanic in ASTRA, including all numerical parameters, state transitions, edge cases, animation hooks, audio triggers, and feel targets. Movement is the foundational layer upon which combat, traversal, and spatial reasoning are built. Every value in this document is authoritative — if a constant appears both here and in another document, this document is the source of truth until a tuning pass overrides it via `06_cross_cutting/tuning_constants.md`.

**Scope:** Player-character ground movement, aerial movement, vertical traversal, and all transitions between these states. Does not cover vehicle movement, mounted movement, or scripted movement sequences.

**Phase gating:**
- Phase 1 (shipping): M-001 through M-005, state machine core, collision response
- Phase 2 (post-launch): M-006 through M-008, wall traversal suite

---

## 2. Design Philosophy

Five pillars govern all movement design decisions. When a tuning conflict arises, resolve it by referencing these pillars in priority order.

| Priority | Pillar | Description |
|----------|--------|-------------|
| 1 | Positioning Tool | Movement exists to gain or deny spatial advantage. It is not an evasion tool. A player who moves should end up somewhere meaningfully different, not dancing in place. |
| 2 | Three-Dimensional Freedom | All three dimensions must be usable at all times during gameplay. Level design and mechanics must never reduce play to a 2D plane for more than 3 consecutive seconds. |
| 3 | Skill Expression | Execution should reward practice. A new player and an expert should traverse the same space at meaningfully different speeds. The skill ceiling is high; the skill floor is low. |
| 4 | Readability | Every movement state must be visually and audibly distinguishable by an opponent within 150ms. No movement trick should look like something it isn't. |
| 5 | Implementability | The state machine must be representable as a finite state automaton with no ambiguous transitions. Every state, transition, and edge case is enumerated here. If it is not enumerated, it does not exist in the design. |

---

## 3. Movement State Machine

### 3.1 State Inventory

The player character occupies exactly one primary movement state at any given frame. Sub-states (e.g., "sprinting while aiming") are tracked via flags, not nested states.

| State ID | State Name | Description |
|----------|-----------|-------------|
| S-01 | IDLE | Standing still on ground. No directional input. |
| S-02 | WALK | Moving on ground at base speed. WASD input active. |
| S-03 | SPRINT | Moving forward at elevated speed. Sprint input held. |
| S-04 | AIR | Airborne. Either from jump, fall, or launch. |
| S-05 | SLIDE | Sliding on ground. Reduced hitbox, committed direction. |
| S-06 | CROUCH | Stationary or slow crouch. Reduced hitbox. |
| S-07 | WALL_RUN | Running along a vertical wall surface. Phase 2 only. |
| S-08 | WALL_JUMP | Momentary launch state after wall jump (resolves to AIR). Phase 2 only. |
| S-09 | MANTLE | Climbing onto a ledge. Locked animation. Phase 2 only. |

### 3.2 State Transition Diagram

```
                    ┌──────────┐
          ┌────────►│   IDLE   │◄───────────────┐
          │         │  [S-01]  │                 │
          │         └────┬─────┘                 │
          │              │ WASD pressed           │ All inputs released
          │         ┌────▼─────┐                 │ (or speed < 0.1 m/s)
          │    ┌───►│   WALK   │◄────────┐       │
          │    │    │  [S-02]  │         │       │
          │    │    └────┬─────┘         │       │
          │    │         │ Sprint held   │       │
          │    │    ┌────▼─────┐         │       │
          │    │    │  SPRINT  │─────────┤       │
          │    │    │  [S-03]  │ Crouch  │       │
          │    │    └────┬─────┘ input   │       │
          │    │         │              │       │
          │    │    ┌────▼─────┐   ┌────▼────┐  │
          │    │    │   AIR    │   │  SLIDE  │  │
          │    │    │  [S-04]  │   │ [S-05]  │──┘
          │    │    └────┬─────┘   └────┬────┘
          │    │         │              │ Slide ends
          │    │    ┌────▼─────┐   ┌────▼────┐
          │    │    │ GROUNDED │   │ CROUCH  │──┘
          │    │    │ (LAND)   │   │ [S-06]  │
          │    │    └────┬─────┘   └─────────┘
          │    │         │
          │    └─────────┘ (speed < 0.1 m/s → IDLE)
          │
          │    ┌─────────────────────────────────────┐
          │    │  PHASE 2 EXTENSIONS                 │
          │    │                                     │
          │    │  AIR ──► WALL_RUN ──► WALL_JUMP     │
          │    │  [S-04]   [S-07]       [S-08]       │
          │    │      │                           │   │
          │    │      └────────── AIR ◄──────────┘   │
          │    │                                     │
          │    │  AIR + ledge ──► MANTLE [S-09]      │
          │    │                    │                │
          │    │               GROUNDED              │
          │    └─────────────────────────────────────┘
          └── Any state ──► IDLE (on death, respawn, cutscene)
```

### 3.3 Transition Table

| From | To | Trigger | Priority |
|------|----|---------|----------|
| IDLE [S-01] | WALK [S-02] | WASD input detected | 1 |
| WALK [S-02] | IDLE [S-01] | All WASD released AND speed < 0.1 m/s | 1 |
| WALK [S-02] | SPRINT [S-03] | Sprint input held + forward (W) input | 2 |
| SPRINT [S-03] | WALK [S-02] | Sprint released OR forward input released | 2 |
| SPRINT [S-03] | SLIDE [S-05] | Crouch input while sprinting | 3 |
| SPRINT [S-03] | AIR [S-04] | Jump input | 3 |
| WALK [S-02] | AIR [S-04] | Jump input | 3 |
| AIR [S-04] | IDLE [S-01] | Ground contact AND speed < 0.1 m/s | 1 |
| AIR [S-04] | WALK [S-02] | Ground contact AND speed >= 0.1 m/s | 1 |
| SLIDE [S-05] | CROUCH [S-06] | Slide duration expired OR speed < 2.0 m/s | 2 |
| SLIDE [S-05] | AIR [S-04] | Jump input OR slide off ledge | 3 |
| CROUCH [S-06] | IDLE [S-01] | Crouch released + no WASD | 1 |
| CROUCH [S-06] | WALK [S-02] | Crouch released + WASD | 1 |
| AIR [S-04] | WALL_RUN [S-07] | Wall detected in range + toward-wall input (Phase 2) | 4 |
| WALL_RUN [S-07] | AIR [S-04] | Duration expired OR input released OR wall ends | 2 |
| WALL_RUN [S-07] | WALL_JUMP [S-08] | Jump input during wall run (Phase 2) | 3 |
| WALL_JUMP [S-08] | AIR [S-04] | Immediate (same frame, resolves to AIR) | 1 |
| AIR [S-04] | MANTLE [S-09] | Ledge detected within mantle range (Phase 2) | 4 |
| MANTLE [S-09] | IDLE [S-01] | Mantle animation complete | 1 |

---

## 4. Core Mechanics (Phase 1)

### 4.1 Walk [M-001]

| Property | Value | Notes |
|----------|-------|-------|
| Mechanic ID | M-001 | |
| Input | WASD | All four cardinal directions relative to camera yaw |
| Activation | Always available when grounded (states S-01, S-02, S-06) | |
| Speed | 4.5 m/s | Maximum horizontal velocity in any ground direction |
| Acceleration | 20.0 m/s² | Time to max speed: 0.225s |
| Deceleration | 30.0 m/s² | Time to zero from max: 0.15s |
| Direction | Relative to camera yaw | Camera pitch does not affect movement direction |
| Head Bob | 0.02m amplitude, 1.8 Hz cycle | Disable-able in settings under "Camera Motion" |
| Footstep Interval | 0.45s | Varies with surface material (see audio table below) |

**Activation Rules:**
- WASD input is processed as a unit vector. Diagonal movement (W+A) is normalized so diagonal speed equals cardinal speed (4.5 m/s), not 6.36 m/s.
- Input is sampled at the start of each frame. Partial frame inputs are not interpolated.

**Constraints:**
- Walking while aiming down sights (ADS): speed reduced to 2.0 m/s. Transition time to ADS-walk speed: 0.15s.
- Walking while reloading: speed reduced to 3.0 m/s. Returns to full speed 0.1s after reload completes.
- Walking while taking damage: no speed penalty. Damage does not affect movement (per Pillar 1).

**Edge Cases:**
- Walking into wall: horizontal velocity zeroed on the collision axis. Perpendicular axis preserved.
- Walking off ledge: transitions to AIR state [S-04] on the frame the ground check fails. Horizontal velocity is preserved exactly.
- Walking on ice (slippery surface): acceleration reduced to 8.0 m/s², deceleration reduced to 5.0 m/s². Same max speed.

**Audio Hooks:**

| Surface Type | Footstep Sound | Volume (dB) | Enemy Hear Range (m) |
|-------------|---------------|-------------|---------------------|
| Concrete | step_concrete_01-04 | -6 | 15 |
| Metal | step_metal_01-03 | -3 | 22 |
| Dirt | step_dirt_01-04 | -12 | 8 |
| Grass | step_grass_01-03 | -18 | 5 |
| Water (shallow) | step_water_01-02 | -9 | 12 |

---

### 4.2 Sprint [M-002]

| Property | Value | Notes |
|----------|-------|-------|
| Mechanic ID | M-002 | |
| Input | Hold Shift (default) or toggle via settings | Toggle mode: press to engage, press again to disengage |
| Activation | From WALK [S-02] only. Requires forward input (W). Cannot activate sideways or backward. | |
| Speed | 7.0 m/s | Forward-only |
| Acceleration | 12.0 m/s² | Ramp-up: 0.583s from walk (4.5) to sprint (7.0). From standstill: 0.35s to walk, then 0.208s ramp to sprint. |
| Deceleration | 25.0 m/s² | When sprint released: 0.1s to walk speed |
| FOV Shift | +5° (90° → 95°) over 0.25s, ease-out curve | Returns to 90° over 0.2s on exit |
| Head Bob | 0.04m amplitude, 2.8 Hz cycle | Double the walk amplitude |
| Footstep Interval | 0.28s | 60% faster than walk |

**Activation Rules:**
- Sprint is direction-locked: only W (forward relative to camera) activates and maintains sprint. Pressing A or D during sprint does NOT cancel sprint but also does NOT change direction — the player continues forward.
- Releasing W while holding Shift drops immediately to WALK [S-02].
- Taking damage does NOT cancel sprint (Pillar 1: movement is positioning, not fragility). However, a "stagger" effect (screen shake of 1.5px amplitude for 0.15s) communicates the hit.

**Constraints:**
- Cannot sprint while aiming down sights (ADS). Attempting to enter ADS while sprinting first decelerates to walk (0.1s), then enters ADS.
- Cannot sprint while firing a weapon that has a movement penalty (heavy weapons: speed clamped to 5.5 m/s during sprint).
- Cannot sprint backward or sideways. Input is validated each frame.

**Edge Cases:**
- Sprint into wall: velocity zeroed on collision axis. Player remains in SPRINT state. Camera FOV stays at 95°. Sprint is maintained as long as Shift+W are held. On paper this is "wasted input" but feels natural.
- Sprint off ledge: enters AIR [S-04]. Horizontal velocity preserved at sprint speed. FOV begins returning to 90° over 0.5s while airborne.
- Sprint while encumbered (carrying objective item): speed reduced to 5.0 m/s. FOV shift reduced to +3°.

**Visual Feedback:**
- Slight motion blur at screen periphery (radial blur, intensity 0.08, disable-able).
- Arm swing animation increases in amplitude by 40% compared to walk.
- Dust particles spawn behind player on appropriate surfaces, emission rate: 12 particles/s.

---

### 4.3 Jump [M-003]

| Property | Value | Notes |
|----------|-------|-------|
| Mechanic ID | M-003 | |
| Input | Space bar | |
| Activation | Grounded only (states S-01, S-02, S-03, S-06). Input buffered for 100ms (6 frames at 60fps) before landing. | |
| Vertical Velocity | 5.5 m/s upward | Initial impulse, applied once on activation |
| Gravity | -9.81 m/s² | Applied every frame while in AIR [S-04] |
| Max Jump Height | ~1.54m | Calculated: v² / (2g) = 5.5² / 19.62 = 1.542m |
| Time to Apex | ~0.56s | v/g = 5.5 / 9.81 |
| Total Air Time | ~1.12s | Apex to landing (symmetric arc from flat ground) |
| Horizontal Velocity | Preserved from ground state | Sprint-jump preserves 7.0 m/s horizontal |
| Coyote Time | 80ms | Player can jump for 80ms after walking off a ledge (ground check was true within last 80ms) |
| Jump Buffer | 100ms | Space bar pressed up to 100ms before landing queues the jump |

**State Transition:** Any grounded state → AIR [S-04]

**Activation Rules:**
- Jump input is consumed on activation. Holding Space does not re-trigger jump on landing — the player must release and re-press.
- Coyote time and jump buffer can stack: a player who presses Space 90ms before landing (within buffer) while having just left a ledge (within coyote time) will jump on the landing frame.
- Jump cannot be activated during SLIDE [S-05] unless the slide has been active for at least 0.15s (prevents accidental slide-jumps on fast input).

**Constraints:**
- Only one regular jump per ground contact. Double jump is a separate mechanic (M-004).
- Cannot jump while in MANTLE [S-09] (animation-locked).
- Cannot jump while grounded and in CROUCH [S-06] (crouch jump is not supported; uncrouch first).

**Edge Cases:**
- Jump into ceiling: vertical velocity zeroed on the frame of ceiling contact. Player remains in AIR state. Gravity resumes. Horizontal velocity unaffected.
- Jump on moving platform: vertical velocity is relative to the platform. Player inherits platform velocity on launch.
- Jump while on slope (>45°): jump direction is surface normal, not straight up. Adjusted vertical velocity = 5.5 * cos(slope_angle).
- Water jump (waist-deep): vertical velocity reduced to 3.0 m/s. Water resistance slows horizontal by 30%.

**Audio Hooks:**

| Surface Type | Jump Sound | Landing Sound |
|-------------|-----------|---------------|
| Concrete | jump_concrete_01 | land_concrete_01 |
| Metal | jump_metal_01 | land_metal_01 |
| Dirt | jump_dirt_01 | land_dirt_01 |
| Grass | jump_grass_01 | land_grass_01 |

Landing sounds trigger at 0.0 volume for soft landings (fall time < 0.3s) and scale to full volume by fall time 1.0s.

---

### 4.4 Double Jump [M-004]

| Property | Value | Notes |
|----------|-------|-------|
| Mechanic ID | M-004 | |
| Input | Space bar while in AIR [S-04] | Must have already used regular jump M-003 this air phase |
| Vertical Velocity | 4.5 m/s upward | 18.2% less than regular jump |
| Horizontal Velocity | Preserved from current air velocity | No directional boost — strictly vertical impulse |
| Additional Height | ~1.03m above the point of activation | v² / (2g) = 4.5² / 19.62 |
| Total Max Height | ~3.04m (from ground) | Achieved by double jumping at the apex of the regular jump |

**State Transition:** AIR [S-04] → AIR [S-04] (double_jump_available flag set to false)

**Activation Rules:**
- Flag `double_jump_available` is set to `true` on landing (on the frame ground contact is detected).
- Flag is set to `false` on activation of M-004.
- Only one double jump per air phase. Landing resets the flag.

**Constraints:**
- Cannot double jump during WALL_RUN [S-07]. Wall run has its own disengage options (wall jump, drop).
- Cannot double jump if the player was launched by an external force (explosion, grapple, knockback). External launches consume the double jump. This is intentional: a player who is knocked airborne has already been "denied" their double jump as part of the knockback's power budget.
- Double jump does not reset on wall jump (M-007). If double jump was already used, wall jumping does not restore it.

**Edge Cases:**
- Double jump immediately after regular jump (within 0.05s): wasted. The player barely gains height because the first jump's upward velocity is replaced, not added to. Design intent: timing the double jump at the apex is the skill-rewarding choice.
- Double jump at the exact apex of regular jump: maximum total height of ~3.04m. This is the optimal timing and should be the "taught" timing in any tutorial content.
- Double jump near wall: can immediately enter WALL_RUN [S-07] if wall is within 0.5m laterally (Phase 2).

**Visual Feedback:**
- Brief radial burst beneath feet on activation (energy discharge effect, 0.25s duration, 0.3m radius, particle count: 16).
- Camera: very subtle downward dip (0.01m) on activation, recovers over 0.1s.
- Player model: legs briefly curl upward then extend, communicating the upward thrust.

---

### 4.5 Slide [M-005]

| Property | Value | Notes |
|----------|-------|-------|
| Mechanic ID | M-005 | |
| Input | Crouch (Ctrl) while in SPRINT [S-03] | Must have forward velocity >= 5.0 m/s |
| Activation | Only from SPRINT [S-03]. Must be moving forward. | |
| Initial Speed | Current sprint velocity (min 7.0 m/s, max 9.0 m/s with momentum) | |
| Deceleration | 3.0 m/s² | Very slow decay — slide is a speed-preserving tool |
| Maximum Duration | 1.2 seconds | Can be canceled early |
| Minimum Speed Threshold | 2.0 m/s | Below this, slide ends and transitions to CROUCH [S-06] |
| Hitbox Height | 1.08m (60% of standing 1.8m) | Capsule collider shortens, center drops by 0.36m |
| Camera Height | Drops to crouch eye level (0.96m from ground) | Transition over 0.12s |
| Camera Tilt | 2.0° in direction of slide | Returns to 0° over 0.15s on slide end |
| Accuracy Penalty | +40% weapon spread | Can shoot during slide but with significant penalty |
| Cooldown | None | Slide can be immediately re-initiated if conditions are met |

**State Transition:** SPRINT [S-03] → SLIDE [S-05]

**Activation Rules:**
- Slide direction is locked to the player's horizontal velocity vector at the moment of activation. Direction cannot be changed during the slide.
- Slide can be canceled by: (a) releasing crouch input, (b) pressing jump, (c) duration expiring, (d) speed dropping below 2.0 m/s.
- Slide cannot be initiated from WALK [S-02] — only from SPRINT [S-03].

**Constraints:**
- Cannot change direction during slide. Input is ignored for direction changes.
- Cannot reload during slide (animation lock). Slide's 1.2s duration vs typical reload of 1.8-2.5s means a slide will often end before a reload could complete anyway.
- Cannot interact with objects (doors, pickups) during slide.

**Edge Cases:**
- Slide off ledge: enters AIR [S-04] on the frame ground check fails. Horizontal velocity preserved at slide speed (up to 9.0 m/s). Gravity begins applying. Slide-jump is a valid and powerful combo.
- Slide into wall: slide ends immediately on collision. Transitions to CROUCH [S-06] pressed against the wall.
- Slide on steep uphill (>30°): slide ends immediately, speed reduced to 2.0 m/s (uphill kills momentum).
- Slide on downhill slope: deceleration reduced to 1.0 m/s². Slide effectively maintains or slightly gains speed on declines. On slopes > 15° decline, deceleration becomes 0.0 m/s² (perpetual slide until duration expires).
- Slide into another player: player collision applies. Slide is treated as a soft collision (no damage, both players are displaced). The sliding player does not lose speed from player collision.

**Audio Hooks:**

| Surface Type | Slide Sound | Volume (dB) | Enemy Hear Range (m) |
|-------------|------------|-------------|---------------------|
| Concrete | slide_concrete_01 | 0 | 25 |
| Metal | slide_metal_01 | +3 | 35 |
| Dirt | slide_dirt_01 | -6 | 15 |
| Grass | slide_grass_01 | -9 | 12 |

**Visual Feedback:**
- Dust/surface particle trail: emission rate 24 particles/s, lifetime 0.6s, spread angle 30° behind player.
- Camera shake: 0.5px amplitude, 8 Hz, duration: entire slide.
- Player model: legs extended forward, body leaning back at 15° from vertical.

---

### 4.6 Crouch [M-006-Core]

| Property | Value | Notes |
|----------|-------|-------|
| Mechanic ID | M-006-Core | (Numbered to distinguish from Phase 2 M-006 Wall Run) |
| Input | Crouch (Ctrl) while grounded and not sprinting | |
| Activation | From IDLE [S-01], WALK [S-02], or at end of SLIDE [S-05] | |
| Speed | 2.0 m/s | Reduced movement speed |
| Hitbox Height | 1.08m (same as slide) | |
| Eye Height | 0.96m from ground | |
| Transition Time | 0.2s to enter crouch, 0.2s to stand | Hitbox interpolates during transition |
| Weapon Accuracy | +15% reduction in spread | Crouching is the accuracy stance |

**Edge Cases:**
- Crouch under low ceiling: cannot uncrouch. Attempting to uncrouch while obstructed plays a "bump head" animation and sound. Player must move to a space with sufficient clearance (1.8m) to stand.
- Crouch on moving platform: crouch is maintained relative to the platform.

---

## 5. Advanced Mechanics (Phase 2)

### 5.1 Wall Run [M-006]

| Property | Value | Notes |
|----------|-------|-------|
| Mechanic ID | M-006 | |
| Input | Movement key toward wall while in AIR [S-04] | Automatic detection + input confirmation |
| Activation | Must be in AIR. Wall surface within 0.5m laterally. Contact height between 1.0m and 3.0m above ground. | |
| Speed | 7.5 m/s along wall surface | Slightly faster than sprint |
| Duration | Maximum 1.5 seconds | Timer starts on wall contact frame |
| Gravity | Reduced to 2.0 m/s² (downward along wall) | Slow descent, not hover |
| Camera Tilt | 5.0° toward wall | Communicates wall-side, returns over 0.2s on exit |
| FOV Shift | +3° (90° → 93°) over 0.2s | Subtler than sprint FOV shift |
| Wall Detection | 0.5m lateral raycast, 3 rays (hip, chest, head height) | All 3 must return wall to validate surface |

**Activation Rules:**
- Wall run requires the player to hold the movement key toward the wall (A/D or W depending on approach angle). Releasing the toward-wall input ends the wall run — player drops into AIR [S-04] with current velocity.
- Wall run maintains forward movement along the wall surface. The player's facing direction rotates to align with the wall tangent over 0.15s.
- Wall run cannot be initiated on the same wall segment twice in one air phase. A "wall segment" is defined by a unique collider. Transferring to a different collider (even on the same visual wall) resets the restriction.

**Constraints:**
- Cannot wall run on surfaces tagged as "non-wall-runnable" (e.g., glass, ice, slippery surfaces).
- Cannot wall run if player velocity toward wall exceeds 12.0 m/s (too fast — would "slam" into wall, not run on it).
- Cannot wall run while carrying heavy objective items.

**Edge Cases:**
- Wall run on outside corner (convex): wall run ends as the wall surface disappears. Player enters AIR [S-04].
- Wall run on inside corner (concave, V-shape): can transfer to the adjacent wall. Timer resets to 1.5s. This IS the wall-run-chain mechanic. Maximum chain: 3 wall segments per air phase (prevents infinite chaining).
- Wall run on moving/breaking wall: if wall moves or breaks, wall run ends immediately. Player enters AIR [S-04].
- Wall run on a wall shorter than the player: wall run activates normally but ends when player reaches the top edge. If the top edge is within mantle range (0.5m above contact point), transitions to MANTLE [S-09].

**Audio:** `wallrun_loop_01` (looping), volume scales with speed. `wallrun_end_01` on exit. Enemy hear range: 30m.

**Visual Feedback:** Particle trail along wall surface (16 particles/s, surface-colored). Subtle speed lines in peripheral vision.

---

### 5.2 Wall Jump [M-007]

| Property | Value | Notes |
|----------|-------|-------|
| Mechanic ID | M-007 | |
| Input | Jump (Space) during WALL_RUN [S-07] or while touching a wall in AIR [S-04] | |
| Activation | Must be in contact with wall surface (within 0.3m) | |
| Lateral Velocity | 7.0 m/s away from wall surface | Along wall normal vector |
| Vertical Velocity | 4.0 m/s upward | |
| Combined Launch Speed | ~8.06 m/s | Vector magnitude of lateral + vertical |
| State Transition | WALL_RUN [S-07] or AIR [S-04] → AIR [S-04] | |

**Activation Rules:**
- Wall jump direction is approximately perpendicular to the wall surface plus an upward component. The exact angle is: launch_direction = wall_normal * 7.0 + world_up * 4.0, then normalized and scaled.
- Wall jump cannot be performed from the same wall segment twice in one air phase (same restriction as wall run).

**Edge Cases:**
- Wall jump between two close walls (corridor, 2-4m wide): can chain wall jumps (ping-pong). Each wall is used once per air phase. A 3m-wide corridor allows approximately 3-4 bounces before the player reaches the ceiling or runs out of new wall segments.
- Wall jump immediately after double jump: both the double jump flag and wall jump flags track independently. Using M-004 does not prevent M-007, and vice versa. However, using M-007 after M-004 does NOT restore the double jump.

**Audio:** `walljump_01` — satisfying bass thump + surface-dependent scrape. Volume: 0 dB. Enemy hear range: 28m.

---

### 5.3 Air Strafe [M-008]

| Property | Value | Notes |
|----------|-------|-------|
| Mechanic ID | M-008 | |
| Input | A/D keys while in AIR [S-04] | |
| Air Control | 30% of ground acceleration (6.0 m/s²) | |
| Maximum Additional Speed | 1.5 m/s above initial horizontal velocity | Hard cap prevents air acceleration exploits |
| Forward/Backward Air Control | 0% | Cannot accelerate forward or decelerate in air |

**Design Rationale:**
30% air control lets players make meaningful trajectory corrections — adjusting a jump to land on a platform that's 1-2m off-target — without enabling the chaotic evasion patterns of arena shooters like Quake or Apex Legends. You commit to your jump direction but have a small correction window. This satisfies Pillar 3 (skill expression — good players will correct their jumps) and Pillar 4 (readability — opponents can predict where a jumping player will land because the trajectory is mostly committed).

**Constraint:** No Quake-style strafe acceleration. Turning the camera while pressing A/D in air does NOT increase speed. Only the raw A/D input magnitude is used for acceleration calculation. This is a deliberate design choice to keep the skill floor accessible while maintaining the skill ceiling through timing-based mechanics (double jump timing, slide-jump combos) rather than input exploit mechanics.

---

## 6. Environmental Interactions

### 6.1 Slope Handling

| Slope Angle | Behavior |
|------------|---------|
| 0° - 15° | Normal ground movement. No speed modification. |
| 15° - 30° | Uphill: speed reduced by 15%. Downhill: speed increased by 10%. |
| 30° - 45° | Uphill: speed reduced by 30%. Downhill: speed increased by 20%. Slide deceleration halved. |
| 45° - 60° | Ramps. Cannot walk up. Can slide down. Slide deceleration zeroed. |
| > 60° | Walls. Cannot stand on. Treated as wall surfaces for wall run detection. |

### 6.2 Surface Materials

| Material | Walk Speed Mult | Sprint Speed Mult | Slide Decel Mult | Sound Category |
|----------|----------------|-------------------|-----------------|---------------|
| Concrete | 1.0 | 1.0 | 1.0 | concrete |
| Metal Grating | 1.0 | 1.0 | 1.2 | metal |
| Dirt | 0.95 | 0.95 | 0.8 | dirt |
| Grass | 0.9 | 0.9 | 0.6 | grass |
| Ice | 0.9 | 0.9 | 0.3 | ice |
| Shallow Water (<0.3m) | 0.8 | 0.75 | 1.5 | water |
| Deep Water (>0.3m) | Swimming mechanics (separate doc) | — | — | water |

### 6.3 Moving Platforms

- Player inherits platform velocity on boarding (frame of contact).
- Platform velocity is added to player velocity each frame while grounded on platform.
- Jumping from a moving platform: player retains platform velocity at the moment of jump. Does not continue to track platform velocity after leaving the surface.
- Slide on a moving platform: slide velocity is relative to the platform. Sliding in the same direction as platform movement yields additive speed.

---

## 7. Movement Constants Reference

All constants in one table for quick reference. These are the authoritative values.

| Constant Name | Value | Unit | Used By |
|--------------|-------|------|---------|
| `WALK_SPEED` | 4.5 | m/s | M-001 |
| `WALK_ADS_SPEED` | 2.0 | m/s | M-001 |
| `WALK_RELOAD_SPEED` | 3.0 | m/s | M-001 |
| `SPRINT_SPEED` | 7.0 | m/s | M-002 |
| `SPRINT_HEAVY_WEAPON_SPEED` | 5.5 | m/s | M-002 |
| `SPRINT_ENCUMBERED_SPEED` | 5.0 | m/s | M-002 |
| `CROUCH_SPEED` | 2.0 | m/s | M-006-Core |
| `GROUND_ACCEL` | 20.0 | m/s² | M-001 |
| `GROUND_DECEL` | 30.0 | m/s² | M-001 |
| `SPRINT_ACCEL` | 12.0 | m/s² | M-002 |
| `SPRINT_DECEL` | 25.0 | m/s² | M-002 |
| `JUMP_VELOCITY` | 5.5 | m/s | M-003 |
| `DOUBLE_JUMP_VELOCITY` | 4.5 | m/s | M-004 |
| `GRAVITY` | 9.81 | m/s² | M-003, M-004, M-008 |
| `WALL_RUN_GRAVITY` | 2.0 | m/s² | M-006 |
| `SLIDE_DECEL` | 3.0 | m/s² | M-005 |
| `SLIDE_MIN_SPEED` | 2.0 | m/s | M-005 |
| `SLIDE_MAX_DURATION` | 1.2 | s | M-005 |
| `SLIDE_ACCURACY_PENALTY` | 0.40 | multiplier | M-005 |
| `WALL_RUN_SPEED` | 7.5 | m/s | M-006 |
| `WALL_RUN_MAX_DURATION` | 1.5 | s | M-006 |
| `WALL_RUN_MAX_CHAINS` | 3 | count | M-006 |
| `WALL_RUN_DETECT_RANGE` | 0.5 | m | M-006 |
| `WALL_JUMP_LATERAL` | 7.0 | m/s | M-007 |
| `WALL_JUMP_VERTICAL` | 4.0 | m/s | M-007 |
| `AIR_CONTROL_FRACTION` | 0.30 | ratio | M-008 |
| `AIR_MAX_BONUS_SPEED` | 1.5 | m/s | M-008 |
| `COYOTE_TIME` | 80 | ms | M-003 |
| `JUMP_BUFFER_TIME` | 100 | ms | M-003 |
| `PLAYER_HEIGHT_STANDING` | 1.8 | m | Collider |
| `PLAYER_HEIGHT_CROUCH` | 1.08 | m | Collider |
| `PLAYER_EYE_HEIGHT_STANDING` | 1.6 | m | Camera |
| `PLAYER_EYE_HEIGHT_CROUCH` | 0.96 | m | Camera |
| `PLAYER_RADIUS` | 0.3 | m | Capsule collider |
| `PLAYER_MASS` | 80.0 | kg | Physics (for external forces) |
| `BASE_FOV` | 90.0 | degrees | Camera |
| `SPRINT_FOV` | 95.0 | degrees | Camera |
| `WALL_RUN_FOV` | 93.0 | degrees | Camera |
| `CROUCH_HITBOX_RATIO` | 0.60 | ratio | Collider |

---

## 8. Game Feel Targets

Game feel is the subjective impression of responsiveness and weight. These are the qualitative targets that tuning should aim for. If numerical tuning conflicts with feel, flag it for design review rather than silently prioritizing numbers.

| Aspect | Target | How Measured |
|--------|--------|-------------|
| Responsiveness | Input-to-action latency <= 50ms (3 frames at 60fps) | Frame counting via slow-motion capture |
| Weight | Character should feel "grounded" — not floaty, not snappy like a twitch shooter | Player survey: 7/10 "weight" rating target in playtest |
| Flow | A player should be able to chain walk → sprint → slide → jump → air strafe without any input gap or "dead frame" where no state is active | Automated test: state machine has no unhandled transitions |
| Readability | An opponent watching at 30fps (spectator) should correctly identify the player's movement state with >= 90% accuracy in a blind test | Playtest: 20 participants, 50 clips each |
| Skill Gap | Expert player traversal of a 100m test course should be 25-35% faster than a new player | Timed playtest: 10 experts vs 10 new players |
| Momentum | Speed changes should feel smooth. No instant speed jumps (except jump, which should feel instant by design) | Velocity graph analysis: no discontinuities > 1.0 m/s except on jump |

---

## 9. Technical Implementation Notes

### 9.1 Frame Rate Independence

All values are specified in real-world units (m/s, m/s², seconds). Implementation must use delta-time scaling. Key formulas:

```
velocity += acceleration * deltaTime
position += velocity * deltaTime + 0.5 * acceleration * deltaTime²
```

For state transitions that occur at exact times (coyote time, jump buffer, slide duration), use accumulated time, not frame counting, to ensure behavior is identical at 30fps, 60fps, and 120fps.

### 9.2 Collision Detection

Player uses a capsule collider (radius 0.3m, height 1.8m standing / 1.08m crouching). Ground check is performed via a downward sphere trace from the capsule's bottom hemisphere, with a tolerance of 0.05m. The ground check returns:
- `is_grounded`: bool
- `ground_normal`: Vector3 (used for slope calculations)
- `ground_material`: SurfaceMaterial enum (used for audio and friction)

### 9.3 Network Replication

Movement is client-predicted with server reconciliation. Client sends input packets at 60Hz containing:
- Timestamp (uint32, ms)
- Input bitmask (WASD + Jump + Crouch + Sprint = 8 bits)
- Camera yaw/pitch (float16 each)
- Client-predicted position (Vector3, quantized to 1cm precision)
- Client-predicted velocity (Vector3, quantized to 0.01 m/s precision)

Server validates movement against its own simulation. If server position diverges from client-predicted position by more than 0.15m, server sends a correction. Client interpolates toward the corrected position over 0.1s to avoid visual snaps.

### 9.4 State Machine Implementation

Recommended: enum-backed finite state machine with explicit transition table. Pseudocode:

```
enum MovementState { IDLE, WALK, SPRINT, AIR, SLIDE, CROUCH, WALL_RUN, WALL_JUMP, MANTLE }

struct Transition {
    from: MovementState
    to: MovementState
    condition: fn(PlayerState) -> bool
    priority: int
}

fn update_state(player: PlayerState, transitions: [Transition]) {
    valid = transitions.filter(t => t.from == player.state && t.condition(player))
    valid.sort_by(|a, b| b.priority - a.priority)  // highest priority first
    if valid.len() > 0 {
        player.state = valid[0].to
        on_enter(player, valid[0].to)
    }
}
```

Transition evaluation order: priority-descending. First matching transition wins. If two transitions have the same priority, the one defined earlier in the table wins (deterministic).

---

## 10. Edge Case Registry

Every known edge case, its expected behavior, and its test case ID.

| EC ID | Description | Expected Behavior | Test Case |
|-------|-------------|-------------------|-----------|
| EC-001 | Walk into corner (two walls) | Velocity zeroed on both collision axes. Player stops. | TC-MOVE-001 |
| EC-002 | Sprint into wall, then uncrouch | No crouch transition. Player is standing. Sprint continues against wall. | TC-MOVE-002 |
| EC-003 | Jump, then land on sloped ground (>30°) | Land. Transition to WALK. Speed adjusted for slope. | TC-MOVE-003 |
| EC-004 | Slide off ledge into another slide surface | Enter AIR on first ledge departure. Re-enter SLIDE on second surface contact if crouch still held and speed >= 5.0 m/s. | TC-MOVE-004 |
| EC-005 | Double jump then wall run then wall jump | Each mechanic tracked independently. All three can be used in one air phase. | TC-MOVE-005 |
| EC-006 | Coyote time + jump buffer overlap | Player jumps on the landing frame. Both windows are valid simultaneously. | TC-MOVE-006 |
| EC-007 | Wall run on a breakable wall | Wall breaks. Wall run ends. Enter AIR. | TC-MOVE-007 |
| EC-008 | Slide into another player | Soft collision. Both displaced. Slider does not lose speed. | TC-MOVE-008 |
| EC-009 | Jump while on a rising elevator | Jump velocity is relative to platform. Player goes higher than a static-ground jump. | TC-MOVE-009 |
| EC-010 | Walk, sprint, and slide all in one fluid input sequence | Walk (W) → Sprint (Shift, 0.35s ramp) → Slide (Ctrl, 1.2s) → Jump (Space). No dead frames. Full state machine coverage. | TC-MOVE-010 |
| EC-011 | Double jump immediately on regular jump (0.05s) | Height gain is minimal (~1.1m total). Player "wastes" the double jump. | TC-MOVE-011 |
| EC-012 | Sprint-jump off a ledge, air strafe, land on moving platform | Sprint velocity preserved on jump. Air strafe adjusts trajectory. Platform velocity added on landing. | TC-MOVE-012 |

---

## 11. Appendix

### A. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-01-15 | Systems Design Team | Initial draft — core mechanics only |
| 1.1.0 | 2026-02-20 | Systems Design Team | Added Phase 2 mechanics (wall run, wall jump, air strafe) |
| 1.2.0 | 2026-03-10 | Systems Design Team | Added environmental interactions, edge case registry |
| 1.3.0 | 2026-04-01 | Systems Design Team | Network replication spec, coyote time added |
| 1.4.0 | 2026-05-05 | Systems Design Team | Final review pass — all values locked for alpha build |

### B. See Also

| Document | Relationship |
|----------|-------------|
| `02_game_design/player_experience.md` | Player experience pillars and feel targets |
| `02_game_design/weapon_system.md` | Movement penalties during weapon actions (ADS, reload, fire) |
| `02_game_design/health_and_damage.md` | Knockback interactions with movement states |
| `06_cross_cutting/tuning_constants.md` | Override file for post-launch tuning passes |
| `03_technical/network_architecture.md` | Client prediction and server reconciliation details |
| `03_technical/animation_state_machine.md` | Animation hooks triggered by movement state transitions |

### C. Glossary

| Term | Definition |
|------|-----------|
| Air Phase | The period from leaving the ground to returning to it. All "per air phase" flags reset on ground contact. |
| Coyote Time | A grace period (80ms) after leaving a ledge where the player can still jump as if grounded. |
| Jump Buffer | A grace period (100ms) before landing where a jump input is queued and executed on the landing frame. |
| Wall Segment | A unique physics collider that defines a wall surface. Different colliders = different segments. |
| Ground Check | A downward sphere trace from the capsule's bottom hemisphere with 0.05m tolerance. |
| Momentum Preservation | Horizontal velocity is maintained across state transitions unless explicitly modified. |

---

*End of document. All values are design-authoritative pending playtest validation. Changes to any constant must be logged in the revision history and communicated to engineering and QA.*
