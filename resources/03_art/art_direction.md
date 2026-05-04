# Art Direction

---

## Art Direction Statement

JuanSchott uses a **low-poly, high-finish** art style: minimal polygon counts rendered with high-quality lighting and materials. Geometry is simple and faceted with strong silhouettes. Textures are minimal or absent. Color gradients and controlled palettes define surface appearance. Materials use subtle reflections and smooth shading for a polished, premium look despite low complexity.

The aesthetic blends Eastern temple geometry — drawing from the tiered platforms of Angkor Wat, the sweeping curves of Hindu temples, the mandala-like floor patterns of sacred architecture — with alien, unknowable design. The result is sacred, ominous, beautiful, and wrong. Architecture built by beings with their own concept of beauty, at a scale that dwarfs human proportion. Every surface should feel like it was shaped by hands that think differently about form, about space, about what it means to build something holy.

This is not retro. This is not nostalgic. This is a deliberate aesthetic that treats simplicity as sophistication. Every polygon is placed with intent. Every material choice communicates. Every light source has a reason.

---

## What "Low-Poly, High-Finish" Means

The phrase is the foundation of every visual decision in JuanSchott. It is not a contradiction — it is a philosophy. Complexity lives in the rendering, not in the mesh.

### Low-Poly

Polygon budgets are aggressively restrained. This is a design choice, not an optimization compromise.

- **Character models:** 500–2,000 triangles. Extremely low by modern standards, but sufficient when silhouettes are strong and materials do the visual work.
- **Weapon models:** 200–800 triangles. A gun is a shape, not a sculpture. Every triangle serves the outline or a functional visual element.
- **Map geometry:** Large, simple shapes. No complex meshes. A wall is a wall — a flat plane with material definition. A pillar is a cylinder. A platform is a box. Complexity comes from composition, not from individual mesh density.
- **No subdivision surfaces.** No high-poly baking. No normal maps. The geometry you see is the geometry that exists. What you see is what was built.
- **Strong silhouettes are prioritized over fine detail.** If a shape reads clearly at distance, at speed, in shadow, then it works. If it requires close inspection to understand, it fails.

The discipline of low poly forces intentionality. When you cannot add detail, you must choose what matters. Every edge is a statement. Every face is a decision. This restraint produces clarity — and clarity produces a kind of beauty that high-poly work often buries under noise.

### High-Finish

The "high-finish" half elevates simple geometry into premium visual quality. This is where JuanSchott distinguishes itself from retro indie aesthetics.

- **Materials:** Flat color with subtle metallic and roughness variation. Not fully PBR-complex, but PBR-inspired. A stone wall is not a flat gray plane — it has a slight roughness, a faint warm tint, a barely perceptible specular response that shifts as the player moves past it. The material tells you what the surface is without needing a photograph.
- **Lighting:** Strong directional light combined with controlled ambient. Clean, crisp shadows with soft penumbra at edges. No baked textures — real-time lighting does all the work. Shadows should feel architectural, grounding objects in their environment. The primary light source is treated with the reverence of temple lighting: directional, warm, intentional.
- **Shading:** Smooth shading on most surfaces. Hard edges only where deliberately placed — architectural edges, weapon detail lines, the rims of platforms. The transition between smooth and hard edges is itself a design tool, communicating structure and material change.
- **Reflections:** Subtle and controlled. No mirror reflections anywhere in the game. Soft specular highlights on metallic surfaces, gentle fresnel effects on polished stone, faint bounce light on floors near bright walls. Reflections exist to suggest material quality, not to show the world.
- **Post-processing:** Minimal and disciplined. Subtle bloom on bright surfaces — emissive elements, sky openings, light sources. Tonemapping to maintain readable values across all lighting conditions. Color grading per map to reinforce palette choices. Nothing else. No chromatic aberration. No motion blur. No vignette except as a damage indicator in combat. No film grain. No depth of field. The image is clean, confident, and unapologetically digital.

The result should look like a premium product — something designed, not something constrained. A player encountering JuanSchott for the first time should never think "this looks low-poly." They should think "this looks intentional." The low triangle count is a style, not a limitation. It is a feature, not a fallback.

---

## Visual Identity Pillars

Five pillars support every visual decision in JuanSchott. When in doubt about any art choice, return to these. If a decision serves at least two pillars simultaneously, it is almost certainly correct.

