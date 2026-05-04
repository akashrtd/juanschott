# JuanSchott — Development Roadmap Overview

**Document Version:** 1.0  
**Last Updated:** 2026-05-05  
**Status:** Planning  

---

## 1. Introduction

This document provides the master development roadmap for **JuanSchott**, a fast-paced arena shooter built with Bevy 0.18 and Avian 3D 0.6. The roadmap is organized into five sequential phases, each culminating in a playable, testable milestone. The project follows a strict philosophy: **every phase ships something you can play.** There are no invisible infrastructure phases. No phase exists solely to lay groundwork that only becomes tangible later. Each phase adds visible, interactable features on top of the previous one, ensuring continuous playable feedback throughout development.

The game targets competitive 1v1 and small-team arena combat with a distinctive low-poly, high-finish visual style blending Eastern temple aesthetics with alien science-fiction elements. Movement is deep and expressive; time-to-kill is low; weapon variety and an add-on modification system create strategic depth.

---

## 2. Development Philosophy

### 2.1 Playable Milestones

Every phase delivers a build that can be launched, played, and evaluated. This is not optional — it is the organizing principle of the entire roadmap. If a phase cannot be played end-to-end upon completion, the phase definition is wrong and must be re-scoped.

This philosophy serves three purposes:

1. **Motivation.** Contributors can see and feel progress. A playable build is worth a thousand status reports.
2. **Validation.** Design assumptions are tested early. If movement does not feel right in Phase 1, we learn before building combat on top of it.
3. **Parallelism.** A playable build at each phase boundary allows multiple contributors to work on different aspects simultaneously without stepping on each other.

### 2.2 Sequential Dependencies

The phases are strictly ordered: **Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5.** Each phase depends on the deliverable of the previous phase. You cannot build combat without movement. You cannot build multiplayer without combat. You cannot polish rendering without a functional multiplayer game. You cannot ship content without all of the above.

However, within each phase, many tasks can be parallelized. See Section 6 for contribution guidance.

### 2.3 Version Naming

Each phase maps to a version number. Upon completion of a phase, the build is tagged accordingly:

| Phase | Version | Significance |
|-------|---------|--------------|
| Phase 1 | v0.1 | Movement Prototype |
| Phase 2 | v0.2 | Combat Prototype |
| Phase 3 | v0.3 | Multiplayer Alpha |
| Phase 4 | v0.4 | Visual Polish Alpha |
| Phase 5 | v0.5 | Beta |

Version v0.5 represents the beta milestone: the game is feature-complete and content-sufficient for closed beta testing. Post-beta work (optimization, additional content, community feedback integration) is outside the scope of this roadmap.

---

## 3. Phase Summary

| Phase | Scope | Primary Deliverable | Estimated Duration | Version |
|-------|-------|-------------------|-------------------|---------|
| **1. Movement + Test Map** | Player locomotion, camera, test arena | Player can walk, run, jump, double-jump, slide around a test arena at 120fps | 4–6 weeks | v0.1 |
| **2. Combat** | Weapons, health, damage, death, respawn, add-ons (basic), wall-run | Two players can fight locally with full weapon variety, low TTK, health regen | 6–8 weeks | v0.2 |
| **3. Multiplayer** | lightyear netcode, server/client split, prediction, lag compensation, matchmaking basics | Playable 1v1 online match with acceptable latency compensation | 8–10 weeks | v0.3 |
| **4. Rendering** | Custom shaders, outline post-process, VFX, color grading, lighting | Game visuals match art direction document at target framerates | 6–8 weeks | v0.4 |
| **5. Content** | 3 maps, full weapon/add-on rosters, game modes, HUD, sound, menus | Complete game loop from menu to match to score screen. Beta-ready. | 8–10 weeks | v0.5 |

**Total Estimated Duration:** 32–42 weeks (approximately 8–10 months)

These estimates assume a small team (2–4 active contributors) working part-time. A full-time dedicated team could compress this significantly. The estimates include buffer for iteration and playtesting within each phase.

---

## 4. Phase Dependency Graph

```
Phase 1: Movement + Test Map
    │
    ▼
Phase 2: Combat
    │
    ▼
Phase 3: Multiplayer
    │
    ▼
Phase 4: Rendering
    │
    ▼
Phase 5: Content
```

Each arrow represents a hard dependency. Phase N cannot begin until Phase N-1 has met its acceptance criteria and been formally signed off.

### Why This Order?

- **Movement before Combat:** Combat is meaningless without a character that can move through space. Movement feel is foundational — it must be locked in before we layer combat on top.
- **Combat before Multiplayer:** Netcode requires a known game simulation to replicate. If combat mechanics are still changing, multiplayer work will be wasted on rework.
- **Multiplayer before Rendering:** Visual polish is the last thing to lock down. Rendering work on top of a single-player-only game would need to be re-integrated once multiplayer splits the architecture.
- **Rendering before Content:** Maps and content should be built once the rendering pipeline is final, so that art assets are created against the actual shader and post-processing stack they will ship with.

---

## 5. Risk Assessment

### Phase 1 — Movement + Test Map

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Movement feel does not meet design intent | Medium | High | Allocate extra playtest iteration time. Compare against reference games frame-by-frame. |
| Avian 3D physics edge cases (slope handling, capsule collision) | Medium | Medium | Prototype collision scenarios early. Be prepared to implement custom ground detection if raycast approach is insufficient. |
| Blender-to-Bevy glTF pipeline issues | Low | Medium | Validate the pipeline with a simple cube before investing in complex map geometry. |

