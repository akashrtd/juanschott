# Phase 4 — Rendering

**Version Target:** v0.4  
**Estimated Duration:** 6–8 weeks  
**Status:** Planning  
**Dependencies:** Phase 3 (v0.3) complete  

---

## 1. Overview

Phase 4 transforms the functional multiplayer game into a visually polished product that matches JuanSchott's art direction. The game should look like a **low-poly, high-finish** experience that blends Eastern temple aesthetics with alien science-fiction elements. This means custom shaders, outline post-processing, weapon visual effects, per-map color grading, player model differentiation, and environmental lighting — all while maintaining the performance targets established in earlier phases.

Rendering is intentionally placed after multiplayer. Visual polish on a single-player game would need to be re-integrated once the server/client split changes the rendering pipeline. By this phase, the architecture is stable, and rendering work will carry directly into the final product.

By the end of Phase 4, screenshots and gameplay footage should match the art direction document. The game should run at 120fps on high-end hardware, 60fps on mid-range hardware, and 30fps on low-end hardware.

---

## 2. Goals

- Implement a custom toon/flat-color WGSL shader matching the art style
- Add an outline post-processing effect for character readability
- Create weapon visual effects (muzzle flash, tracers, beam visuals, impact effects)
- Implement per-map color grading
- Differentiate player models visually
- Tune environmental lighting for atmosphere and gameplay readability

---

## 3. Art Direction Summary

### 3.1 Visual Pillars

1. **Low-poly, high-finish:** Models use flat shading with clean geometry. No realistic textures — instead, carefully chosen flat colors with subtle material variation.
2. **Eastern temple × alien aesthetic:** Architectural elements draw from Japanese and Southeast Asian temple design (torii gates, pagoda silhouettes, tatami-grid patterns) but with alien materials (iridescent surfaces, bioluminescent accents, impossible geometries).
3. **Readability first:** Players must be instantly distinguishable from the environment. Outline post-processing, high-contrast color palettes, and clear silhouettes ensure this.
4. **Clean and intentional:** Every visual element serves a purpose. No noise, no clutter. The aesthetic is minimalist but not empty — every surface and shape is deliberate.

### 3.2 Color Palette

- **Environment:** Muted earth tones and weathered stone grays for temple elements. Accent colors in teal, gold, and deep red for alien elements.
- **Players:** High-contrast colors against the environment. Saturated and distinct per player.
- **Weapons:** Dark metallic base with glowing accent colors matching the weapon's energy type.
- **VFX:** Bright, saturated, and brief. Weapon effects should be immediately readable.

---

## 4. Task Breakdown

### 4.1 Custom Toon / Flat-Color Shader

**Task 4.1: WGSL shader foundation**

Write a custom WGSL shader in Bevy's rendering pipeline:
- Flat shading with a single directional light (no per-pixel lighting complexity)
- Solid color output with optional subtle gradient for depth perception
- The shader should support:
  - Base color (set per material)
  - Ambient occlusion approximation (darken areas facing away from the light, but as discrete steps, not smooth gradients — this is the "toon" element)
  - Optional rim lighting for edge highlighting (separate from the outline post-process)

**Task 4.2: Toon shading steps**

Implement discrete lighting steps (cel shading):
- 2–3 light intensity bands: fully lit, half lit, shadow
- The transition between bands should be sharp, not smooth — this is the characteristic toon look
- The shadow color is a darkened version of the base color, not pure black
- This creates the stylized, hand-drawn appearance specified in the art direction

**Task 4.3: Material system**

Create a custom material that integrates with the WGSL shader:
- Base color (Vec4)
- Shadow color (Vec4, auto-calculated from base color if not specified)
- Rim light color and intensity (optional, for player models)
- Emissive color for glowing elements (alien materials, weapon energy)
- The material should be assignable to any mesh via Bevy's standard material component pattern

**Task 4.4: Shader performance optimization**