### Pillar 1: Strong Silhouettes

Every object in JuanSchott must be identifiable by its outline alone. This is the single most important visual rule.

Players, weapons, map elements — all must possess distinctive, readable shapes. A player should be able to identify an opponent's weapon class from across the map, in shadow, at speed. A weapon's silhouette should telegraph its function: a long, angular silhouette reads as a precision weapon; a short, boxy silhouette reads as close-range; a slim, elongated silhouette with glowing elements reads as an energy weapon.

Character silhouettes must differentiate species at a glance. A Human reads as Human — natural proportions, nothing exaggerated. A Humanoid reads as something other — the limbs too long, the proportions slightly off in a way that registers before the conscious mind names it. An Artificial Human reads as too perfect — symmetry that feels uncanny. A Cybersymbiote reads as hybrid — the silhouette interrupted by mechanical elements that break the organic flow.

This pillar serves both art direction and gameplay. Readability is not separate from beauty — it is a form of beauty. A clean silhouette is an elegant silhouette.

### Pillar 2: Controlled Palette

Each map in JuanSchott has a defined color palette of 4–6 colors. No rainbow environments. No color chaos. Every hue in a scene was chosen and placed with purpose.

Colors communicate:

- **Warm tones** mark areas of relative safety, rest, and player orientation. Lobbies, spawn zones, and non-combat spaces lean warm.
- **Cool tones** signal danger, alien presence, and environmental threat. Combat arenas, Pantheon-controlled zones, and environmental hazards lean cool.
- **Accent colors** mark interactive elements, objectives, and points of interest. A door that can be opened has a subtle accent highlight. An objective glows faintly with a designated accent hue.
- **Neutral tones** form the architectural base — the stone, the metal, the floor. They provide the visual silence against which the palette speaks.

The discipline of limited palettes forces cohesion. When every surface color was chosen rather than sampled from a texture, the result is a visual harmony that photographs and texture libraries cannot produce. The environment reads as designed — because it was.

### Pillar 3: Material Over Texture

No textures. This is absolute.

Surface variation comes entirely from material properties: metallic versus matte, rough versus smooth, emissive versus reflective. A single flat color with the right material properties looks better on a low-poly model than a detailed texture ever could. Textures introduce visual noise that fights against the clean geometry. They add information without adding meaning. They make low-poly look worse, not better, because the viewer's eye catches the contradiction between simple form and complex surface.

Instead, the material system does the work:

- **Stone:** Warm base color, medium roughness, no metallic, very faint specular. Reads as solid, ancient, and grounded.
- **Metal:** Cool or neutral base color, low roughness, moderate metallic, clear specular response. Reads as crafted, precise, and alien.
- **Emissive elements:** Self-illuminated color, no reliance on external light for visibility. Reads as powered, active, otherworldly.
- **Glass/crystal:** Transparent or translucent, high smoothness, refractive hints. Reads as fragile, valuable, and beyond human manufacture.
- **Flesh (characters):** Subsurface scattering approximation, warm undertones, slight variation across the surface. Reads as alive without requiring pore-level detail.

Material communicates identity. Material communicates temperature. Material communicates history. A metallic pillar and a stone pillar can share the exact same polygon count and silhouette language, but their materials tell the player everything about their origin and purpose.

### Pillar 4: Scale That Dwarfs

The Vimana was not built for humans. It was not built for anything that thinks like a human about space, about comfort, about proportion. The architecture must communicate cosmic scale at every turn.

Doorways are too tall — three meters, four, sometimes more. Steps are too wide, forcing a visual rhythm that no human gait naturally matches. Pillars rise beyond sight, their capitals lost in overhead shadow. Platforms span distances that make a single player feel like an ant on a temple floor. Ceilings are distant, sometimes absent entirely, opening into spaces so vast that the light behaves differently — softer, more diffuse, as if the air itself has room to breathe.

This is not about making things arbitrarily large. It is about making them wrong-large. The proportions are not just big; they are scaled to a body plan that is not human. A doorway that is three meters tall but only one meter wide suggests a being that is tall and narrow. A staircase with steps half a meter deep suggests a stride far longer than any human's. These subtle wrongnesses accumulate into a pervasive sense that the player is small, visiting, tolerated — not the intended occupant.

