# Anti-Goals

> Anti-goals explicitly define what JuanSchott is NOT. They exist to prevent scope creep, misalignment, and feature bloat. If someone proposes a feature that violates an anti-goal, the answer is no.

## JuanSchott Is NOT a Speed Demon Arena Shooter

**Reject:** Quake Champions, Unreal Tournament, Tribes-level movement speed and air control.

JuanSchott's movement is tactical and precise. Wall-running and double-jumping are **positioning tools**, not evasion mechanics. The game should never feel like players are flying around the map too fast to track. Speed is deliberate, earned, and readable.

**Concrete limits:**
- Maximum ground speed: 9.0 m/s (sprint) — not 15+ m/s like arena shooters
- Air control is limited (30% of ground control) — you commit to your jump trajectory
- No bunny hopping, no air-strafe acceleration, no rocket jumping equivalents

---

## JuanSchott Is NOT a Hero Shooter

**Reject:** Overwatch, Paladins, Apex Legends character-based design.

Every player shares the same base frame. There are no heroes, no classes, no unique character abilities. The add-on system provides strategic customization within a shared framework. Two players with the same loadout should play identically. Skill expression comes from aim, movement, positioning, and map knowledge — not from character selection.

**Concrete rules:**
- No hero/class selection screen
- No character-specific abilities
- No character-specific movement mechanics
- Add-ons are equippable items, not character-defining traits

---

## JuanSchott Is NOT a Military Simulator

**Reject:** Call of Duty, Battlefield, Insurgency, Squad.

This is a science-fiction arena game. There is no attempt at realism in weapons, movement, or setting. Weapons can be energy-based, projectile-based, or beam-based. Movement defies conventional physics. The setting is a god-built satellite. Fun and readability always trump realism.

**Concrete rules:**
- No realistic weapon recoil patterns
- No bullet drop for hitscan weapons
- No stamina systems
- No realistic damage modeling (no limb-specific damage)
- No military ranks, factions, or aesthetics

---

## JuanSchott Is NOT a Battle Royale

**Reject:** Fortnite, PUBG, Apex Legends (BR mode), Warzone.

Matches are small, focused, and structured. 1v1, 3v3, or 6v6. There is no shrinking zone, no loot scavenging, no 100-player chaos. Every player starts with their chosen loadout. Every match is a controlled competitive environment.

**Concrete rules:**
- No battle royale mode
- No looting or inventory management
- No shrinking play area
- No player count above 12 (6v6 maximum)

---

## JuanSchott Is NOT Pay-to-Win or Grind-to-Win

**Reject:** Any progression system where time or money buys power.

All weapons and add-ons are available to all players. There is no progression system that gates competitive advantage. If a progression system exists, it is cosmetic only (skins, titles, visual flair). A brand new player has access to the same tools as a 1000-hour veteran.

**Concrete rules:**
- No gameplay-affecting unlocks
- No stat-boosting purchases
- No "starter weapons" that are worse than "unlocked weapons"
- Cosmetics, if any, are strictly visual

---

## JuanSchott Is NOT a Story-Driven Single-Player Game

**Reject:** Halo campaign, Doom Eternal, Half-Life narrative experiences.

There is no single-player campaign. The lore is delivered through the LoL/Overwatch model — environmental storytelling, character identities, seasonal arcs, cinematics, and a codex. The multiplayer match is the core experience. Story enhances context; it does not replace gameplay.

**Concrete rules:**
- No single-player campaign
- No cutscenes during gameplay
- No NPCs to talk to
- No dialogue trees
- Lore is optional — players can ignore it entirely and still enjoy the game

---

## JuanSchott Is NOT Maximum Visual Fidelity

**Reject:** Cyberpunk 2077, The Last of Us Part II, photorealistic rendering.

The art style is deliberately low-poly. This is not a limitation — it is a design choice. The game looks premium through materials and lighting, not through polygon count or texture resolution. The visual identity should be instantly recognizable as JuanSchott, not mistaken for "a game trying to look realistic."

**Concrete rules:**
- No photorealistic textures
- No high-poly character models
- No ray-traced global illumination requirement
- No visual effect that drops framerate below the target
- Beauty through material quality, not geometric complexity

---

## JuanSchott Is NOT a Closed Development Project

**Reject:** Proprietary game development, closed-source engines, private roadmaps.

Everything is open. Code, assets, design documents, decisions. The development process is transparent. Contributors can see what is being worked on, why decisions were made, and how to help. The project succeeds or fails in public.

**Concrete rules:**
- All code is MIT/Apache 2.0 licensed
- All design documentation is public
- All development happens in public repositories
- No private design decisions — if it affects the game, it is documented
- Contributors are credited

---

## Summary Table

| Anti-Goal | Rejects | Boundary |
|-----------|---------|----------|
| Not an arena shooter | Quake/UT speed | Tactical precision, not chaos |
| Not a hero shooter | Overwatch characters | Same base frame, modular add-ons |
| Not a military sim | CoD/Battlefield realism | Sci-fi, readability over realism |
| Not a battle royale | Fortnite/PUBG | Small structured matches |
| Not pay/grind-to-win | Any P2W/G2W system | All gameplay tools available to all |
| Not story-driven | Halo/Doom campaigns | Lore as context, not core |
| Not max fidelity | Photorealism | Low-poly high-finish by design |
| Not closed development | Proprietary games | Fully open-source |

## See Also

- [Design Pillars](./design_pillars.md) — what JuanSchott IS (the positive counterpart to this document)
- [Vision and Origin](./vision_and_origin.md) — the full project vision
- [Decisions Log](./decisions_log.md) — rationales for every major choice