Ensure the custom shader meets performance targets:
- Flat shading is inherently cheap — verify we are not adding overhead
- Test with 100+ mesh instances on screen simultaneously
- Profile GPU time for the shader pass on target hardware
- Optimize: minimize texture lookups (we use flat colors, so there should be few to none), minimize per-pixel computation

**Estimated effort:** 10–14 hours

---

### 4.2 Outline Post-Process

**Task 4.5: Outline shader implementation**

Implement a screen-space outline post-processing effect:
- **Method:** Render objects to a separate outline pass with a solid color, then detect edges using a Sobel filter or depth-based edge detection
- **Result:** A clean, consistent outline around all outlined objects (primarily player models and interactive elements)
- **Properties:** Outline thickness (2–4 pixels at 1080p), outline color (default: black or dark gray), outline opacity

**Task 4.6: Selective outlining**

Not everything should have outlines. Implement selective outlining:
- Player models: always outlined (for gameplay readability)
- Interactive objects (weapons, pickups): outlined when relevant
- Environment geometry: no outline (would create visual noise)
- Use a render layer or component-based approach to mark which entities receive outlines

**Task 4.7: Outline post-process integration**

Integrate the outline post-process into Bevy's rendering pipeline:
- The outline pass runs after the main scene render
- Composited on top of the scene
- Ensure the outline renders correctly over transparent objects and VFX
- Ensure the outline does not render over the weapon view model (it should not have a visible outline)

**Task 4.8: Performance profiling**

Profile the outline post-process:
- Measure the GPU cost at 1080p, 1440p, and 4K
- The outline pass should add no more than 1–2ms of GPU time
- If the cost is too high at higher resolutions, consider rendering the outline at a lower resolution and upscaling

**Estimated effort:** 8–12 hours

---

### 4.3 Weapon Visual Effects

**Task 4.9: Muzzle flash**

Create a muzzle flash effect for hitscan and projectile weapons:
- A bright, brief flash at the weapon's muzzle position when firing
- Duration: 1–2 frames (extremely brief)
- Visual: a flat plane or billboard with an additive-blended bright texture
- Color matches the weapon's energy type
- Size varies slightly per shot for organic feel

**Task 4.10: Tracer effects**

Create tracer visuals for projectile weapons:
- A stretched bright mesh or particle trail following the projectile's path
- Color matches the projectile type
- Fades over the projectile's lifetime
- The tracer should be visible to both the shooter and other players

**Task 4.11: Beam weapon visuals**

Implement beam weapon rendering:
- A continuous, bright beam from the weapon muzzle to the hit point
- The beam should have: a core (bright, narrow) and an outer glow (wider, semi-transparent)
- Animate the beam with a scrolling texture or UV offset for an energy-flow effect
- Impact point: persistent glow/sparks while the beam is on target
- The beam visual should be distinct from projectile tracers — it is a continuous line, not a traveling projectile

**Task 4.12: Impact effects**

Create impact effects for all weapon types:
- **Surface hit:** Small burst of particles in the surface's color direction. Brief flash. Duration: 0.2–0.3 seconds.
- **Player hit:** Brief red/orange flash on the hit player's model. Distinct from surface impact for feedback.
- **Headshot hit:** Larger, more dramatic impact effect. Unique visual/audio cue (placeholder audio is fine).
- All effects use pooled particles/entities for performance — no spawning and despawning per effect.

**Task 4.13: VFX particle system**

Implement or integrate a particle system for VFX:
- Particles have: position, velocity, lifetime, color, size, and opacity
- Particles can be emitted from a point, line, or surface
- Over lifetime: particles fade, shrink, and/or change color
- System supports pooling for performance
- If Bevy's built-in particles are insufficient, implement a custom GPU-instanced particle system

**Estimated effort:** 15–20 hours

---

### 4.4 Color Grading

**Task 4.14: Color grading system**

Implement per-map color grading as a post-processing step:
- **Exposure:** Overall brightness adjustment
- **Contrast:** Mid-tone contrast curve
- **Saturation:** Global saturation adjustment
- **Color tint:** Subtle color shift for mood (warm, cool, etc.)
- **Vignette:** Darkening at the screen edges for focus