The emotional target: the player should feel like a pilgrim in a cathedral built for gods. Reverence mixed with unease. Beauty that is not for them.

### Pillar 5: Sacred Geometry

The architectural language draws from the geometry of sacred spaces without replicating any specific tradition.

Sweeping curves that suggest vaulted ceilings and arched doorways without being arches. Tiered platforms that recall temple steps and ziggurat levels without being stairs. Carved surfaces where geometric patterns — not figurative ones — repeat in ways that suggest meaning without conveying specific symbols. Mandala-like arrangements in floor and wall geometry: concentric, radial, symmetrical, centered. Not copied from any Hindu or Buddhist source, but inspired by the same human instinct to express the infinite through pattern.

The geometry evokes the sacred without appropriating any religion. It achieves this by abstracting the underlying mathematical principles — symmetry, repetition, nesting, radial balance — and expressing them through alien proportions and non-human scale. The result feels like religion without being religious, like worship without an object of worship.

Curves dominate over angles in sacred spaces. Angles dominate in functional spaces — combat arenas, corridors, utility areas. This contrast itself becomes a spatial language: curved environments feel contemplative, angled environments feel purposeful. The player learns to read the architecture as a guide.

---

## Reference Targets

The following references inform the visual direction. They are guides, not templates. No single reference defines the final look; the synthesis of all of them, filtered through the specific requirements of JuanSchott's gameplay and setting, produces the target aesthetic.

### Games

- **Monument Valley** — Clean geometry rendered with impeccable care. Controlled color palettes where every hue serves composition. Impossible architecture that feels natural within its own logic. The way light and shadow define form without texture. The confidence to be simple.
- **Journey** — Sweeping curves in landscape and architecture. Warm color gradients that communicate temperature and emotion. An overwhelming sense of scale — the player is small against structures that are vast. Movement through beautiful spaces where the beauty itself is part of the experience.
- **Firewatch** — Flat color palette with strong directional lighting. Minimal texture, maximum atmosphere. Proving that color and light alone can create depth and richness. The way time of day transforms the same geometry into entirely different emotional experiences.
- **Hyper Light Drifter** — Top-down perspective, different from JuanSchott, but the principles are identical: clean shapes, bold color, no texture reliance, material definition through color and shading. The confidence to let the art be the art without hiding behind post-processing or visual clutter.

### Real-World Architecture

- **Angkor Wat** — Tiered platforms creating a sense of ascent and pilgrimage. Carved surfaces where every flat plane has been given meaning through pattern. Pillars that define space without enclosing it. The integration of structure and ornament into a unified whole where you cannot tell where the engineering ends and the art begins.
- **Hindu Temples** — Sweeping curves in tower forms (shikharas) that draw the eye upward. Mandala-pattern floor plans that organize space radially around a sacred center. Vertical emphasis in every proportion — height as aspiration, height as devotion. The use of repeated small elements (sculptural figures, architectural motifs) to create large-scale visual rhythm.
- **Rendezvous with Rama (fictional)** — Arthur C. Clarke's cylindrical world: a vast interior space where the curve of the ground is visible overhead. Cosmic scale that makes humans irrelevant. Alien architecture that is beautiful but clearly not designed for human aesthetics or human comfort. The sense of exploring something that was built by intelligence, but intelligence that thinks differently about every fundamental assumption.

### Synthesis

JuanSchott takes the geometric cleanliness of Monument Valley, the scale and warmth of Journey, the color confidence of Firewatch, the material simplicity of Hyper Light Drifter, the tiered sacredness of Angkor Wat, the vertical aspiration of Hindu temples, and the cosmic alienation of Rama — and synthesizes them into a single coherent visual language that serves competitive FPS gameplay.

---

## Color Philosophy

Color in JuanSchott is not decoration. It is communication. Every color choice carries information about space, safety, threat, function, and origin.

### Per-Map Palettes

Each map defines its own palette of 4–6 colors. These palettes are not arbitrary — they are selected to serve the map's emotional tone and gameplay function.

### Core Color Roles

