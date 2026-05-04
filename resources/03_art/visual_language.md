# Visual Language

---

This document defines the visual grammar of JuanSchott — the specific, enforceable rules governing shape, color, readability, motion, and effects across every element the player sees. Where [Art Direction](art_direction.md) establishes philosophy and principles, this document translates those principles into concrete specifications. Every artist, animator, and technical designer should be able to reference this document to answer the question: "What should this look like, and why?"

---

## Shape Language

Shape language is the geometric vocabulary of JuanSchott. Every object in the game is built from a defined set of forms, and deviations from this vocabulary break visual coherence. The shape system is not arbitrary — it is engineered to support silhouette readability, gameplay communication, and the aesthetic of sacred alien architecture rendered in low-poly precision.

### Player Characters

The player character's shape exists within strict constraints. The gameplay collider is a capsule (or cylinder, depending on final physics implementation), and the visual mesh must respect this underlying volume. The character is, at its most essential level, a readable capsule with limbs.

**Base form:** Capsule or cylinder. This is non-negotiable. The character's torso and head volume must read as a unified vertical mass from every viewing angle — front, side, three-quarter, and above. Additive geometry (arms, legs, weapon) extends from this core volume but never breaks its fundamental capsule readability.

**Silhouette priority:** A player character in pure shadow — rendered as a flat black shape against a lighter background — must be instantly identifiable as a player and not as map geometry. This is the ultimate test. If a shadowed character could be mistaken for a pillar, a protrusion, or an architectural element, the character design has failed. The capsule form, the limb extensions, and the weapon mounting all serve this requirement.

**Prohibition on silhouette-breaking protrusions:** No backpacks. No large shoulder pads. No trailing scarves, capes, or dangling elements. No oversized helmets that expand the head volume beyond the capsule's top curve. Every element added to the character must contribute to clarity, not visual noise. The weapon, mounted on the character's side, is the sole significant protrusion — and it serves double duty by communicating loadout information through silhouette alone.

**Weapon visibility:** The held weapon extends from the character's silhouette at the hip or shoulder level. At medium range, the weapon's profile — angular for hitscan, bulky for projectile, slim and glowing for beam — must be readable against the character's body. The weapon is part of the silhouette, not hidden behind it. This may require slight asymmetry in the character's idle pose to ensure the weapon profile is exposed to the camera.

### Map Architecture — Vimana Surfaces

The Vimana's architecture speaks a geometric language that is internally consistent but deliberately alien. It evokes the sacred without replicating any real-world tradition. Every architectural form serves both spatial design and visual identity.

**Pillars:** Tall cylinders. Sometimes tapered — wider at the base, narrowing as they rise — suggesting a gravitational logic that predates human engineering. Pillars are always taller than seems necessary. A pillar that supports a ceiling four meters above the floor might be six meters tall, its capital extending into shadow above the visible light line. This exaggeration serves the scale pillar: the architecture was not built to human proportion.

**Platforms:** Flat rectangular or polygonal slabs. Clean, sharp edges. Surfaces are planes — no curvature on horizontal walking surfaces unless the map design calls for a specific ramp or bridge. Platforms are frequently suspended in space, held by invisible or energy-tether means, reinforcing that the Vimana defies conventional structural logic. The undersides of platforms are visible from below and should be finished with the same material care as the tops.

**Walls:** Flat vertical surfaces interrupted by carved geometric patterns. Patterns are regular, repeating, and non-representational — they suggest language or meaning without being any real-world script or symbol system. Patterns are carved into the surface (recessed) or raised in relief, creating depth and shadow interaction that changes with lighting angle. The patterns are the Vimana's texture substitute: they provide visual richness on flat planes without relying on image-based textures.

**Arches:** Sweeping curves connecting vertical surfaces or spanning openings. Inspired by the visual logic of temple archways — the way sacred architecture uses the arch to draw the eye upward and create a sense of passage — but rendered at alien scale and with alien proportions. Arches in the Vimana are wider at the apex than at the base, the inverse of human structural convention. This subtle wrongness accumulates into pervasive otherness.

