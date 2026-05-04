# Rendering Pipeline — JuanSchott

**Document Version:** 1.0  
**Last Updated:** 2026-05-05  
**Status:** Active  

---

## 1. Overview

JuanSchott uses Bevy 0.18's forward rendering pipeline as its sole rendering path. The game's visual identity is defined by a flat-color, low-poly stylized aesthetic with hard-edged shadows and minimal post-processing. This design is not merely artistic — it is a deliberate engineering constraint to ensure the game runs at 60fps on integrated graphics hardware from 2012 onward.

The rendering architecture is built around three cameras, three render layers, and a lightweight post-processing chain. There are no screen-space effects, no deferred passes, and no texture-dependent materials. Every rendering decision is optimized for low draw-call counts and minimal GPU memory bandwidth.

---

## 2. Forward Rendering Configuration

### 2.1 Why Forward Rendering

Bevy supports both forward and deferred rendering paths. JuanSchott uses forward rendering exclusively. This decision is driven by the following factors:

- **Lower GPU requirements.** Forward rendering has a fixed overhead per light source per fragment, which scales predictably. For a scene with one directional light and zero to two point lights, forward rendering is significantly cheaper than deferred, which requires multiple render targets (G-buffer) regardless of scene complexity.
- **No MSAA complications with deferred.** MSAA is trivially supported in forward rendering but requires additional resolve passes in deferred. While JuanSchott ultimately uses FXAA rather than MSAA, the forward path keeps the option open.
- **Simplicity.** Forward rendering has fewer moving parts — one geometry pass, one lighting calculation per fragment, one output. This reduces the surface area for rendering bugs and makes debugging with tools like RenderDoc straightforward.
- **Target hardware alignment.** Intel HD 4000 and similar integrated GPUs have limited render-target bandwidth. Forward rendering minimizes bandwidth consumption by writing each fragment once, compared to deferred's G-buffer write + lighting pass read.

### 2.2 Bevy Forward Pipeline Flow

The rendering pipeline executes in the following order each frame:

1. **Visibility determination.** Bevy's visibility system computes which entities are within each camera's frustum. Entities on render layers not visible to a given camera are excluded early.
2. **Shadow map pass.** The directional light renders a shadow map at 2048×2048 resolution for cameras that have shadow rendering enabled. Only world-geometry entities (Layer 0) cast shadows.
3. **Geometry pass (per camera).** For each visible camera, all mesh entities on the camera's visible render layers are rendered. Vertex shaders transform geometry; fragment shaders compute flat color + stepped lighting.
4. **Post-processing.** The rendered frame passes through the post-processing chain: tonemapping, bloom, FXAA, color grading.
5. **UI rendering.** The UI camera renders 2D elements on Layer 2 using Bevy's built-in UI renderer.
6. **Present.** The final composited frame is presented to the swapchain.

---

## 3. Camera Setup

JuanSchott uses three cameras, each configured for a distinct purpose. All three cameras render every frame, but their render layers ensure they never interfere with each other.

### 3.1 Main Camera (3D World)

| Property | Value |
|---|---|
| Projection | Perspective |
| Field of View | 75° horizontal |
| Near Plane | 0.1 |
| Far Plane | 1000.0 |
| Render Layers | Layer 0 (world geometry, other players) |
| Tonemapping | ACES |
| MSAA | Off (FXAA used instead) |
| Shadow Rendering | Enabled |

The main camera is attached to the player entity and follows the player's head transform. It handles all 3D world rendering: map geometry, other player models, projectiles, and environmental effects. The 75° horizontal FOV provides a wide-enough view for competitive gameplay without introducing excessive fisheye distortion.

The camera uses ACES tonemapping as the first stage of the post-processing chain. ACES was chosen over Reinhard or AgX for its pleasing highlight rolloff, which prevents the bright, flat-colored surfaces from clipping harshly.

### 3.2 View Model Camera (First-Person Weapon)

| Property | Value |
|---|---|
| Projection | Perspective |
| Field of View | 55° horizontal |
| Near Plane | 0.01 |
| Far Plane | 10.0 |
| Render Layers | Layer 1 (view model — weapon and hands) |
| Tonemapping | ACES (same as main camera) |
| Shadow Rendering | Disabled |
| Clear Color | Transparent (no background clear) |