- **Base Tone:** Warm stone — sandy beige, cream, pale gold. This is the default surface color of the Vimana's architecture. It communicates age, solidity, and a warmth that contrasts with the alien elements. Every map starts here and diverges.
- **Warm Accent:** Deep red, saffron orange, or temple gold. Used for architectural highlights, functional elements, and areas of human-relative warmth. These colors draw the eye and signal approachability.
- **Alien Accent:** Bioluminescent blue-green. This is the signature color of Pantheon technology and alien presence. It marks elements that are active, powered, or controlled by non-human intelligence. Its cool temperature creates immediate visual tension against the warm base.
- **Shadow Tone:** Deep indigo or warm black. Never pure black — pure black reads as dead, digital, absent. Shadow in JuanSchott has color. It has warmth or coolness. It suggests depth rather than void.
- **Light Tone:** Warm white or soft gold. Never cold white — cold white reads as clinical, sterile, human-institutional. The Vimana's light sources, whether natural or artificial, carry warmth. Even alien light sources have a faint warmth, as if the beings who built this place understand light differently but not oppositely.
- **Player Team Colors:** Distinct, saturated, and readable against any map palette. Warm red versus cool blue is the baseline. These colors must never appear as significant environmental colors — they are reserved for player identification. If a map's palette includes a hue too similar to either team color, the palette is adjusted, not the team color.

### Color Interaction Rules

- A map's palette must always include at least one warm tone and at least one cool tone. Monochromatic warmth or coolness produces visual monotony and loses the communicative power of temperature contrast.
- The alien accent color appears sparingly. Overuse dilutes its impact. When everything glows, nothing is alien.
- Gradients are preferred over hard color transitions. A wall that shifts from warm beige at its base to cool shadow at its top communicates more than two flat colors meeting at a seam.
- Emissive colors must serve a function. They indicate active technology, danger zones, or interactive elements. Decorative emissive use is prohibited.

---

## Species Visual Differentiation

All playable species share the same hitbox for gameplay balance. Visual differentiation communicates origin and identity without affecting competitive fairness.

### Design Principles

Each species must be immediately distinguishable at combat range (20–50 meters) in under one second. This means the visual differences must manifest in silhouette, material, and color — the three systems that read at speed and distance.

| Species | Visual Identity | Material Cues | Color Cues |
|---------|----------------|---------------|------------|
| **Human** | Natural proportions, grounded stance, no exaggeration. Reads as baseline. | Standard organic materials. Slight roughness variation across skin, clothing fabric response, leather and metal on gear. | Earth tones for base, natural skin tone range, muted gear colors. No emissive elements. No non-natural hues. |
| **Humanoid** | Elongated limbs, torso slightly compressed, proportions that read as graceful but not human. The uncanny valley of body plan. | Smoother skin material than Human, slight subsurface scattering with non-natural tint. Clothing materials are finer, closer to the body. | Subtle non-natural skin tones — pale blue, deep gold, faint violet. Not saturated, but noticeable. The colors suggest alien origin without screaming it. |
| **Artificial Human** | Too-perfect symmetry. Every feature mirrored with mechanical precision. Flawless skin with no variation. The uncanny valley of perfection. | Near-zero roughness on skin. Faint luminescence in low light — not emissive, but a subtle brightening of specular response in shadow. Every surface is too smooth, too even. | Neutral to cool skin tones with unnatural uniformity. No blemish, no variation, no warmth. If a Human is an oil painting, an Artificial Human is a photograph — technically accurate but missing something. |
| **Cybersymbiote** | Hybrid organic and mechanical form. Visible circuitry patterns on skin — not surface detail but material integration. Mechanical joints at elbows, knees, knuckles. Glowing accents at integration points. | Mixed materials: organic skin alongside polished metal, exposed circuitry with slight emissive response. The contrast between organic and machine is the point. | Base skin tone in the Human range, but interrupted by metallic accents in silver or dark chrome. Circuitry glows faintly with the alien accent color (blue-green) at integration points. The glow is not decorative — it indicates active cybernetic systems. |

### Shared Rules

- No species is larger or smaller than another in gameplay terms. Visual scale differences are illusion — slightly longer limbs on Humanoids read as taller without actually being taller.
- All species share the same animation timing for gameplay actions. Visual flair can differ (a Humanoid reloads with a graceful wrist rotation, a Cybersymbiote reloads with mechanical precision), but the functional timing is identical.
- Silhouette variation between species should be achievable with different visual attachments, proportions, and material treatments on the same base mesh. This is a practical optimization as much as a design philosophy.

---

## Weapon Visual Language