**Steps:** Too wide and too tall for comfortable human use. Each step demands a full stride, not a half-step. The riser height is slightly above what a human body plan considers natural. The Vimana was not built for us. Steps are always wider than they need to be for a single person — they suggest a being with a broader stance, a longer gait, a different relationship with horizontal movement. Players should feel physically mismatched to the architecture they traverse.

### Weapons

Weapon shape language is a taxonomy. Each weapon class has a defined geometric identity that communicates its function before the player ever fires it.

**Hitscan weapons:** Angular, precise, minimal. Every line is straight. Every surface is a flat plane. The geometry communicates surgical intent — this is a tool for exact placement of damage at a point. There are no curves, no bulk, no decoration. The profile is lean and directional, with a clear front-to-back axis that indicates the firing direction. Think of a scalpel, not a hammer.

**Projectile weapons:** Slightly bulkier and more industrial. Visible mechanical elements — a barrel opening, a chamber, a loading port, a drum or magazine. The geometry communicates kinetic force: this thing launches physical objects. The profile is stockier than hitscan weapons, with a wider body that suggests internal mechanism and ammunition capacity. The visual language says "manufactured tool of blunt force."

**Beam weapons:** Slim, elegant, with glowing elements traced along the barrel's length. Curves are permitted here — the only weapon class where curvature is acceptable. The geometry traces the path of energy from source to emitter, with visible conduits or capacitors along the weapon's body. The profile is the most refined of all classes. Glowing elements use the alien accent color (bioluminescent green-cyan) to communicate energy type. The visual language says "focused power, not brute force."

---

## Silhouette Hierarchy

In any given frame, the player's eye processes visual information in a strict priority order. This hierarchy is engineered, not assumed. Every element's visual prominence must respect this ranking. If a lower-priority element visually competes with a higher-priority element, the lower element must be subdued.

1. **Player silhouette.** The single most important visual element on screen at any time. Must be instantly distinguishable from all environment geometry under all lighting conditions, at all distances, at all speeds. If there is any ambiguity about whether a shape is a player or a map element, the silhouette system has failed.

2. **Team color indicator.** Must be readable at any distance where a player is visible. At close range, team color appears on the character model itself. At medium range, it reads as a colored highlight or trim. At long range, it reduces to a colored dot or aura. The team color must never be confused with environmental colors — this is enforced at the palette design level.

3. **Weapon type.** Communicates the threat the player poses. At medium range, weapon silhouette identifies class (hitscan, projectile, beam). At close range, individual weapon identity is readable. Weapon visibility must not be obscured by character geometry or VFX.

4. **Add-on visual tells.** If a player has an active add-on that alters gameplay (shield, speed boost, special ability), a corresponding visual tell must be visible. This tell is always secondary to team color — it overlays or surrounds the player but does not replace or obscure the team indicator.

5. **Species visual tells.** The subtlest layer. Species differences are visible at close range (skin material, proportions, emissive accents) but should never compete with the four layers above. Species is identity information, not gameplay-critical information.

---

## Color Language

Color in JuanSchott is a communication system. Every color carries defined meaning, and no color appears in the game without a reason. This table defines the core color assignments. These are not suggestions — they are rules.

| Color | Hex Reference | Meaning | Where Used |
|-------|---------------|---------|-----------|
| Warm red / orange | `#D94F3D` / `#E8874A` | Enemy team, danger, damage dealt | Enemy player highlights, damage vignette, hostile UI elements, enemy nameplates |
| Cool blue / teal | `#4A90D9` / `#3DBDB5` | Friendly team, safety, allied presence | Friendly player highlights, team UI, spawn zones, ally nameplates |
| Gold / amber | `#D4A843` / `#C9963A` | Pantheon elements, rewards, judgment | Pantheon architecture accents, reward pickups, judgment event indicators, sacred geometry highlights |
| Bioluminescent green-cyan | `#00E5A0` / `#4AEDC4` | Alien / Vimana technology, active systems | Energy sources, beam weapons, Vimana active systems, powered architecture, Pantheon technology |
| Deep red | `#8B1A1A` / `#A02020` | Critical damage, death, fatal threat | Low health indicator, kill feed entries, death screen overlay, execution prompts |
| White / bright neutral | `#F0EDE8` / `#E8E4DC` | Information, HUD, neutral data | Crosshair, text, scores, neutral UI elements, cursor, objective markers |