**Task 4.15: Map-specific grading profiles**

Create color grading profiles for each map:
- Profiles are defined as data (tunable constants or asset files)
- Applied when the map loads
- The grading should enhance the map's visual theme without distorting player readability
- Player colors and VFX should remain accurate after grading (account for grading in VFX color selection)

**Task 4.16: Color grading shader**

Write the color grading as a WGSL post-process shader:
- Runs after the main scene render and outline pass, before UI
- Apply adjustments in linear color space for correctness
- Support runtime adjustment for tuning (expose parameters as a resource)

**Estimated effort:** 6–8 hours

---

### 4.5 Player Model Differentiation

**Task 4.17: Player model design**

Design player models that are:
- Visually distinct from each other (different colors, possibly different silhouettes)
- Clearly different from the environment (high contrast, outline rendering)
- Readable at all combat distances (silhouette, color, outline)
- Consistent with the low-poly art style

For v0.4, player models can be simple geometric shapes (stylized capsules, blocky humanoids) as long as they are visually distinct. Detailed character models can come in Phase 5 if needed.

**Task 4.18: Per-player color system**

Implement a system that assigns distinct colors to each player:
- Player 1: Team color A (configurable)
- Player 2: Team color B (configurable)
- Colors are applied to the player model material at spawn
- Colors are replicated over the network so all clients see the same colors

**Task 4.19: Player model visual feedback**

Add visual feedback to player models:
- Damage flash: the model briefly flashes white or red when taking damage
- Death: the model ragdolls or fades out (simple fade for v0.4)
- Respawn: the model appears with a brief spawn-in effect (scaling up from zero, or a particle burst)

**Estimated effort:** 8–10 hours

---

### 4.6 Environmental Lighting

**Task 4.20: Lighting setup framework**

Establish the environmental lighting system:
- Primary directional light (sun/moon equivalent) — the main light source for the toon shader
- Ambient light — provides base illumination so shadow areas are not pure black
- Point lights or spot lights for local accent lighting (torch effects, glowing alien elements)

**Task 4.21: Map lighting per environment**

Light each map environment to support the art direction:
- **Temple maps:** Warm directional light from above, long shadows, golden hour feel. Accent lighting from glowing alien elements (teal/cyan point lights).
- **Alien maps:** Cool directional light, stark shadows, bioluminescent accents. Moody and atmospheric but still readable.
- **Hybrid maps:** Blend of warm and cool — contrast between temple and alien architectural elements expressed through lighting.

**Task 4.22: Dynamic lighting elements**

Add dynamic lighting where it enhances gameplay and atmosphere:
- Muzzle flashes briefly illuminate nearby geometry
- Beam weapons cast dynamic light on surrounding surfaces
- Explosions (if applicable) produce brief light bursts
- These effects are additive to the base lighting and must not overpower it

**Task 4.23: Shadow quality**

Configure shadow rendering for the art style:
- Sharp shadows (consistent with the toon aesthetic — soft shadows would clash)
- Shadow resolution should be sufficient to not show obvious pixelation at typical camera distances
- Shadow draw distance should cover the playable area without excessive GPU cost

**Estimated effort:** 8–12 hours

---

## 5. Task Summary

| Task Group | Tasks | Estimated Hours |
|-----------|-------|----------------|
| Custom Shader | 4.1, 4.2, 4.3, 4.4 | 10–14 |
| Outline Post-Process | 4.5, 4.6, 4.7, 4.8 | 8–12 |
| Weapon VFX | 4.9, 4.10, 4.11, 4.12, 4.13 | 15–20 |
| Color Grading | 4.14, 4.15, 4.16 | 6–8 |
| Player Models | 4.17, 4.18, 4.19 | 8–10 |
| Environmental Lighting | 4.20, 4.21, 4.22, 4.23 | 8–12 |
| **Total** | **23 tasks** | **55–76 hours** |