Weapons in JuanSchott follow a strict visual taxonomy. A player should be able to identify a weapon's class and function from its shape alone. The design language is consistent across all weapons, creating a coherent visual system rather than a collection of individual designs.

### Class Visual Rules

- **Hitscan Weapons:** Clean, precise, angular geometry. Minimal decoration. Form follows function. Every edge is a straight line, every surface is a flat plane. The weapon communicates accuracy through its geometry — nothing about it is soft or ambiguous. Material is primarily matte with subtle metallic accents on functional components. Color is typically a single neutral tone (dark gray, gunmetal, warm bronze) with material variation providing visual interest.

- **Projectile Weapons:** Slightly more industrial in design. Visible launch mechanisms — a barrel opening, a chamber, a loading port. Sturdier proportions that suggest kinetic force. Slightly thicker profiles than hitscan weapons. Material includes more visible metal components. The visual language says "this thing launches physical objects" — the geometry must make that clear.

- **Beam Weapons:** Elegant and slim. Glowing elements along the barrel that trace the energy path from source to emitter. The geometry is the most refined of all weapon classes — curves are permitted here where they are not on ballistic weapons. Energy-focused design with visible power conduits or capacitors. Material leans toward polished surfaces with clear emissive accents. The alien accent color (blue-green) is acceptable for beam weapon energy elements.

### Universal Weapon Rules

- **Low-poly, single-color body** with subtle material variation across components. No camouflage patterns. No textures. No weathering. Weapons in JuanSchott are manufactured, not scavenged. They look purpose-built and well-maintained.
- **Material variation communicates function.** The grip is a different material than the barrel. The trigger assembly has a different finish than the frame. These variations are material-only — no texture, no normal mapping.
- **No ornamentation for its own sake.** Every visual element on a weapon should suggest a function. If a design element does not communicate something about how the weapon operates, it should not exist. Decoration is the enemy of readability.
- **Weapon color should not compete with team colors.** Weapons are contextual objects, not identity markers. Their colors should be neutral enough to read against any team color without visual conflict.

---

## Animation Style

Animation bridges the gap between low-poly geometry and living characters. The animation philosophy prioritizes feel over fidelity.

### Movement Animations

Fluid and weighty. Characters move with a sense of mass and momentum, not with the rigid, mechanical motion that low-poly characters often default to. A running character's body leans into the direction of travel. A stopping character's weight shifts visibly. Turning involves the whole body, not just a rotation of the root.

Despite low polygon counts, animation should make characters feel organic. Smooth interpolation between states. No hard cuts between idle and run, or run and stop. Transitions should blend naturally, conveying the physical reality of a body in motion.

### Combat Animations

Snappy and decisive. Quick draw, quick reload, quick death. Combat timing is fast in JuanSchott, and animations must match. A reload that takes 1.5 seconds should feel like 1.5 seconds of focused, efficient action — not 1.5 seconds of waiting. Every frame of a combat animation should communicate purpose.

Death animations are fast. A character drops. No dramatic ragdoll convulsions, no prolonged dying sequences. Combat in JuanSchott is fast, and death is fast. The visual language should respect the pace of the gameplay.

### Idle Animations

Subtle but present. A slight breathing motion in the torso. A gentle weight shift from one foot to the other on a slow cycle. The occasional small adjustment — a hand moving, a head turning slightly. Characters should feel alive while waiting, not like statues propped upright. The low-poly style makes stillness more noticeable, so idle animation must counteract the static quality of the geometry.

### Facial Animation

Minimal by design. Helmets and masks are preferred for player characters. This decision serves multiple purposes simultaneously:

- Reduces animation complexity and cost
- Improves silhouette readability by standardizing head shapes
- Reinforces the tone — the Vimana is hostile, and characters are equipped for it
- Eliminates the uncanny valley of facial animation on low-poly geometry

When faces are visible (NPCs, story moments), animation should be limited to jaw movement for speech and broad emotional expressions. No nuanced facial performance. The style does not support it, and attempting it would undermine the aesthetic.

---

## Environment Art Direction

### Architectural Language

The Vimana's architecture follows a consistent visual grammar:

- **Vertical emphasis:** Vertical lines dominate. Pillars, doorframes, wall paneling — the eye is drawn upward constantly. This serves both the scale pillar and the sacred geometry pillar.
- **Repetition with variation:** Architectural elements repeat, but never identically. Each pillar is the same shape as its neighbors but slightly different in proportion or detail. This suggests organic construction — built, not manufactured — and prevents visual monotony.
- **Depth through layering:** Flat surfaces are broken by recessed panels, protruding ledges, and stepped geometry. A wall is never just a wall — it has depth. This creates shadow lines and material transitions that add visual richness without adding polygon count.
- **Negative space as design:** Empty space is intentional. Vast halls with minimal objects. Long sight lines through open doorways. The absence of clutter makes the architecture itself the visual content.

### Spatial Reading

Players should be able to read a space within seconds of entering it. The architecture communicates:

- **Where to go:** Sight lines, light sources, and color accents guide movement.
- **Where cover exists:** Geometry that provides cover is visually distinct — lower, darker, more enclosed.
- **Where threats may come from:** Elevated positions, dark doorways, and long sight lines signal danger zones.
- **What is interactive:** Interactive elements carry a subtle visual treatment — slight emissive accent, different material, distinct color from surrounding architecture.

---

## What JuanSchott's Art Is NOT

Clarity about what the art is not prevents drift during development. If any of these descriptions start appearing in the game's visuals, the art direction has been compromised.

- **NOT pixel art.** The low-poly geometry is not a pixel art affectation. Surfaces are smooth, not chunky. Colors blend, not quantize.
- **NOT voxel art.** There are no cubes. The geometry uses the full range of polygonal shapes — triangles, quads, ngons, curves approximated by line segments. No Minecraft, no Teardown.
- **NOT photorealistic.** There is no attempt to simulate reality. The materials suggest reality; they do not replicate it. A stone wall reads as stone without pretending to be a photograph of stone.
- **NOT anime or manga style.** Characters do not have exaggerated eyes, spiky hair, or anime proportions. The aesthetic is global, not genre-specific.
- **NOT retro, 8-bit, or 16-bit.** This is not a nostalgia project. The visual style does not reference any specific era of gaming. It is contemporary and forward-looking.
- **NOT ripped from any specific cultural tradition.** The architecture is inspired by Eastern sacred spaces but is not a copy of them. No religious symbols. No direct quotations of specific temples. The geometry evokes the sacred without appropriating the sacred. Inspiration, not imitation.

---

## Art Pipeline Guidelines

### Modeling Standards

- All models must be reviewed for silhouette clarity at intended viewing distance before approval.
- Triangle counts are hard budgets, not suggestions. Exceptions require art director approval.
- Models should use smooth shading by default. Hard edges must be explicitly marked and justified.
- No hidden geometry. If a face cannot be seen by the player, it should not exist.

### Material Standards

- All materials must be tested under the primary directional light and in shadow. A material that only looks good in one lighting condition is a failed material.
- Emissive values must be reviewed for bloom behavior. Over-bright emissives that blow out in post-processing will be rejected.
- Material variation should be achievable through property adjustments, not through separate material instances. The material system should be efficient.

### Lighting Standards

- Every map has a single primary directional light. Additional lights must be justified by gameplay or architecture.
- Shadow quality must be consistent. Shadow acne, peter-panning, and shadow map artifacts are bugs, not style.
- Ambient light levels must be calibrated per map to ensure readability of characters and weapons in all areas. No area of a map should be so dark that a player cannot identify an opponent.

---

## Quality Bar

The quality bar for JuanSchott's art is not "does it look good for a low-poly game?" The quality bar is "does it look good?" Period. The low-poly style is a choice that should make the game look distinctive and premium, not constrained and amateurish.

If a piece of art looks like it could be from a student project, it is not finished. If a scene looks like the team ran out of budget, the composition needs to be reworked. The aesthetic must feel like the team chose this style because it was the best possible choice for the project, not because it was the easiest or cheapest option.

Every screen of JuanSchott should look like concept art. That is the bar.

---

## See Also

- [Visual Language](visual_language.md) — Detailed breakdown of the visual grammar used across all art systems
- [Material and Lighting](material_and_lighting.md) — Technical specifications for materials, lighting setups, and rendering pipeline
- [UI and HUD Design](ui_and_hud_design.md) — Visual design for user interface, HUD elements, and information display
- [The Vimana](../01_lore/the_vimana.md) — Lore and spatial description of the setting that drives all architectural and environmental art decisions
