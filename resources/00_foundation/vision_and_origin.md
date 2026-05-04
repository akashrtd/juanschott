# Vision and Origin

> "One Shot. God's Blessing. The Ship."
> — The three meanings of JuanSchott

## Logline

JuanSchott is an open-source, competitive multiplayer FPS set inside a god-built satellite arena, where tactical precision gunplay meets parkour movement across fully three-dimensional vertical spaces, rendered in a low-poly high-finish art style.

## Vision Statement

JuanSchott exists because modern multiplayer FPS games have abandoned two things that once defined the genre: **optimization** and **spatial creativity**. Games ship requiring dedicated GPUs for flat maps where every fight happens on the ground. Movement has been simplified to sprint + slide. Maps are two-dimensional layouts with cosmetic height variation.

JuanSchott reclaimss both. Every surface is a potential path. Every dimension is a combat space. Wall-running, double-jumping, and sliding turn positioning into a skill as important as aim. The low-poly high-finish art style proves that visual beauty does not require hardware brute force. And the entire project is open-source, built on Bevy, growing with the engine and the community.

The game is set in a science-fiction universe where a mysterious satellite arena called the **Vimana** appears in Earth's orbit, operated by a **Pantheon** of godlike beings who demand entertainment in exchange for humanity's survival. Fighters enter the Vimana, compete in matches, and the collective quality of their performance over a season determines whether Earth's dying biosphere heals or deteriorates further. The name **JuanSchott** is both the protagonist's title and a layered multilingual pun: "One Shot" in competitive gaming parlance, "God's Blessing" and "Ship" in the world's lore.

## How This Idea Started

The project began with a question: **how do you build an FPS that runs well on old hardware without looking like it belongs in 2003?**

### Phase 0: Pure Software Rendering (Abandoned)

The initial approach was radical — build a purely software-rendered FPS using compute shaders and SIMD, bypassing the GPU entirely. Research went deep:

- **olive.c** was evaluated as a potential software rasterizer base. A minimalist C library for pixel-level framebuffer manipulation. Too limited for what was needed.
- **Doom and Quake source code** was studied extensively. Doom's span-based column rendering, its surface cache for pre-baked lit textures, and its colormap-based lighting. Quake's BSP traversal, edge-list rasterization, PVS (Potentially Visible Set) culling, and affine texture mapping with perspective correction.
- **Compute budgets** were calculated for 1080p at 120fps using SIMD/AVX2: approximately 2.6 million pixels per frame, each requiring texture sampling, lighting, and blending. The math worked on paper but the visual ceiling was clear.
- **Conclusion:** Software rendering at this resolution would produce visuals equivalent to Quake 3 or Counter-Strike 1.6. Impressive for 1999, not for a modern game that aspires to a distinct visual identity.

The decision was made to **abandon pure software rendering** and rethink the entire approach. The core values — optimization, accessibility, performance on modest hardware — remained. The method changed.

### Phase 1: Engine Evaluation

With software rendering off the table, the question became: **which open-source engine can deliver modern visuals on old hardware while being future-proof?**

Four engines were evaluated in depth:

| Engine | License | Stars | Verdict |
|--------|---------|-------|---------|
| **Godot 4** | MIT | 110K | Strong contender. OpenGL Compatibility mode for old hardware. Massive community. Stable API. Full editor. Production-proven. |
| **Bevy** | MIT/Apache | 45.9K | Pre-1.0 Rust ECS engine. Superior architecture for FPS (native ECS, automatic parallelism, Rust safety). No editor. Breaking changes every 3-5 months. |
| **O3DE** | Apache 2.0 | 8K | Feature-complete but cumbersome. AWS-dependent tooling. Small community. |
| **Flax Engine** | BSD-3 | 6K | Capable but proprietary-company-controlled. Small community. |

Initial recommendation leaned toward **Godot 4** for stability and community size. But the user's philosophy shifted the calculus: this would be an open-source project that **grows with Bevy** via community contribution. Breaking changes become migration PRs, not private suffering. The ECS architecture is fundamentally better for an FPS. Rust's safety guarantees matter for netcode.