The view model camera renders the first-person weapon and hand models. A separate camera is necessary because the weapon model requires a different FOV (55° vs 75°) to prevent the weapon from appearing disproportionately large. The narrower FOV compresses the weapon slightly, making it look natural at the bottom-right of the screen.

The near plane is set to 0.01 to prevent clipping when the weapon model's geometry approaches the camera origin. The far plane is only 10.0 because view models never need to render distant geometry. The shadow rendering is disabled to avoid wasting shadow-map bandwidth on the view model, which does not need to cast shadows onto itself.

This camera does not clear the background — it renders on top of the main camera's output using Bevy's camera ordering system (higher `order` value renders later, compositing on top).

### 3.3 UI Camera

| Property | Value |
|---|---|
| Projection | Orthographic |
| Render Layers | Layer 2 (UI elements) |
| Tonemapping | None |
| Shadow Rendering | Disabled |

The UI camera renders all 2D UI elements: health bars, ammo counters, crosshair, kill feed, scoreboard overlay, and menu screens. It uses orthographic projection so UI elements are sized in logical pixels and are not affected by perspective distortion.

UI elements are rendered after the 3D cameras to ensure they always appear on top. No tonemapping or post-processing is applied to the UI camera's output.

---

## 4. Custom Material Pipeline (Phase 4)

JuanSchott's visual identity uses a custom WGSL shader for flat-color, stylized materials. This shader replaces Bevy's default PBR material for most world and entity geometry.

### 4.1 Material Design Philosophy

The game has no textures. Every surface is a flat color with lighting applied in discrete steps (cel-shading/toon-shading variant). This approach:

- Eliminates texture memory entirely (savings of 64–256MB compared to a textured game).
- Eliminates texture sampling in the fragment shader, reducing bandwidth.
- Creates a cohesive, clean aesthetic that scales well across all resolution tiers.
- Allows artists to define materials entirely through a single color value per surface.

### 4.2 Shader Structure

The custom material shader is written in WGSL (WebGPU Shading Language) and is structured as follows:

#### Vertex Shader

```wgsl
@vertex
fn vertex(
    @builtin(vertex_index) vertex_index: u32,
    @location(0) position: vec3<f32>,
    @location(1) normal: vec3<f32>,
) -> VertexOutput {
    var out: VertexOutput;
    let world_position = mesh_functions::get_world_position(position);
    let world_normal = normalize(mesh_functions::get_world_normal(normal));
    out.clip_position = mesh_functions::get_clip_position(world_position);
    out.world_position = world_position;
    out.world_normal = world_normal;
    return out;
}
```

The vertex shader performs standard MVP transformation using Bevy's built-in mesh function utilities. It passes the world-space position and world-space normal to the fragment shader. No vertex color attributes are used — all color information comes from the material's base color uniform.

#### Fragment Shader

```wgsl
@fragment
fn fragment(
    @location(0) world_position: vec3<f32>,
    @location(1) world_normal: vec3<f32>,
) -> @location(0) vec4<f32> {
    let base_color = material::get_base_color();
    let light_dir = normalize(-light_functions::get_directional_light_direction());
    let view_dir = normalize(camera_functions::get_view_position() - world_position);

    // Diffuse: stepped lighting (3-step toon shading)
    let NdotL = dot(world_normal, light_dir);
    let diffuse_step = select(
        select(0.3, 0.6, NdotL > 0.25),
        1.0,
        NdotL > 0.6
    );
    let diffuse = base_color.rgb * diffuse_step;

    // Specular: subtle, hard-edged
    let half_dir = normalize(light_dir + view_dir);
    let NdotH = dot(world_normal, half_dir);
    let specular_step = select(0.0, 0.15, NdotH > 0.85);
    let specular = vec3<f32>(specular_step);

    // Ambient: fixed low value
    let ambient = base_color.rgb * 0.15;

    let final_color = ambient + diffuse + specular;
    return vec4<f32>(final_color, base_color.a);
}
```

The fragment shader implements three lighting components:

1. **Flat base color.** Sourced from the material uniform. No texture sampling.
2. **Stepped diffuse lighting.** The dot product of the surface normal and light direction (`NdotL`) is quantized into three discrete steps: 0.3 (shadow), 0.6 (mid), and 1.0 (lit). This creates hard-edged toon shading bands that match the low-poly aesthetic.
3. **Subtle specular highlight.** A Blinn-Phong specular term using the half-vector between the view direction and light direction. The specular is gated by a threshold of 0.85 on `NdotH`, creating a small, hard-edged highlight. The specular intensity is only 0.15 to keep it subtle — the game should not look shiny or metallic.

