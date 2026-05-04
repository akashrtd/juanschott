# Performance Targets — JuanSchott

**Document Version:** 1.0  
**Last Updated:** 2026-05-05  
**Status:** Active  

---

## 1. Overview

JuanSchott targets 60fps on integrated graphics hardware from 2012. This is the single most important technical constraint for the project. Every rendering decision, physics configuration, and architectural choice is made in service of this target.

This document defines the hardware tiers the game supports, the per-frame performance budgets, rendering budgets, optimization strategies, profiling tools, and the performance testing protocol. All team members — programmers, artists, and designers — are expected to understand and respect these targets. A frame time regression on Tier 1 hardware is a blocking bug.

---

## 2. Hardware Tiers

JuanSchott is tested against three hardware tiers. Tier 1 is the minimum specification. Tier 2 is the expected specification for most players. Tier 3 is the enthusiast specification where the game should run flawlessly with headroom to spare.

### 2.1 Tier 1 — Potato (Minimum Specification)

| Component | Specification |
|---|---|
| GPU | Intel HD 4000 integrated graphics |
| VRAM | Shared system memory (no dedicated VRAM) |
| RAM | 4 GB DDR3 |
| CPU | Dual-core (e.g., Intel Core i3-3220 @ 3.3GHz) |
| Target Resolution | 1280 × 720 (720p) |
| Target Frame Rate | 60 fps |

**Why this hardware:** The Intel HD 4000 was the integrated GPU in Intel's Ivy Bridge processors, released in 2012. It represents the absolute floor of what can be considered a functional GPU for 3D rendering. If JuanSchott runs at 60fps on this hardware, it will run well on virtually any discrete GPU made in the last decade.

**Key constraints of this tier:**

- No dedicated VRAM — all GPU memory is shared with the CPU, creating bandwidth contention.
- Limited shader core count — complex fragment shaders will be slow.
- No hardware support for modern features (no mesh shaders, no variable rate shading, no hardware ray tracing).
- Driver support is mature but feature-limited (OpenGL 4.0, no Vulkan on older drivers).

### 2.2 Tier 2 — Mid (Expected Specification)

| Component | Specification |
|---|---|
| GPU | NVIDIA GTX 1050 or AMD RX 560 |
| VRAM | 2 GB GDDR5 |
| RAM | 8 GB DDR4 |
| CPU | Quad-core (e.g., Intel Core i5-6500 @ 3.2GHz) |
| Target Resolution | 1920 × 1080 (1080p) |
| Target Frame Rate | 120 fps |

**Why this hardware:** The GTX 1050 and RX 560 represent the most common GPUs in use according to Steam Hardware Survey data for budget gamers. This tier represents the typical player's machine.

### 2.3 Tier 3 — Good (Enthusiast Specification)

| Component | Specification |
|---|---|
| GPU | NVIDIA RTX 3060 or AMD RX 6600 |
| VRAM | 8–12 GB GDDR6 |
| RAM | 16 GB DDR4/DDR5 |
| CPU | Modern 6+ core (e.g., Ryzen 5 5600X) |
| Target Resolution | 2560 × 1440 (1440p) |
| Target Frame Rate | 144 fps |

**Why this hardware:** This tier represents the modern mid-range. JuanSchott should run at the display's maximum refresh rate with no frame drops on this hardware.

---

## 3. Per-Frame Performance Budgets (60fps = 16.67ms)

The following budget allocations are measured at Tier 1 hardware. All times are wall-clock milliseconds per frame.

### 3.1 Budget Breakdown

| System | Budget | Notes |
|---|---|---|
| Game Logic | < 2.0 ms | Input handling, state machines, game rules, networking |
| Physics (Avian) | < 2.0 ms | Collision detection, rigid body integration |
| Rendering | < 10.0 ms | All GPU work: geometry, shading, post-processing |
| UI | < 1.0 ms | HUD rendering, text layout, Bevy UI system |
| Headroom | ~1.67 ms | Buffer for frame spikes, OS scheduling, driver overhead |
| **Total** | **16.67 ms** | **60 fps target** |