**Bevy 0.18** was selected.

### The Vision Crystallizes

With the engine decided, the design conversation revealed the full scope:

- **Movement** inspired by Titanfall 2 — wall-running, double-jumping, sliding — but tuned for tactical precision, not speed chaos
- **Maps** that use all three dimensions — mid-air cover, floating platforms, wall-run surfaces. The ground is just one of many surfaces
- **Weapons** spanning hitscan, projectile, and beam types
- **A modular add-on system** where everyone shares the same base frame but customizes with limited slots for passive stats and active abilities
- **Low TTK** — positioning and first-shot advantage matter
- **Regenerating health** — keeps the pace moving
- **Game modes**: 1v1 (practice), 3v3 (ranked), 6v6 (casual)
- **Art style**: low-poly, high-finish — minimal geometry, no textures, flat color with premium materials and lighting
- **Lore**: the satellite arena, the Pantheon, the seasonal stakes, Earth dying from past failures

The name **JuanSchott** emerged organically. Juan = "One" (competitive callout) + "God's Blessing" (Sanskrit/Spanish). Schott = "Shot" (FPS) + "Ship" (the Vimana). Three layers of meaning, one identity.

## What Makes JuanSchott Different

| Aspect | Most FPS Games | JuanSchott |
|--------|---------------|------------|
| **Map design** | Flat ground with height variation | Full 3D — every surface is playable |
| **Movement** | Sprint + slide | Wall-run, double-jump, slide — positioning as skill |
| **Art** | Texture-heavy, GPU-dependent | Low-poly, high-finish — runs on old hardware |
| **Engine** | Proprietary, closed | Open-source Bevy — grows with the community |
| **Monetization** | Battle passes, loot boxes, skins store | Fully open-source |
| **Lore delivery** | Cutscene campaigns or nothing | LoL/Overwatch model — environmental storytelling, seasonal arcs |
| **Customization** | Hero-based or none | Modular add-on system — same base, strategic slots |
| **Stakes** | Match-to-match or none | Seasonal — collective performance shifts Earth's orbit |

## Project Philosophy

1. **Open-source first.** Every line of code, every asset, every design decision is public. The project grows through community contribution.
2. **Performance is a feature.** 1080p at 120fps on decent hardware. 720p at 60fps on a potato. Optimization is not a post-launch task.
3. **Design with purpose.** Every mechanic, every art choice, every lore element exists for a reason. Nothing is added because other games have it.
4. **Grow with the engine.** Bevy is pre-1.0. Breaking changes are expected. The project treats migrations as community events, not crises.
5. **The setting justifies the gameplay.** The Vimana was built for spectacle. Wall-run surfaces exist because the Pantheon wants to see acrobatic combat. Mid-air cover exists because flat-ground fights are boring to gods. Every design choice has an in-universe explanation.

## Target Audience

- **Primary**: Competitive FPS players who value movement skill and positioning (Titanfall 2, Quake, Apex Legends veterans)
- **Secondary**: Players with modest hardware who feel locked out of modern FPS games
- **Tertiary**: Open-source contributors and Rust/Bevy developers interested in game development
- **Aspirational**: Esports organizers looking for a transparent, community-owned competitive FPS

## Platforms

- **Launch**: PC (Windows, Linux, macOS via wgpu)
- **Future**: WebAssembly (Bevy supports wasm via WebGPU/WebGL)
- **Not planned**: Consoles (open-source project, no publisher relationships)

## Business Model

JuanSchott is **free and open-source**. MIT/Apache 2.0 license (matching Bevy). No monetization planned. The project exists to be played, contributed to, and learned from.

## See Also

- [Design Pillars](./design_pillars.md) — the 5 rules governing every design decision
- [Anti-Goals](./anti_goals.md) — what JuanSchott is explicitly NOT
- [Decisions Log](./decisions_log.md) — every major decision and its rationale