### Color Interaction Rules

- **Team colors are reserved.** Warm red/orange and cool blue/teal must never appear as significant environmental colors. If a map's palette naturally trends toward either team color, the palette is adjusted, not the team color. Team readability is a gameplay requirement, not an aesthetic preference.
- **No unassigned saturated colors.** Every saturated hue in the game must map to one of the defined meanings above. If an artist introduces a new saturated color, it must be assigned a meaning or removed. Unsaturated, neutral tones (grays, tans, muted earth tones) are available for environmental use without restriction.
- **Desaturation at distance.** Environmental colors desaturate naturally with distance through atmospheric perspective. Team colors and weapon energy colors do not — they maintain saturation to preserve readability. This differential desaturation is a deliberate rendering choice.
- **No rainbow environments.** Any scene using more than six distinct hues in significant surface area has violated the palette discipline. Complexity comes from value and material variation, not from hue quantity.

---

## Distance Readability

Visual elements must function at every engagement distance the game supports. The readability requirements change with range, and each distance bracket has specific deliverables.

### Close Range (0–10 meters)

This is the intimate distance. Players at this range are either teammates in coordination or enemies in direct combat.

- Full character detail visible: species features, add-on tells, material surface quality, facial features (if exposed).
- Individual weapon components readable: barrel, grip, trigger assembly, emissive elements.
- Surface material detail distinguishable: metallic versus matte, polished versus rough, organic versus synthetic.
- Environmental surface carvings and patterns fully visible. Material transitions clear.
- Damage direction indicators precise — the red vignette points exactly to the source.

### Medium Range (10–30 meters)

This is the primary combat engagement distance for most weapons. Readability here is gameplay-critical.

- Player silhouette is the dominant visual signal. The capsule form and weapon extension read clearly.
- Weapon type recognizable by silhouette class: angular (hitscan), bulky (projectile), slim-glowing (beam).
- Team color clear and unambiguous. Applied as a trim, highlight, or overlay on the character.
- Add-on active effects visible as glows, particles, or aura effects surrounding the character.
- Environmental detail reduced — patterns on walls become texture-like, but surface material still reads.

### Long Range (30–60 meters)

This is the distance of tactical awareness. Players at this range represent potential threats or opportunities.

- Player reads as a moving shape. Team color is the sole reliable identifier.
- Weapon type is barely visible — only the broadest silhouette distinction (long versus short) is readable.
- No fine detail. No species differentiation. No add-on specifics. Pure silhouette and color.
- Movement pattern becomes the secondary identifier: a player moves differently from floating debris or map animation.

### Very Long Range (60+ meters)

At extreme range, players are abstract signals.

- Player reads as a colored dot with movement. The team color dot must remain saturated against any background.
- Deliberate contrast is maintained between player colors and the map's environmental palette. This is verified per-map during lighting and color pass.
- No silhouette detail whatsoever. A moving colored pixel is the entire visual deliverable.
- Environmental geometry at this range serves as backdrop only — atmospheric haze and desaturation create visual depth.

---

## Environmental Visual Language

### Vimana Surfaces

The Vimana's surfaces tell a story of alien construction, ageless preservation, and sacred purpose. Their visual language is consistent across all maps.

**Base material:** Warm, stone-like appearance. Not actual stone — the material is something else entirely, something that does not exist on Earth — but it reads as warm, matte, and ancient. The base color sits in the sandy beige to pale gold range. It feels like it has always been here and will always be here.

**Carved patterns:** Geometric, regular, alien. Lines and shapes repeat in ways that suggest mathematical meaning — fractals, tessellations, non-Euclidean arrangements — but never resolve into any recognizable symbol or script. Patterns are either lighter or darker than the base surface, creating contrast through value rather than color. The patterns are the Vimana's voice: they say "this was built by intelligence" without saying what that intelligence is.