At 8–12 productive hours per week, this maps to the 6–8 week estimate.

---

## 6. Acceptance Criteria

Phase 4 is complete when **all** of the following are true:

1. **Custom Shader:** All geometry renders with the toon/flat-color shader. Discrete lighting bands are visible. The shader produces no visual artifacts. The aesthetic matches the art direction document.

2. **Outline Post-Process:** Player models have clean, consistent outlines. Outlines are selective (environment has no outlines). Outlines do not render over the weapon view model. Outline cost is under 2ms GPU time at 1080p.

3. **Muzzle Flash:** All hitscan and projectile weapons produce a visible muzzle flash when firing. The flash is brief, bright, and color-coded per weapon type.

4. **Tracers:** Projectile weapons display tracer trails. Tracers are visible to all players. Tracers fade over the projectile's lifetime.

5. **Beam Visuals:** Beam weapons render as a continuous bright beam with a core and glow. The beam has an energy-flow animation. Impact effects are visible at the beam's endpoint.

6. **Impact Effects:** Surface hits, player hits, and headshots each have distinct visual effects. Effects are brief, readable, and performant.

7. **Color Grading:** Each map has a distinct color grading profile that enhances its visual theme. Player readability is maintained. The grading does not distort VFX colors.

8. **Player Differentiation:** Players are visually distinct through color, outline, and silhouette. Player models are readable at all combat distances. Damage, death, and respawn visual feedback is functional.

9. **Environmental Lighting:** Lighting supports the art direction and provides clear gameplay visibility. Dynamic lighting from weapons enhances atmosphere without overwhelming the base lighting. Shadows are sharp and consistent with the toon style.

10. **Performance:** The game runs at target framerates on all hardware tiers:
    - High-end (RTX 3070 / Ryzen 7 equivalent): 120fps at 1440p
    - Mid-range (GTX 1060 / Ryzen 5 equivalent): 60fps at 1080p
    - Low-end (integrated graphics / older GPUs): 30fps at 1080p with reduced settings

---

## 7. Test Protocol

### 7.1 Visual Fidelity Review

1. Take screenshots of each map from multiple angles.
2. Compare against the art direction reference images.
3. Verify: color palette, lighting mood, shader style, outline consistency, VFX readability.
4. Get sign-off from the art direction owner before proceeding.

### 7.2 Performance Benchmark

1. Load each map with two players, all weapon types firing simultaneously.
2. Record: average FPS, 1% low FPS, GPU frame time, CPU frame time.
3. Test on three hardware tiers (or simulate via resolution scaling and GPU throttling).
4. Verify all tiers meet their target framerates.

### 7.3 Readability Test

1. Play a 1v1 match on each map.
2. Verify: player models are instantly visible against all backgrounds, weapon effects are clearly readable, impact feedback is unmistakable.
3. Adjust colors, outlines, or VFX intensity if readability is compromised.

---

## 8. Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Custom WGSL shader has cross-vendor compatibility issues | Medium | High | Test on NVIDIA, AMD, and Intel GPUs early. Maintain a fallback to Bevy's StandardMaterial. |
| Outline post-process is too expensive at high resolutions | Medium | Medium | Implement resolution scaling for the outline pass. Profile early. |
| VFX particle system requires custom implementation | Medium | Medium | Evaluate Bevy's particle ecosystem first. If insufficient, implement a minimal GPU-instanced particle system scoped to JuanSchott's needs. |
| Art direction is difficult to achieve within technical constraints | Low | High | Iterate early with simple test scenes. Get art direction feedback on rough implementations before investing in polish. |

---

## 9. Exit Criteria

Upon completing Phase 4 and meeting all acceptance criteria:

1. Tag the repository as `v0.4`.
2. Capture screenshots and video footage for comparison against the art direction document.
3. Conduct a performance benchmark and record results for all hardware tiers.
4. Proceed to Phase 5 (Content).
