# Material and Lighting Specification

**Document Version:** 1.0
**Last Updated:** 2026-05-05
**Status:** Active — Canonical Reference
**Owner:** Art Direction / Rendering Engineering

---

## Table of Contents

1. [Material System Overview](#1-material-system-overview)
2. [Material Library](#2-material-library)
   - [Environment Materials](#21-environment-materials)
   - [Character Materials](#22-character-materials)
   - [Weapon Materials](#23-weapon-materials)
   - [Material Usage Matrix](#24-material-usage-matrix)
3. [Lighting Setup](#3-lighting-setup)
   - [Primary Light](#31-primary-light-the-vimana-core)
   - [Ambient Light](#32-ambient-light)
   - [Point Lights](#33-point-lights-map-specific)
   - [Volumetric Fog](#34-volumetric-fog)
4. [Shadow Style](#4-shadow-style)
5. [Post-Processing Chain](#5-post-processing-chain)
6. [Custom Shader Requirements](#6-custom-shader-requirements)
7. [Performance Budgets](#7-performance-budgets)
8. [Implementation Notes](#8-implementation-notes)
9. [See Also](#9-see-also)

---

## 1. Material System Overview

JuanSchott renders all surface appearance through custom WGSL shaders built on Bevy's `CustomMaterial` system. **Materials are entirely procedural.** There are no texture maps — no albedo textures, no normal maps, no roughness or metallic texture channels. Every surface in the game derives its visual character from a small set of scalar and color parameters evaluated per-fragment in the shader.

This is a deliberate architectural decision, not a limitation. Flat materials serve three purposes simultaneously:

- **Performance.** Eliminating texture sampling removes the largest single consumer of GPU memory bandwidth on target hardware. On Intel HD 4000-class GPUs, texture fetches are the dominant cost in most fragment shaders. By replacing them with a handful of uniform reads, we keep fragment shading well within budget even at the native resolution of our minimum-spec machines.
- **Visual coherence.** A texture-less pipeline enforces a clean, disciplined aesthetic by default. Every surface is legible at a glance. There is no risk of noisy textures clashing with the low-poly geometry or muddying the silhouette read that Pillar 4 (competitive clarity) demands.
- **Workflow simplicity.** Artists and level designers specify materials through data — a handful of values in a material definition file. No UV unwrapping, no texture authoring, no mip-chain management. This keeps the asset pipeline fast and the iteration cycle short.

### Surface Parameters

Each material is defined by the following properties:

| Parameter | Type | Range | Description |
|---|---|---|---|
| Base color | `Color` (sRGB hex) | Any | Flat, uniform surface color. No per-pixel variation. |
| Metallic | `f32` | 0.0 – 1.0 | Dielectric-to-metallic blend. 0.0 is a pure dielectric (plastic, stone, wood). 1.0 is a fully metallic surface. |
| Roughness | `f32` | 0.0 – 1.0 | Microsurface roughness. 0.0 produces a mirror-smooth specular reflection. 1.0 produces a fully diffuse, matte surface with no discernible specular highlight. |
| Emissive color | `Color` (sRGB hex) | Any | The color of light emitted by the surface. Set to black (#000000) for non-emissive surfaces. |
| Emissive strength | `f32` | 0.0 – 2.0+ | Multiplier on emissive color. Controls bloom intensity and apparent brightness of glow effects. |
| Shading model | `enum` | smooth / flat | Per-object toggle. Smooth uses interpolated vertex normals. Flat uses face normals (faceted look). |

### Physically Based Rendering Model

The shader implements a Cook-Torrance microfacet BRDF with the Smith-GGX geometry function and the GGX/Trowbridge-Reitz normal distribution function. Fresnel is handled via the Schlick approximation with the base color as F0 for metallic surfaces and a fixed 0.04 dielectric F0 for non-metallic surfaces.

This model was chosen because it produces predictable, physically plausible results across the full range of metallic and roughness values. Artists can reason about materials intuitively: "more metallic = more reflective," "lower roughness = tighter highlight." The math is well-understood, GPU-friendly, and maps cleanly to the parameter space above.

---

## 2. Material Library

The following materials constitute the complete palette for JuanSchott. No ad-hoc materials should be introduced without updating this document and receiving sign-off from the art lead.

### 2.1 Environment Materials

#### VIMANA_STONE

| Property | Value |
|---|---|
| **Base color** | `#D4C4A8` (warm sandstone) |
| **Metallic** | 0.0 |
| **Roughness** | 0.7 |
| **Emissive color** | none |
| **Emissive strength** | 0.0 |
| **Shading** | smooth |

The primary surface material for the Vimana's interior. Used for walls, floors, platforms, ramps, stairs, and any large structural surface that reads as "the body of the ship." The warm sandstone tone anchors the palette and provides a neutral, high-value backdrop against which characters, weapons, and energy effects can be read at a glance — a direct requirement of Pillar 4.

At roughness 0.7, the specular highlight is broad and soft, giving the surface a slightly worn, matte finish without appearing powdery or dull. The metallic value of 0.0 ensures the material behaves as a pure dielectric: no colored reflections, no Fresnel brightening at grazing angles beyond the standard dielectric 4% reflectance.

**Design intent:** This material should feel like cut stone — something ancient and crafted, not modern or industrial. It is the visual baseline of the Vimana.

---

#### VIMANA_CARVED

| Property | Value |
|---|---|
| **Base color** | `#BFA97A` (darker warm stone) |
| **Metallic** | 0.0 |
| **Roughness** | 0.8 |
| **Emissive color** | none |
| **Emissive strength** | 0.0 |
| **Shading** | smooth |

Used for carved patterns, inset panels, decorative borders, and any surface detail that should read as "intentionally shaped" rather than structural. The slightly darker, more saturated tone than VIMANA_STONE creates a subtle tonal separation that defines decorative elements without requiring a strong value contrast.

The higher roughness (0.8) makes carved surfaces appear slightly more weathered — as though the carved areas have accumulated more wear over centuries. This is a minor detail but contributes to the sense of age and history in the environment.

**Design intent:** Differentiate decorative geometry from structural geometry through tonal shift, not through emissive effects or strong contrast.

---

#### VIMANA_METAL

| Property | Value |
|---|---|
| **Base color** | `#8B7355` (bronze-like) |
| **Metallic** | 0.6 |
| **Roughness** | 0.4 |
| **Emissive color** | none |
| **Emissive strength** | 0.0 |
| **Shading** | smooth |

Structural and decorative metal elements throughout the Vimana. Door frames, railings, support beams, hinge hardware, grille covers, and mechanical joints. The partial metallic value (0.6) places this in the "tarnished bronze" zone — it picks up colored reflections from the environment (tinted by its base color) but doesn't produce a clean mirror reflection. The lower roughness (0.4) gives it a discernible specular highlight that catches the Vimana Core light, making metal elements subtly glint as the player moves through the space.

This glinting behavior is important. In a texture-less pipeline, specular highlights are one of the few cues available to distinguish metal from stone at a distance. The highlight should be visible but not distracting — it should read as "metallic" without becoming a beacon.

**Design intent:** Provide a clear material read for structural metal. The specular behavior should reinforce the ancient-civilization aesthetic — think aged bronze, not polished chrome.

---

#### VIMANA_GLOW

| Property | Value |
|---|---|
| **Base color** | `#2A6B5A` (deep teal) |
| **Metallic** | 0.2 |
| **Roughness** | 0.3 |
| **Emissive color** | `#00FFAA` |
| **Emissive strength** | 0.5 |
| **Shading** | smooth |

The signature accent material. Used for energy seams, glowing patterns, bioluminescent Vimana systems, power conduits, and any element that should read as "alive" or "powered." The teal-to-green emissive glow is the single most distinctive visual element in the game's palette — it is the color players will associate with the Vimana's presence.

The base color (`#2A6B5A`) is chosen so that when emissive is disabled (e.g., in a power-off state), the surface reads as a dark, muted teal — still thematically consistent but clearly inactive. When emissive is enabled, the bright `#00FFAA` pushes well above the tonemapping threshold, triggering bloom and creating a soft halo around glowing surfaces.

The slight metallic value (0.2) gives the surface a subtle sheen even when unpowered, suggesting a material that is not purely organic but has a technological quality. Roughness at 0.3 keeps the specular highlight relatively tight, reinforcing the "energy conductor" read.

**Design intent:** This material defines the Vimana's identity as a living, technological organism. It must be used with restraint — overuse dilutes its impact and creates visual noise that harms competitive readability.

---

### 2.2 Character Materials

#### PLAYER_BASE

| Property | Value |
|---|---|
| **Base color** | varies (species-dependent) |
| **Metallic** | 0.1 |
| **Roughness** | 0.5 |
| **Emissive color** | none |
| **Emissive strength** | 0.0 |
| **Shading** | smooth |

The default material for player character bodies. Base color is determined at runtime by the player's species selection. Metallic and roughness are kept at neutral midpoints — slightly rougher than skin, smoother than stone — to produce a clean, readable silhouette without drawing attention to the material itself.

The roughness of 0.5 is calibrated to produce a visible but not prominent specular highlight under the Vimana Core directional light. This gives characters a subtle sense of volume and form without the specular becoming a distraction during combat.

**Design intent:** Player characters must be maximally readable. The material should support silhouette recognition without competing for visual attention.

---

### 2.3 Weapon Materials

#### WEAPON_MATTE

| Property | Value |
|---|---|
| **Base color** | `#3A3A3A` (dark gray) |
| **Metallic** | 0.3 |
| **Roughness** | 0.6 |
| **Emissive color** | none |
| **Emissive strength** | 0.0 |
| **Shading** | smooth |

The primary material for weapon bodies — grips, barrels, housings, and structural components. Dark gray sits at a low enough value to remain unobtrusive in the player's lower peripheral vision while still being distinguishable from pure black. The slight metallic value gives weapons a subtle material identity that separates them from organic character surfaces.

**Design intent:** Weapons should be present but not visually dominant. The matte finish ensures that weapon geometry doesn't compete with crosshair tracking or target acquisition.

---

#### WEAPON_GLOW

| Property | Value |
|---|---|
| **Base color** | weapon-specific |
| **Metallic** | 0.1 |
| **Roughness** | 0.2 |
| **Emissive color** | weapon-specific |
| **Emissive strength** | 0.8 |

Used for weapon energy elements — beam weapon barrels, plasma coils, charge indicators, and firing effect emitters. Each weapon archetype defines its own emissive color. The high emissive strength (0.8) ensures that weapon energy elements consistently trigger bloom, creating a visible "charge ready" or "firing" state that the player can read peripherally.

The low roughness (0.2) produces a tight, bright specular highlight that amplifies the perceived energy intensity of these elements under direct lighting.

**Design intent:** Weapon glow elements serve as gameplay feedback — they communicate weapon state (charging, ready, overheating) through color and intensity. They must be bright enough to read without looking down.

---

### 2.4 Material Usage Matrix

| Material | Walls | Floors | Platforms | Doors | Rails | Carved | Energy | Players | Weapons |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| VIMANA_STONE | ■ | ■ | ■ | | | | | | |
| VIMANA_CARVED | | | | | | ■ | | | |
| VIMANA_METAL | | | | ■ | ■ | | | | |
| VIMANA_GLOW | | | | | | | ■ | | |
| PLAYER_BASE | | | | | | | | ■ | |
| WEAPON_MATTE | | | | | | | | | ■ |
| WEAPON_GLOW | | | | | | | | | ■ |

---

## 3. Lighting Setup

Lighting in JuanSchott is designed to serve two masters: atmosphere and competitive clarity. Every light in the scene must justify its existence against both criteria. If a light enhances atmosphere but harms readability, it is removed. If a light improves readability but breaks atmosphere, it is adjusted until it does neither.

### 3.1 Primary Light (The Vimana Core)

| Property | Value |
|---|---|
| **Type** | Directional light |
| **Color** | `#FFF5E0` (warm white) |
| **Intensity** | 60,000 lux |
| **Direction** | Top-down (central axis → inner surface) |
| **Shadows** | Enabled — PCF soft shadows |
| **Shadow map resolution** | 2048 × 2048 |
| **Shadow bias** | Calibrated per-map (default: 0.001) |

The Vimana Core light is the single dominant light source in every scene. It simulates the diffuse glow of the Vimana's central energy axis — the "sun" at the center of the world. As a directional light, it produces consistent, parallel illumination across the entire playable space, which is critical for predictable shadow casting and consistent material appearance.

The warm white color (`#FFF5E0`) is slightly shifted toward yellow-orange, reinforcing the ancient, warm aesthetic of the Vimana interior. This is not a neutral white — it is intentionally warm to make the space feel enclosed and lived-in, as though the light itself has been filtered through centuries of stone and metal.

At 60,000 lux, this light is the single greatest contributor to scene brightness. It must be strong enough to fully illuminate horizontal surfaces (floors, platforms) while leaving vertical surfaces (walls facing away from the core) in recognizable shadow. This contrast is what gives the Vimana interior its sense of directionality and architectural depth.

**Shadow configuration:** PCF (Percentage-Closer Filtering) with a 3×3 sample kernel. Not a full Gaussian blur — the shadow edges should remain slightly crisp, consistent with the hard-edged shadow aesthetic described in Section 4. The PCF pass simply prevents jagged shadow map aliasing, not realistic soft shadow penumbrae.

---

### 3.2 Ambient Light

| Property | Value |
|---|---|
| **Type** | Ambient light (uniform hemisphere) |
| **Color** | `#FFE0B0` (warm gold) |
| **Intensity** | 0.3 |

The ambient light exists to ensure that shadowed areas are never pure black. Pure black shadows violate Pillar 4 — they destroy depth perception, conceal enemies, and make the environment feel like a collection of floating shapes rather than a coherent space.

The warm gold color is matched to the Vimana Core's color temperature, as though the ambient light is simply scattered Core light bouncing off interior surfaces. This maintains tonal coherence: even shadowed areas feel like they belong to the same space.

The intensity of 0.3 is calibrated to produce a visible but clearly secondary illumination level. Shadowed areas should read as "dim but present," not "evenly lit." The ratio between directly lit and ambient-only surfaces should be approximately 8:1 to 10:1 in perceived brightness.

---

### 3.3 Point Lights (Map-Specific)

| Property | Value |
|---|---|
| **Type** | Point light |
| **Color** | `#00FFAA` (teal/green — matched to VIMANA_GLOW) |
| **Range** | 5–15 meters |
| **Intensity** | Low (supplementary only) |
| **Shadows** | **Disabled** |
| **Falloff** | Inverse square |

Point lights are used sparingly and exclusively for localized accent lighting near VIMANA_GLOW surfaces — bioluminescent clusters, energy conduits, active power nodes. Their role is to cast a small pool of teal-tinted light onto nearby geometry, reinforcing the "powered" read of emissive elements and creating localized color contrast against the warm ambient.

**Point light shadows are always disabled.** The GPU cost of additional shadow maps is not justified for accent lighting, and the visual benefit is negligible given the small range and low intensity. Shadow casting for gameplay purposes is handled exclusively by the Vimana Core directional light.

Point lights must not be placed in areas where they could create visual confusion during combat. Specifically:
- No point lights within 3 meters of a commonly used sightline.
- No more than 2 point lights visible simultaneously from any single player position.
- Point lights near pickups or objectives must not change the apparent color of those gameplay elements.

---

### 3.4 Volumetric Fog

| Property | Value |
|---|---|
| **Density** | Very low (barely perceptible) |
| **Color** | Warm white (matched to ambient) |
| **Usage** | Large open spaces only |

Volumetric fog is a situational effect, not a constant presence. It is used exclusively in large, open map areas (arena chambers, grand halls, exterior catwalks) to enhance depth perception by providing an atmospheric gradient — distant geometry is slightly faded, near geometry is crisp. This reinforces the sense of scale in large spaces.

**Fog is never used in combat areas.** In tight corridors, arenas, or any space where players are actively fighting, fog would reduce visibility and obscure enemy silhouettes — a direct Pillar 4 violation. Fog is an architectural tool for establishing scale, not a combat-area effect.

---

## 4. Shadow Style

JuanSchott uses **hard-edged shadows** — crisp, geometric shadow boundaries that complement the low-poly art style.

This is a deliberate aesthetic choice. Soft, realistic shadow penumbrae look visually incongruous when cast by low-poly geometry with flat materials. The hard edge reinforces the stylized, graphic quality of the rendering and creates shadow shapes that are themselves geometric compositions — triangles, trapezoids, and parallelograms that extend the visual language of the low-poly mesh into the shadow domain.

| Property | Value |
|---|---|
| **Shadow type** | Hard-edged (minimal PCF — anti-alias only) |
| **Shadow distance** | 50 meters from camera |
| **Cascade splits** | Optimized for close-to-mid range (0–30m primary combat zone) |
| **Shadow map resolution** | 2048 × 2048 |
| **PCF kernel** | 3×3 (alias reduction only, not softening) |

The 50-meter shadow distance ensures that all combat-relevant geometry casts visible shadows. Beyond 50 meters, shadows fade and geometry is typically at a distance where shadow absence is not perceptible during gameplay.

Cascade split ratios should be tuned to allocate the majority of shadow map texel density to the 0–20 meter range, where players spend the most time observing shadow interactions (enemy shadows, platform shadows, their own shadow). The 20–50 meter range receives progressively lower texel density, which is acceptable given the reduced visual impact at those distances.

---

## 5. Post-Processing Chain

Post-processing is minimal. Every effect must earn its GPU cost against the budget of minimum-spec hardware.

### Enabled Effects

| Effect | Configuration | Rationale |
|---|---|---|
| **Tonemapping** | ACES Filmic | Preserves warm tones and prevents blown-out highlights. Handles the wide dynamic range between lit surfaces and emissive glow elements. The ACES curve is preferred over Reinhard for its superior handling of saturated colors — critical for maintaining the teal glow's vibrancy. |
| **Bloom** | Threshold: high, Intensity: low–medium | Only bright emissive elements (VIMANA_GLOW, WEAPON_GLOW) exceed the bloom threshold. This creates a soft, subtle halo around energy elements without bleeding bloom into non-emissive surfaces. Bloom intensity should be calibrated so the halo extends approximately 10–15 pixels from the source at 1080p. |
| **FXAA** | Default preset | Clean anti-aliasing for low-poly geometry edges. FXAA is chosen over MSAA because it handles the full scene in a single pass, is compatible with the custom shader pipeline, and produces acceptable results on the predominantly straight edges of low-poly geometry. |
| **Color Grading** | Per-map LUT | Each map may specify a subtle color grading LUT to shift the overall palette slightly. This allows maps to have distinct atmospheric identities while remaining within the established material system. LUT adjustments should be minor — ±5% on any single channel. Major grading shifts indicate the wrong base palette, not a need for more grading. |

### Disabled Effects

| Effect | Reason Disabled |
|---|---|
| **Motion Blur** | Obscures vision during fast movement. Direct Pillar 4 violation. Players must be able to track targets while strafing, jumping, and flicking. |
| **Chromatic Aberration** | Adds color fringing that reduces edge clarity and can be mistaken for rendering artifacts. No aesthetic benefit in a flat-material pipeline. |
| **Screen Space Reflections** | Computationally expensive. Our materials are predominantly rough or flat — SSR would contribute almost nothing visible while consuming significant GPU time. |
| **Depth of Field** | Not appropriate for a competitive FPS. Any blur that removes information from the player's view is a Pillar 4 violation. |
| **Film Grain** | Adds visual noise that can mask distant enemy movement and interfere with crosshair clarity. |
| **Vignette** | Darkens peripheral vision. Only acceptable as a transient damage indicator (screen-edge red pulse), never as a persistent artistic effect. |

---

## 6. Custom Shader Requirements

The custom toon/stylized material shader, scheduled for Phase 4 implementation, must satisfy the following requirements:

### Stepped Lighting

The shader should quantize the diffuse lighting response into 2–3 discrete intensity bands rather than producing a smooth gradient from lit to shadow. This creates the characteristic "toon" look — broad, flat areas of consistent brightness separated by crisp boundaries.

Recommended band structure:
- **Band 1 (shadow):** 30% of full diffuse intensity. Applied where `N · L < 0.0` or in ambient-only regions.
- **Band 2 (mid):** 70% of full diffuse intensity. Applied where `0.0 ≤ N · L < threshold` (threshold ≈ 0.5).
- **Band 3 (full):** 100% of full diffuse intensity. Applied where `N · L ≥ threshold`.

The transition between bands should be abrupt — no dithering, no smoothstep feathering. The hard band boundary is the desired visual result.

### Specular Highlights

Despite the stepped diffuse model, the shader should retain a smooth, physically-based specular highlight. The specular component should not be quantized. This combination (stepped diffuse + smooth specular) produces a result that reads as stylized without feeling flat or lacking in material differentiation.

### Emissive Support

Emissive color must be added after the lighting calculation and must not be affected by the stepping function. Emissive surfaces glow at a consistent intensity regardless of their orientation relative to light sources. This is critical for VIMANA_GLOW, which must appear self-illuminated at all times.

### GPU Budget

The shader must be lightweight enough to run at 60 FPS on Intel HD 4000-era integrated graphics. This means:
- No dependent texture reads (we have no textures, so this is inherent).
- No complex branching in the fragment shader. Use `step()` and `mix()` for band selection.
- No per-fragment transcendental functions beyond what the PBR specular already requires.
- Target: fewer than 50 ALU instructions per fragment for the base material pass.

---

## 7. Performance Budgets

The following budgets assume minimum-spec hardware (Intel HD 4000, dual-core CPU, 4 GB RAM, 1366×768 native resolution).

| Budget Item | Limit | Notes |
|---|---|---|
| Shadow map resolution | 2048 × 2048 | Single directional light only. |
| Max point lights per frame | 8 | Shadows disabled. |
| Shadow-casting lights | 1 | The Vimana Core directional light only. |
| Post-processing passes | 4 | Tonemap, bloom, FXAA, color grade. |
| Material shader ALU | < 50 instructions/fragment | Per the custom shader budget above. |
| Overdraw target | < 2.5× average | Enforced by geometry density guidelines. |
| Draw calls per frame | < 300 | Including transparent/emissive passes. |
| VRAM target | < 512 MB | Total GPU memory allocation at min-spec. |

The absence of textures is the single largest performance win in this system. On integrated GPUs, texture sampling dominates fragment shader execution time and memory bandwidth consumption. By eliminating textures entirely, the material system reduces per-fragment cost to a handful of arithmetic operations and a few uniform reads — a fraction of the cost of even a single-texture material in a conventional pipeline.

---

## 8. Implementation Notes

### Material Definition Format

Materials should be defined as Bevy asset files (RON format) with the following structure:

```ron
(
    name: "VIMANA_STONE",
    base_color: (0.831, 0.769, 0.659),
    metallic: 0.0,
    roughness: 0.7,
    emissive: (0.0, 0.0, 0.0),
    emissive_strength: 0.0,
    shading: Smooth,
)
```

Color values are stored as linear RGB (converted from the sRGB hex values listed above). Use the standard sRGB-to-linear conversion: `linear = pow(sRGB / 255.0, 2.2)`.

### Adding New Materials

1. Define the material parameters in this document first.
2. Get art lead sign-off on the parameters and the material's intended use cases.
3. Create the RON asset file.
4. Assign the material to test geometry and validate under the standard lighting setup.
5. Verify that the new material does not create Pillar 4 violations when placed alongside existing materials.

**Do not create materials without updating this document.** Undocumented materials create inconsistency, make reviews impossible, and inevitably lead to parameter drift across the project.

### Emissive Color and Bloom Interaction

Emissive strength directly controls bloom contribution. The bloom threshold should be set so that only surfaces with an emissive strength above approximately 0.3 produce visible bloom. This ensures that VIMANA_GLOW (strength 0.5) produces a clear, intentional bloom while non-emissive materials remain clean.

---

## 9. See Also

- [Art Direction Document](art_direction.md) — overarching visual philosophy, pillar definitions, and palette rationale.
- Visual Language Document (`visual_language.md`) — shape language, composition rules, and environmental storytelling through geometry.
- [Rendering Pipeline](../04_technical/rendering_pipeline.md) — technical implementation details for the custom shader system, render pass structure, and GPU pipeline configuration.

---

*This document is the authoritative reference for all material and lighting decisions in JuanSchott. When in doubt, defer to the specifications above.*