### Phase 2 — Combat

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Weapon balance requires extensive iteration | High | Medium | Design weapons with tunable constants from day one. Create a balance tuning spreadsheet early. |
| Wall-run/wall-jump introduces physics complexity | Medium | High | Scope wall-run as a stretch goal within the phase. Core combat must ship without it if time is tight. |
| Split-screen or local multi-player input conflicts | Medium | Medium | Validate input routing for two players early. Consider alternate-turn testing as fallback. |

### Phase 3 — Multiplayer

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| lightyear integration complexity exceeds estimates | High | High | Begin lightyear spike in parallel with late Phase 2. Allocate 2-week buffer. |
| Prediction and rollback are difficult to tune | High | High | Use lightyear's built-in prediction first. Only customize if it proves insufficient. |
| Lag compensation for hitscan weapons is non-trivial | Medium | High | Research FPS lag compensation techniques early. Implement server-side rewind for hit validation. |

### Phase 4 — Rendering

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Custom WGSL shaders have compatibility issues across GPUs | Medium | High | Test on multiple GPU vendors early (NVIDIA, AMD, Intel). Keep a fallback to Bevy's default rendering. |
| Outline post-process performance cost | Low | Medium | Profile early. Use screen-space outline techniques that scale with resolution, not scene complexity. |
| Art direction is subjective and may require iteration | Medium | Medium | Produce reference renders early and get sign-off before full production pass. |

### Phase 5 — Content

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Map design requires extensive iteration for gameplay quality | High | Medium | Block out maps with simple geometry first. Playtest gray-box versions before final art pass. |
| Sound design is under-resourced | Medium | High | Prioritize gameplay-critical sounds (weapons, movement, hit feedback). Ambient and music can be added post-beta. |
| Scope creep in game mode rules | Medium | Medium | Define exact win conditions and scoring rules before implementation. Resist adding features during this phase. |

---

## 6. How to Contribute

Each phase contains tasks that can be parallelized by different contributors. Here is a general guide:

### Parallelization Strategy

**Phase 1:** Project setup is sequential (one contributor). After setup, the test map can be built in Blender while the movement systems are implemented in code. Camera sync and input capture can be developed independently of physics.

**Phase 2:** Weapon systems can be developed in parallel — one contributor can work on hitscan while another works on projectiles. The health/damage system is a dependency for weapon work, so it should be completed first. The wall-run system can be assigned to a dedicated contributor once movement code is stable.

**Phase 3:** The server/client split is foundational and must be done first. Once the architecture is in place, prediction, lag compensation, and matchmaking can be developed in parallel by different contributors. The add-on system upgrade can proceed independently.

**Phase 4:** Shader work, VFX, and lighting are largely independent workstreams that can be assigned to different contributors simultaneously. Color grading and post-processing depend on the base shader being complete.

**Phase 5:** Map design is the most parallelizable work — each of the three maps can be built by a different contributor. Sound design, HUD, and menu systems are independent workstreams. Weapon and add-on roster completion can be split among contributors.

### Contribution Workflow

1. Check the phase document for open tasks.
2. Claim a task by commenting in the project tracker.
3. Implement against the acceptance criteria defined in the phase document.
4. Submit for review. The build must pass the phase's test protocol before merge.

---

## 7. Acceptance Criteria Summary

Each phase has a gating acceptance criteria that must be met before advancing:

- **Phase 1:** Player can navigate the test arena fluently using all movement mechanics (walk, run, jump, double-jump, slide) at 120fps on mid-range hardware.
- **Phase 2:** Two players can engage in a full combat loop with all weapon types, health regeneration, death, and respawn. Low TTK is confirmed through playtesting.
- **Phase 3:** Two players on different machines can play a complete 1v1 duel with acceptable latency compensation and no noticeable desync at 100ms ping.
- **Phase 4:** Screenshots and gameplay footage match the art direction document. The game runs at target framerates (120fps high-end, 60fps mid-range, 30fps low-end).
- **Phase 5:** Complete game loop from main menu to matchmaking to match to score screen. All game modes are playable. The build is ready for closed beta distribution.

---

## 8. Post-Roadmap

After Phase 5 (v0.5 Beta), the project enters a post-roadmap phase that is outside the scope of this document. Expected post-beta activities include:

- Closed beta testing with community feedback collection
- Performance optimization pass based on beta hardware data
- Balance iteration based on competitive play data
- Additional map and content development
- Anti-cheat integration
- Server infrastructure scaling
- Launch preparation (v1.0)

These activities will be planned in a separate document once the beta milestone is reached and the game's actual state can be assessed.

---

## 9. Document References

- `phase1_movement.md` — Phase 1 detailed task breakdown
- `phase2_combat.md` — Phase 2 detailed task breakdown
- `phase3_multiplayer.md` — Phase 3 detailed task breakdown
- `phase4_rendering.md` — Phase 4 detailed task breakdown
- `phase5_content.md` — Phase 5 detailed task breakdown

---

*This roadmap is a living document. Phase estimates and task breakdowns will be updated as development progresses and actual velocity data becomes available.*