### 3.2 Budget Rationale

**Game logic (< 2ms):** JuanSchott's game logic is simple: input → movement → shooting → hit detection → score update. There are no complex AI systems, no pathfinding, and no procedural generation. The logic budget is generous for the game's needs. The primary cost is network synchronization (packet parsing and state reconciliation), which should be under 0.5ms for a game with fewer than 16 players.

**Physics (< 2ms):** Avian's physics step for a scene with 16 player capsules, a single static trimesh collider (the map), and a handful of projectiles should be well under 2ms on a dual-core CPU. The trimesh collider is the most expensive component, but Avian's broadphase (using a sweep-and-prune or similar algorithm) culls most collision pairs before the narrowphase runs.

**Rendering (< 10ms):** This is the largest budget and the one most likely to be exceeded. The rendering budget covers all GPU work: vertex processing, fragment shading, shadow mapping, and post-processing. At 720p on Intel HD 4000, the fill-rate budget is approximately 920K pixels per frame. With the custom flat-color shader (no texture sampling, simple stepped lighting), the fragment shader cost per pixel is minimal. The shadow map pass adds one additional geometry render at 2048×2048, but with no filtering, the shadow lookup is a single texture fetch.

**UI (< 1ms):** Bevy's UI system renders 2D elements using a text-rendering pipeline and quad batching. For JuanSchott's simple HUD (health bar, ammo counter, crosshair, kill feed), this budget is sufficient.