No ambient occlusion is computed. The fixed ambient term of 0.15 prevents shadowed surfaces from going fully black.

### 4.3 Material Uniform

Each material instance provides:

| Field | Type | Description |
|---|---|---|
| `base_color` | `vec4<f32>` | RGBA base color of the surface |
| `emissive` | `vec3<f32>` | Emissive color (used for glowing elements) |

Materials are defined in code as Bevy `CustomMaterial` instances. The asset pipeline does not export material definitions from Blender — instead, materials are mapped in code by matching the Blender material name to a `CustomMaterial` definition.

---

## 5. Post-Processing Chain

The post-processing chain applies to the main camera and view model camera outputs. The UI camera bypasses all post-processing.

### 5.1 Tonemapping — ACES

ACES (Academy Color Encoding System) tonemapping is applied as the first post-processing step. It compresses HDR color values into the [0, 1] range with a cinematic curve. This is critical for the game's flat-color aesthetic: without tonemapping, bright surfaces under direct light would clip to white, destroying the intentional color choices.

Bevy 0.18 provides built-in ACES tonemapping via the `Tonemapping::AcesFitted` component on cameras. No custom implementation is required.

### 5.2 Bloom — Threshold-Based, Subtle

Bloom is applied with a high luminance threshold (0.8) so that only the brightest surfaces — such as emissive elements, skybox highlights, and projectile effects — produce bloom. The bloom radius and intensity are kept low to maintain the clean, flat aesthetic. Over-bloomed flat colors would undermine the stylized look.

Configuration:

| Parameter | Value |
|---|---|
| Threshold | 0.8 |
| Intensity | 0.1 |
| Radius | 0.3 |

### 5.3 FXAA

FXAA (Fast Approximate Anti-Aliasing) is the sole anti-aliasing method. It was chosen over MSAA for the following reasons:

- **Cost.** FXAA is a single full-screen pass that costs approximately 0.5ms at 1080p. MSAA 4x costs significantly more, requiring multiple samples per pixel during the geometry pass.
- **Compatibility with forward rendering.** FXAA operates on the final image and does not interact with the geometry pipeline at all.
- **Sufficiency.** JuanSchott's low-poly geometry has relatively few edges. FXAA's edge-detection approach handles the remaining aliasing adequately. The lack of detailed textures means FXAA's slight blurring is imperceptible.

Bevy 0.18 provides FXAA via the `Fxaa` component on cameras.

### 5.4 Color Grading

A subtle color grading pass adjusts the final image's saturation, contrast, and gamma to achieve a consistent look. The grading is minimal — the goal is consistency across maps, not dramatic color manipulation.

Configuration:

| Parameter | Value |
|---|---|
| Saturation | 1.05 (slightly boosted) |
| Contrast | 1.1 (slightly boosted) |
| Gamma | 1.0 (neutral) |

Bevy 0.18's `ColorGrading` component provides these controls natively.

### 5.5 Post-Processing Implementation

Bevy 0.18 introduces the `FullscreenMaterial` feature, which allows custom post-processing shaders to be implemented as materials that render on a full-screen quad. JuanSchott uses this feature for any custom post-processing passes that are not covered by Bevy's built-in effects. The post-processing chain is:

1. ACES tonemapping (built-in)
2. Bloom (built-in)
3. FXAA (built-in)
4. Color grading (built-in)

All four passes are Bevy built-ins. No custom `FullscreenMaterial` passes are required at launch. If custom effects are needed in future phases (e.g., a vignette or chromatic aberration for damage feedback), they can be implemented as `FullscreenMaterial` shaders inserted into the chain.

---

## 6. Shadow Mapping

### 6.1 Configuration

| Parameter | Value |
|---|---|
| Light Type | Directional (single sun) |
| Shadow Map Resolution | 2048 × 2048 |
| Shadow Filter | None (hard-edged) |
| Cascade Count | 1 (single cascade) |
| Bias | Tuned per map to prevent shadow acne |