**Edges:** Clean and sharp. No weathering. No rust. No decay. No patina. No erosion. The Vimana is ageless. Its surfaces do not degrade. A pillar that has stood for millennia looks as though it was placed yesterday. This is not neglect — it is a property of the material itself. The absence of wear communicates supernatural preservation and reinforces the player's smallness against something that does not age.

**Ambient glow:** Faint bioluminescence emanates from seams between architectural elements and from the carved patterns themselves. This glow is subtle — it creates atmosphere, not illumination. It suggests that the Vimana is not inert; it has energy, purpose, processes occurring beneath the surface. The glow pulses on a slow cycle (8–12 seconds), barely perceptible, giving the architecture a respiratory rhythm.

### Floating Structures

Floating geometry is a signature visual element of the Vimana. It communicates the Pantheon's mastery over physics and creates dynamic spatial compositions impossible in conventional architecture.

- **Deliberately geometric forms:** Cubes, pyramids, cylinders, polygonal prisms. No organic shapes. No erosion. No irregularity. Each floating element is a pure geometric solid, as if cut from the void by mathematical intent.
- **Energy tethers:** Visible as thin beams of light connecting the floating element to a larger structure or to the ground plane below. Tethers are hair-thin at distance, slightly thicker at close range, and always carry the bioluminescent green-cyan accent color. They are the only visible explanation for why these masses defy gravity.
- **Subtle drift:** Floating platforms shift position on a slow cycle — centimeters of movement over tens of seconds. This drift is barely noticeable in a single glance but creates a living quality over time. The movement is never enough to affect gameplay (platforms do not drift out of map bounds or create new sightlines), but it is enough to make the environment feel alive.

### Gravity Anomaly Zones

Gravity anomalies are areas where the Vimana's spatial manipulation creates localized physics distortions. Their visual treatment is unique and carefully controlled.

- **Visual distortion:** A slight chromatic shift at the zone's boundaries — elements near the edge separate into faint color fringes. This is the one acceptable use of chromatic aberration in JuanSchott. It is restricted to gravity anomaly zones and must never appear anywhere else in the rendering pipeline.
- **Floating debris:** Small objects — fragments of architecture, dust particles, loose geometric shapes — suspended within the anomaly zone. These objects drift slowly in random patterns, their motion governed by the anomaly's internal physics rather than standard gravity. The debris serves as both visual indicator and spatial anchor, helping players perceive the zone's boundaries.
- **Color shift:** The palette within an anomaly zone shifts slightly cooler than the surrounding environment. Warm tones desaturate. Shadows deepen with a blue-tinged cast. The player's eye registers the shift before the conscious mind identifies the cause — the zone feels wrong before the player understands why.

---

## Motion and Animation Language

Movement in JuanSchott communicates physicality and intent. The animation language supports gameplay readability while maintaining the aesthetic standard of fluid, deliberate motion.

**Player movement:** Fluid but deliberate. No floaty physics feel. Every movement has a sense of weight — a running character leans into motion, a stopping character shifts weight backward, a turning character's body follows the direction change with natural momentum. The animation must feel like a physical body moving through space, not a camera gliding across a surface. Acceleration and deceleration curves are visible in the animation — movement starts and stops with weight, not instantaneously.

**Weapon animations:** Snappy and precise. Draw in 0.3 seconds. Reload in 2.0 seconds. Switch in 0.5 seconds. These timings are gameplay requirements, and the animations must fill them without feeling rushed or sluggish. A 0.3-second weapon draw is three to four frames of decisive motion — the weapon snaps to ready position. A 2.0-second reload is a sequence of efficient, mechanical actions — magazine out, magazine in, charge. Every frame communicates purpose. No wasted motion.

**Map elements:** Subtle and atmospheric. Floating platforms drift on slow cycles. Vimana carved patterns pulse gently on their 8–12 second respiratory rhythm. Energy tethers shimmer faintly. Nothing in the environment is truly static — the Vimana is alive at a tempo below combat speed. The environmental animation serves atmosphere, never distraction. If a player stops to watch a floating platform drift, that is acceptable. If a player is distracted by environmental animation during combat, the animation is too aggressive.