**Headroom (~1.67ms):** This buffer absorbs frame spikes from garbage collection pauses (Rust's allocator is generally fast but not zero-cost), OS scheduling jitter, and driver overhead. If the headroom is consistently consumed, the game will drop frames.

---

## 4. Rendering Budgets

Rendering budgets are specified in terms of GPU resources, not wall-clock time, because GPU time varies by hardware tier. These budgets are defined for Tier 1 at 720p.

### 4.1 Draw Calls

| Metric | Budget | Rationale |
|---|---|---|
| Draw calls per frame | < 500 | Intel HD 4000's driver overhead is ~15µs per draw call. 500 calls × 15µs = 7.5ms driver overhead alone. |

The low-poly art style is the primary mechanism for keeping draw calls low. A map composed of large, flat surfaces with few materials produces far fewer draw calls than a detailed environment with many materials and small objects. Instanced rendering (see Section 6) further reduces draw calls for repeated geometry.

### 4.2 Triangle Count

| Metric | Budget | Rationale |
|---|---|---|
| Triangles per frame | < 100K | Low-poly models and simple map geometry. A typical player model is ~500 triangles. A map is ~20K triangles. 16 players + map = ~28K triangles, well under budget. |

The 100K triangle budget is conservative. Modern GPUs can process millions of triangles per frame. However, the Intel HD 4000's vertex processing throughput is limited, and the budget must account for the shadow map pass (which re-renders all shadow-casting geometry).

### 4.3 Texture Memory

| Metric | Budget | Rationale |
|---|---|---|
| Texture memory | < 64 MB | JuanSchott uses minimal textures. The shadow map (2048×2048 R16) is ~8MB. UI textures are small. The remaining budget is for Bevy's internal render targets. |

The flat-color art style means JuanSchott has almost no texture assets. The 64MB budget is primarily consumed by:

- Shadow map: 2048×2048 × 2 bytes (R16) = ~8MB
- Framebuffer: 1280×720 × 4 bytes (RGBA8) = ~3.5MB
- Post-processing intermediates: ~3.5MB per pass
- Bevy internal textures: ~10–20MB

Total estimated texture memory: ~35–45MB, well under the 64MB budget.

### 4.4 Shadow Map

| Metric | Value |
|---|---|
| Resolution | 2048 × 2048 |
| Format | R16 (depth only) |
| Memory | ~8 MB |
| Cascade Count | 1 |
| Filtering | None (hard-edged shadows) |

A single 2048×2048 shadow map provides adequate shadow quality for the game's map sizes. The hard-edged shadow style eliminates the need for multi-tap filtering, reducing the fragment shader's shadow lookup cost from 4–16 texture fetches to 1.

---

## 5. Optimization Strategies

JuanSchott employs several optimization strategies, some built into Bevy and some implemented in game code.

### 5.1 Frustum Culling

Bevy's visibility system automatically culls entities that are outside the camera's frustum. This is the most impactful optimization for any 3D game: it ensures that only visible geometry is submitted to the GPU.

For JuanSchott, frustum culling eliminates approximately 50–80% of map geometry per frame (depending on the player's position and view direction). This directly reduces draw calls and triangle count.

Frustum culling is built into Bevy and requires no game code to function. Every entity with a `Mesh3d` and `Aabb` component (computed automatically from the mesh) is culled.

### 5.2 Occlusion Culling

Bevy 0.18 provides an occlusion culling system that eliminates entities hidden behind other geometry. This is more aggressive than frustum culling — an entity can be within the frustum but occluded by a wall or large object.

Occlusion culling is particularly valuable for JuanSchott's indoor maps, where large wall surfaces occlude significant portions of the map. It is less valuable for open arenas where most geometry is visible from any position.

Bevy's occlusion culling uses a software rasterizer to determine visibility. The cost of the occlusion pass is typically 0.1–0.3ms, which is easily recouped by the draw calls it eliminates.

### 5.3 LOD (Level of Detail)

Bevy's `visibility_range` feature allows entities to be shown or hidden based on distance from the camera. This can be used for LOD: high-detail models are shown at close range, and low-detail models are shown at long range.

**Current status:** LOD is not needed at launch. JuanSchott's low-poly models are already at their minimum detail level. Reducing triangle count further would produce visibly degraded silhouettes. If future maps are larger and require distant geometry to be simplified, `visibility_range` is available without any engine changes.

### 5.4 Instanced Rendering

Instanced rendering allows the GPU to draw multiple copies of the same mesh in a single draw call. This is critical for repeated geometry like pillars, crates, platform segments, and other props that appear multiple times in a map.

Bevy's mesh instancing is automatic when entities share the same mesh and material. When the renderer detects that multiple entities use identical mesh+material combinations, it batches them into a single instanced draw call.

**Impact:** A map with 20 identical pillars using the same mesh and material renders in 1 draw call instead of 20. This is the single most effective draw-call optimization for JuanSchott's art style, where repeated geometry is common.

### 5.5 No Screen-Space Effects

JuanSchott deliberately excludes all screen-space effects:

- **SSAO (Screen-Space Ambient Occlusion):** 3–5ms on Tier 1 hardware. Aesthetic clash with flat-color style. Replaced by the shader's fixed ambient term.
- **SSR (Screen-Screen Reflections):** 4–8ms on Tier 1 hardware. Not applicable — the game has no reflective surfaces.
- **SSGI (Screen-Space Global Illumination):** 5–10ms on Tier 1 hardware. Not needed — single directional light, fixed ambient.

The combined savings from excluding these effects is 12–23ms per frame — more than the entire rendering budget.

### 5.6 Forward Rendering

Forward rendering is chosen over deferred rendering for lower GPU requirements. See `rendering_pipeline.md` for the full rationale. The key performance benefit is reduced memory bandwidth: forward rendering writes each fragment once, while deferred rendering writes to multiple G-buffer targets and reads them back during the lighting pass.

### 5.7 FXAA Instead of MSAA

FXAA is used instead of MSAA for anti-aliasing. FXAA costs ~0.5ms (single full-screen pass) compared to MSAA 4x's ~2–4ms (per-sample shading during geometry pass). See `rendering_pipeline.md` for the full comparison.

---

## 6. Profiling Tools

Profiling is essential for maintaining performance targets. The following tools are used to identify and diagnose performance issues.

### 6.1 Bevy Built-In Diagnostics

Bevy provides real-time diagnostic information via the `DiagnosticsPlugin`:

| Metric | Source | Usage |
|---|---|---|
| FPS | `FrameTimeDiagnosticsPlugin` | Overall frame rate |
| Frame time (ms) | `FrameTimeDiagnosticsPlugin` | Per-frame duration |
| Entity count | `EntityCountDiagnosticsPlugin` | Total spawned entities |
| System execution time | `tracing` spans | Per-system timing |

These diagnostics are displayed in-game during development via an overlay (toggled with a debug key). They provide a quick sanity check during gameplay testing.

### 6.2 tracing Crate

Bevy uses Rust's `tracing` crate for structured logging and timing. Every Bevy system is instrumented with tracing spans that record execution time. The output can be directed to:

- **Console output (development):** `RUST_LOG=debug cargo run` shows system timing in real-time.
- **Chrome Trace (analysis):** `tracing-chrome` outputs a `.json` file that can be loaded in Chrome's `chrome://tracing` viewer. This provides a visual timeline of system execution, showing which systems run in parallel and which are bottlenecks.
- **Perfetto (analysis):** Compatible with Chrome Trace format.

The tracing output is the primary tool for identifying which Bevy systems are consuming the most CPU time.

### 6.3 RenderDoc

RenderDoc is a frame-capture tool for GPU profiling. It captures a single frame's entire rendering pipeline, allowing inspection of:

- Every draw call and its associated GPU state.
- Render targets at each pipeline stage.
- Shader execution and resource bindings.
- GPU timing per draw call and per render pass.

**Usage for JuanSchott:**

1. Launch the game with RenderDoc attached.
2. Capture a frame during typical gameplay.
3. Inspect the draw call list to identify expensive passes.
4. Check shadow map rendering time.
5. Verify post-processing pass costs.
6. Confirm triangle counts and draw call counts match budgets.

RenderDoc is the definitive tool for answering "why is this frame slow on the GPU?"

### 6.4 Intel VTune

Intel VTune is a CPU profiling tool that provides hardware-level performance analysis:

- **Hotspot analysis:** Identifies the functions consuming the most CPU time.
- **Thread analysis:** Shows thread utilization and synchronization bottlenecks.
- **Memory access analysis:** Identifies cache misses and memory bandwidth bottlenecks.

**Usage for JuanSchott:** VTune is used for deep CPU profiling when the tracing output identifies a system as a bottleneck but the cause is unclear. For example, if the physics system is consuming 3ms (above its 2ms budget), VTune can determine whether the cost is in the broadphase, narrowphase, or solver.

### 6.5 Windows Performance Analyzer / PIX

On Windows, Windows Performance Analyzer (WPA) and PIX (for DirectX) provide similar CPU and GPU profiling capabilities. PIX is particularly useful for GPU timing on Windows, as it integrates with DirectX 12 (which Bevy uses on Windows via wgpu).

---

## 7. Performance Testing Protocol

Performance testing is a continuous process, not a one-time event. The following protocol ensures that performance regressions are caught early and fixed before they reach players.

### 7.1 Milestone Testing

At every project milestone (defined as every 2 weeks during active development), the following test is performed:

1. **Deploy to Tier 1 hardware.** Run the game on the designated Tier 1 test machine (Intel HD 4000, 4GB RAM, dual-core).
2. **Record frame times.** Run a 5-minute gameplay session on each map, capturing frame times using Bevy's diagnostics output logged to a file.
3. **Compute statistics.** For each map session, compute:
   - Average frame time (ms)
   - 95th percentile frame time (ms)
   - 99th percentile frame time (ms)
   - Minimum FPS (1-second average)
   - Number of frames exceeding 20ms (below 50fps)
4. **Compare against thresholds:**
   - Average frame time must be < 16.67ms (60fps).
   - 95th percentile frame time must be < 18ms (~55fps).
   - No more than 1% of frames may exceed 20ms (< 50fps).
   - The minimum 1-second average FPS must be > 55fps.

### 7.2 Regression Policy

If Tier 1 performance drops below the thresholds defined above:

1. **The issue is a blocking bug.** No other work proceeds until it is fixed.
2. **The developer who introduced the regression is responsible for fixing it.** If the regression was introduced in a specific commit, that commit's author investigates and resolves it.
3. **The fix must restore performance to within the budget.** A "close enough" fix is not acceptable. The numbers must meet the thresholds.
4. **The regression must be documented.** The root cause, the fix, and the lesson learned are recorded to prevent recurrence.

### 7.3 Automated Frame Time Capture

During development, a Bevy system logs frame times to a CSV file:

```csv
frame_index,frame_time_ms,timestamp
0,14.2,0.000
1,15.1,16.667
2,14.8,33.333
...
```

This file is captured during every manual playtest on Tier 1 hardware. A script analyzes the CSV and flags any sessions where the thresholds are violated.

### 7.4 Tier 2 and Tier 3 Testing

Tier 2 and Tier 3 hardware are tested less frequently (once per release milestone) but with stricter thresholds:

| Tier | Average FPS Threshold | 95th Percentile FPS Threshold |
|---|---|---|
| Tier 1 (720p) | ≥ 60 fps | ≥ 55 fps |
| Tier 2 (1080p) | ≥ 120 fps | ≥ 110 fps |
| Tier 3 (1440p) | ≥ 144 fps | ≥ 130 fps |

Tier 2 and Tier 3 failures are not blocking bugs but are treated as high-priority issues to be resolved in the next milestone.

### 7.5 Performance Test Scenarios

Testing is performed in the following scenarios to cover all gameplay conditions:

| Scenario | Description | Why It Matters |
|---|---|---|
| Empty map | No players, standing still | Baseline — measures rendering cost with no game logic or physics |
| Full server (16 players) | 16 players in combat | Peak load for game logic, physics, and rendering |
| Weapon firing | Continuous weapon firing with projectile spawning | Tests projectile spawning overhead and rendering |
| Spawn/despawn burst | Multiple players respawning simultaneously | Tests entity spawning overhead |
| Long match | 30-minute continuous play | Tests for memory leaks and performance degradation over time |

The long match test is critical: it catches memory leaks (which manifest as increasing frame times) and resource accumulation (unhandled projectile cleanup, unreleased collider references, etc.). If frame time increases by more than 10% over a 30-minute session, there is a leak.

---

## 8. Performance Budget Summary

### 8.1 Quick Reference — Per-Frame Budgets at 60fps

```
Total frame budget:    16.67 ms
├── Game Logic:        < 2.0 ms
├── Physics (Avian):   < 2.0 ms
├── Rendering:         < 10.0 ms
├── UI:                < 1.0 ms
└── Headroom:          ~1.67 ms
```

### 8.2 Quick Reference — Rendering Budgets

```
Draw calls:            < 500 per frame
Triangles:             < 100K per frame
Texture memory:        < 64 MB
Shadow map:            2048×2048, single cascade, hard edges
Post-processing:       ACES + bloom + FXAA + color grading (all built-in)
Anti-aliasing:         FXAA (no MSAA)
Screen-space effects:  None (no SSAO, SSR, SSGI)
Rendering path:        Forward (not deferred)
```

### 8.3 Quick Reference — Hardware Targets

```
Tier 1 (Potato):  Intel HD 4000, 4GB RAM, dual-core   → 720p @ 60fps
Tier 2 (Mid):     GTX 1050 / RX 560, 8GB RAM, quad    → 1080p @ 120fps
Tier 3 (Good):    RTX 3060 / RX 6600, 16GB RAM, 6+    → 1440p @ 144fps
```

---

## 9. Summary

JuanSchott's performance targets are anchored to a single metric: 60fps on Intel HD 4000 integrated graphics at 720p. This target drives every technical decision in the project, from the forward rendering pipeline and flat-color shader to the exclusion of screen-space effects and the use of FXAA over MSAA. The per-frame budgets allocate 10ms for rendering, 2ms for physics, 2ms for game logic, and 1ms for UI, with 1.67ms of headroom. Performance is tested at every milestone on Tier 1 hardware, and regressions are treated as blocking bugs. The low-poly art style is not just an aesthetic choice — it is the foundation of the game's performance strategy, enabling draw-call counts under 500 and triangle counts under 100K per frame.