A single directional light provides all shadow casting. The shadow map is 2048×2048, which provides sufficient resolution for the game's scale. A single cascade is used because the maps are small enough that one cascade covers the entire play area without visible quality degradation.

### 6.2 Hard-Edged Shadows

Shadows are rendered with no filtering — they produce hard, crisp edges. This is a deliberate aesthetic choice that reinforces the stylized, flat-color visual identity. Soft shadows (PCF, PCSS, or VSM) would look inconsistent with the hard-edged toon shading on the geometry.

The hard-edged shadow approach also has a performance benefit: it eliminates the need for multi-tap shadow sampling, reducing the fragment shader's shadow lookup from 4–16 texture fetches (typical PCF) to a single fetch.

### 6.3 Shadow Casters and Receivers

Only Layer 0 entities (world geometry and other players) participate in shadow mapping. Layer 1 entities (view model) do not cast or receive shadows — they are rendered by a separate camera with shadow rendering disabled. Layer 2 entities (UI) are 2D and do not participate in 3D shadow mapping.

---

## 7. Render Layers

Render layers control which cameras see which entities. This is the primary mechanism for isolating the view model, world geometry, and UI from each other.

| Layer | Contents | Visible To |
|---|---|---|
| Layer 0 | World geometry, other player models, projectiles, environmental effects | Main camera |
| Layer 1 | View model (first-person weapon and hands) | View model camera |
| Layer 2 | UI elements (HUD, menus, crosshair) | UI camera |

### 7.1 Layer Assignment

Entities are assigned to render layers at spawn time via the `RenderLayers` component. The convention is:

- **Map geometry:** `RenderLayers::layer(0)`
- **Other player entities:** `RenderLayers::layer(0)`
- **View model (own player's weapon/hands):** `RenderLayers::layer(1)`
- **UI nodes:** `RenderLayers::layer(2)`

Cameras are configured to see their respective layers:

- **Main camera:** `RenderLayers::layer(0)`
- **View model camera:** `RenderLayers::layer(1)`
- **UI camera:** `RenderLayers::layer(2)`

---

## 8. Performance Rationale

### 8.1 No Screen-Space Effects

JuanSchott does not use SSAO (Screen-Space Ambient Occlusion), SSR (Screen-Screen Reflections), or any other screen-space effect. These effects are excluded for two reasons:

1. **Performance.** Screen-space effects require full-resolution texture fetches with complex kernel sampling. On an Intel HD 4000, SSAO alone can consume 3–5ms per frame. The entire rendering budget for Tier 1 hardware is 10ms.
2. **Aesthetic inconsistency.** SSAO and SSR produce soft, realistic effects that clash with the hard-edged, flat-color stylized look. The game's ambient lighting is handled by the shader's fixed ambient term.

### 8.2 Forward Over Deferred

The forward rendering path was chosen over deferred rendering for the target hardware. Deferred rendering requires:

- Multiple render targets (G-buffer): position, normal, albedo, depth — each consuming GPU memory and bandwidth.
- A separate lighting pass that reads from all G-buffer textures.
- Higher minimum hardware requirements (multiple render target support, sufficient VRAM).

For JuanSchott's low-poly, single-light scene, the deferred path's overhead exceeds its benefits. The forward path renders the same scene with less memory bandwidth and simpler GPU operations.

### 8.3 FXAA Over MSAA

FXAA is used instead of MSAA for anti-aliasing. The rationale:

| Factor | FXAA | MSAA 4x |
|---|---|---|
| Cost | ~0.5ms (single pass) | ~2–4ms (per-sample shading) |
| Compatibility | Works with any content | Requires MSAA-compatible render targets |
| Quality for low-poly | Sufficient | Slightly better edge quality |
| Post-processing interaction | Post-process, no conflicts | Must be resolved before post-processing |

FXAA's 0.5ms cost fits comfortably within the rendering budget. MSAA's 2–4ms cost would consume 20–40% of the entire 10ms rendering budget on Tier 1 hardware.

---

## 9. Summary

JuanSchott's rendering pipeline is designed around three principles: visual consistency, hardware accessibility, and engineering simplicity. The forward rendering path, custom flat-color shader, hard-edged shadows, and minimal post-processing chain work together to produce a distinctive visual style that runs at 60fps on integrated graphics from over a decade ago. Every rendering decision — from the exclusion of screen-space effects to the choice of FXAA over MSAA — is made in service of these principles.