**Death:** Instant ragdoll. No dramatic death animations — no staggering, no clutching wounds, no slow falls. The character drops. Physics takes over. This keeps the visual language clean during combat: alive or not alive. No ambiguous dying states. No lingering visual noise. Death is fast because combat is fast. The ragdoll should settle within 2–3 seconds to prevent bodies from becoming spatial confusion.

---

## VFX Language

Visual effects in JuanSchott are disciplined. Every particle, every flash, every screen effect exists to communicate specific gameplay information. Decorative VFX — effects that exist purely for spectacle — are prohibited. The VFX system is a communication layer, not a cinematic layer.

**Muzzle flash:** Brief, bright, small. Two to three frames of visible light at the weapon's firing point. Not a fireball. Not a bloom explosion. A precise flash that communicates "a shot was fired from here" and then disappears. The flash should be small enough to not obscure the target but bright enough to reveal the shooter's position.

**Tracers:** Thin lines for hitscan weapons, visible for approximately 0.2 seconds. The line traces from muzzle to impact point and fades. Thick, continuous beams for beam weapons — the beam persists as long as the trigger is held, its width communicating the weapon's area of effect. Projectile weapons leave no tracers — the projectile itself is the visible flight path.

**Impact effects:** Small sparks at bullet impact points on hard surfaces. Small dust puffs on soft surfaces. No explosions for bullet impact. Explosive projectiles create a brief, contained burst — bright flash, small debris, quick fade. Impact effects communicate "something hit here" without filling the screen with particles.

**Damage indicators:** Screen-edge red vignette that intensifies with damage volume. Directional — the vignette is thicker on the side of the screen facing the damage source, allowing the player to identify threat direction without taking their eyes from the crosshair. No floating damage numbers. No hit markers on the receiving end. Clean, minimal, functional.

**Healing:** Faint green pulse that radiates outward from the healed player's center. One pulse per heal tick. Minimal — the effect should register without distracting. The green pulse is distinct from the alien green-cyan of Vimana technology (warmer, more yellow-green) to prevent confusion between healing and energy effects.

**Abilities:** Each add-on has a defined VFX brief documented in [Addon System](../02_game_design/addon_system.md). All add-on VFX must respect the following constraints: effect duration must not exceed the ability's active duration, effect intensity must be proportional to gameplay impact, and effect color must not conflict with team color readability.

---

## Implementation Notes

### Rendering Priorities

The rendering pipeline must enforce visual language rules at the technical level:

- Character rendering uses a separate pass that ensures silhouette edge contrast against the environment. Characters must never blend into architecture.
- Team color rendering maintains consistent saturation regardless of lighting conditions. A character in deep shadow must still display readable team color.
- Emissive elements (Vimana glow, weapon energy, add-on effects) are rendered with controlled bloom that does not bleed into adjacent geometry or obscure player silhouettes.
- Atmospheric perspective (distance desaturation) is applied to environmental geometry but exempted from player characters and gameplay-relevant elements.

### Validation Checklist

Every map, character, and weapon must pass the following visual language checks before approval:

- [ ] Player character reads as distinct from environment at all distances
- [ ] Team color is identifiable at maximum render distance
- [ ] Weapon class is readable by silhouette at medium range
- [ ] No environmental color conflicts with team color assignments
- [ ] Silhouette hierarchy is maintained: player > team > weapon > addon > species
- [ ] All VFX serve gameplay communication, not decoration
- [ ] Vimana surfaces follow the defined material and pattern language
- [ ] Animation timings match specified durations
- [ ] No unassigned saturated colors in the scene

---

## See Also

- [Art Direction](art_direction.md) — Philosophy, principles, and high-level visual identity
- [Material and Lighting](material_and_lighting.md) — Technical specifications for materials, lighting setups, and rendering pipeline
- [UI and HUD Design](ui_and_hud_design.md) — Visual design for user interface, HUD elements, and information display
