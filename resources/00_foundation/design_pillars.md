# Design Pillars

> These five pillars govern every design decision in JuanSchott. If a feature, mechanic, or system does not serve at least one pillar, it does not ship.

## Pillar 1: Positioning Is King

**Statement:** The fight is often decided before the first shot is fired.

Low TTK means that whoever has the better angle, the better height, the better cover advantage wins most engagements. Movement mechanics (wall-run, double-jump, slide) are positioning tools, not evasion tools. The best players are those who arrive at the right place at the right time, not those with the fastest reflexes.

**This pillar demands:**
- Maps with multiple viable routes and positional options at every engagement distance
- Movement mechanics that reward spatial awareness and route planning
- Information asymmetry — players who know the map better have a real advantage
- Sound design that communicates positioning (footsteps, wall-run audio cues)
- Kill times fast enough that raw aim rarely wins against positional advantage

**This pillar rejects:**
- Bullet-sponge enemies who can survive being caught out of position
- Movement mechanics designed primarily for dodging (air-strafe dodging, bunny hopping evasion)
- Random elements that override positioning (random spread, RNG mechanics)

---

## Pillar 2: Full 3D Combat

**Statement:** Combat happens everywhere — on the ground, on walls, in the air, on floating platforms, on ceilings. The ground is just one of many surfaces.

Most FPS maps are 2D layouts with cosmetic height. JuanSchott maps are volumes. Every match involves constant vertical movement. Players think in terms of "where am I relative to everyone" across all three axes, not just "who has the better angle on this corridor."

**This pillar demands:**
- Map geometry that makes full use of vertical space — floating structures, wall-run paths, multi-level arenas
- Movement mechanics that make vertical traversal fluid and intuitive
- Weapon balance that accounts for elevation and 3D positioning (projectile arcs, hitscan falloff)
- Camera and HUD design that helps players maintain spatial awareness
- Audio design with vertical cues (footsteps above, wall-run to the left)

**This pillar rejects:**
- Maps where 90% of combat happens on the ground floor
- Chokepoints that funnel all players into flat corridors
- Movement penalties for vertical play (no "fall damage for trying to be creative")
- Line-of-sight design that only considers horizontal planes

---

## Pillar 3: Strategic Customization, Not Power Creep

**Statement:** Every player starts from the same base. Customization creates strategic variety, not statistical superiority.

The add-on system gives players 3 slots to customize their loadout with passive stat modifiers and active abilities. These create meaningful strategic choices — do you counter the enemy's loadout or double down on your own? — without creating an arms race. A stock player with superior positioning beats a loaded-out player who is out of position.

**This pillar demands:**
- Strictly limited customization slots (3) to force meaningful choices
- Tradeoffs baked into every add-on (+speed means -something else, or an active ability has a cooldown)
- Counter-play built into the system (every add-on has a counter add-on)
- Clear visual and audio cues so opponents can read your loadout
- No progression-gated power (all add-ons available from the start, or unlock through play not pay)

**This pillar rejects:**
- Add-on combos that are strictly better than other combos (no "meta builds")
- Raw stat increases without tradeoffs
- Abilities that bypass positioning (teleportation, instant repositioning without risk)
- Any form of pay-to-win or grind-to-win

---

## Pillar 4: Clean and Readable

**Statement:** The player should always understand why they died, where the enemy is, and what is happening. Visual clarity is non-negotiable.

Low-poly high-finish art is not just an aesthetic choice — it is a **competitive advantage**. Minimal geometry means fewer visual distractions. No textures means no camouflaged enemies. Strong silhouettes mean instant player recognition. The premium materials and lighting create beauty without creating noise.

**This pillar demands:**
- Player models with instantly recognizable silhouettes against any background
- Minimal visual clutter — no particle spam, no excessive VFX, no screen shake that obscures
- Clear audio hierarchy — every sound communicates specific information
- UI that shows what matters and hides what doesn't
- Consistent color language — red means danger, blue means team, specific colors for specific weapons

**This pillar rejects:**
- Volumetric fog, bloom, or post-processing that obscures player vision
- Cosmetic customization that breaks silhouette readability
- Visual effects that interfere with gameplay clarity
- Camera effects that disorient (excessive motion blur, chromatic aberration)

---

## Pillar 5: Built to Last, Built Together

**Statement:** The project grows with Bevy, grows with the community, and is designed for years of development, not months.

JuanSchott is open-source. It will be maintained, migrated, and improved by contributors over time. Every technical decision prioritizes maintainability, readability, and contributor onboarding. The seasonal framework creates a natural content pipeline that keeps the game fresh without requiring constant developer attention.

**This pillar demands:**
- Clean, documented codebase with consistent conventions
- Architecture that is modular — new weapons, maps, and add-ons can be added without touching core systems
- Comprehensive documentation (this resource folder) so new contributors can onboard quickly
- Build process that is simple: clone, cargo run, play
- Seasonal framework that structures content creation

**This pillar rejects:**
- Technical debt that accumulates ("we'll refactor later")
- Monolithic architecture where adding a new weapon requires touching 10 files
- Unnecessarily clever code that is fast but unreadable
- Dependencies on proprietary or unmaintained libraries

## Conflict Resolution

When two pillars conflict, the resolution order is:

1. **Clean and Readable** (Pillar 4) overrides everything — if a feature makes the game harder to read, it doesn't ship
2. **Positioning Is King** (Pillar 1) overrides **Strategic Customization** (Pillar 3) — no add-on should negate positioning advantage
3. **Full 3D Combat** (Pillar 2) overrides convenience — if a feature flattens the game back to 2D, it doesn't ship
4. **Built to Last** (Pillar 5) is a tiebreaker — when equally valid options exist, choose the more maintainable one

## See Also

- [Vision and Origin](./vision_and_origin.md) — how the project started and where it's going
- [Anti-Goals](./anti_goals.md) — what JuanSchott explicitly is NOT
- [Player Experience](../02_game_design/player_experience.md) — how pillars translate to player feelings
