# Player Experience

**Project:** JuanSchott
**Document Type:** Game Design Document — Experience Design
**Version:** 1.0
**Last Updated:** 2026-05-05
**Owner:** Design Team
**Status:** Active

---

## Purpose

This document defines what JuanSchott should **feel** like to play. It is not a mechanical specification — that responsibility belongs to the core gameplay loop, movement system, and weapon system documents. This document is concerned with the emotional and tactile texture of the experience: the sensations that remain with a player after they close the game, the microsecond-level responses that build trust between player and software, and the macro-arc of a session that transforms mechanical interactions into meaningful memory.

Every design decision in JuanSchott is ultimately evaluated against the experience goals outlined here. If a mechanic is technically sound but violates these targets, the mechanic changes. Feel is non-negotiable.

---

## Experience Goals

These are the target feelings, listed in strict priority order. When goals conflict, the higher-priority goal wins.

### 1. Fluid Competence

> *"I am in control. My character does what I want, when I want. Movement feels like an extension of my intention."*

Fluid Competence is the bedrock. Without it, no other experience goal matters. A player who fights their own controls will never feel tactical tension, never appreciate spatial mastery, and never experience defiant spectacle. This goal mandates that every input the player issues is translated into on-screen action with minimal latency, maximal predictability, and zero ambiguity. The character is not a separate entity with its own agenda — it is a transparent vessel for the player's will.

This goal is measured by the gap between intention and execution. When a player thinks "I want to wall-run along that ledge and shoot the enemy below," the time between that thought and the on-screen result should approach zero. Not literally zero — physics, network conditions, and human reaction time make that impossible — but the *perceived* gap should be imperceptible.

### 2. Tactical Tension

> *"Every corner could have an enemy. Every position is a choice. I feel the weight of my decisions."*

Tactical Tension is the emotional engine of moment-to-moment play. It emerges from the interplay of map design, time-to-kill, and information asymmetry. The player should never feel truly safe — not because the game is unfair, but because the game respects them enough to present genuine threats. Every doorway is a question. Every sight line is a commitment. Every reload is a calculated risk.

Tension is not the same as anxiety. Anxiety is the feeling of not understanding why you died. Tension is the feeling of understanding exactly what's at stake and having to decide anyway. JuanSchott pursues the latter relentlessly.

### 3. Satisfying Impact

> *"When I land a shot, I FEEL it. When I die, I understand why. Feedback is instant and clear."*

Impact is the sensory reward loop. It is the reason players flinch at their own screen when they hear a headshot, the reason a well-placed grenade elicits a physical exhale. Impact is communicated through three channels simultaneously: visual, auditory, and — where appropriate — haptic. These channels must be synchronized to within a single frame. Desynchronized feedback shatters the illusion of consequence.

Impact also governs the negative feedback loop. When a player takes damage or dies, the game must communicate what happened, where it came from, and — crucially — what the player could have done differently. Clarity in death is the foundation of learning, and learning is the foundation of long-term engagement.

### 4. Spatial Mastery

> *"I know this map. I know where the wall-run paths are. I know the sight lines. My knowledge is my weapon."*

Spatial Mastery is the long-game emotion. It is the feeling a player has after ten hours, fifty hours, two hundred hours in the same map space. The map is not a backdrop — it is a character in the match, and understanding it is a skill as legitimate as aim or timing.

This goal mandates that maps be deep rather than wide. A small map with ten meaningful sight lines is superior to a large map with two. Every wall, ledge, and corner should serve a tactical purpose. The player who has invested time in learning the space should feel that investment rewarded through superior positioning, faster navigation, and better anticipation of enemy movement.

### 5. Defiant Spectacle

> *"I just did something impressive and I know it. The game acknowledged it. The Pantheon would approve."*

Defiant Spectacle is the peak experience — the moments players clip, share, and remember. It lives at the intersection of Fluid Competence and Spatial Mastery: a player who knows the map and commands the mechanics can produce sequences of play that surprise even themselves.

Crucially, spectacle in JuanSchott is **earned**, not granted. There are no cinematic kill cameras, no slow-motion trigger effects, no automated highlight reels. The game does not manufacture spectacle — it creates the conditions for players to manufacture it themselves. The game's role is to *acknowledge* the spectacle after the fact, not to generate it. A clean HUD notification. A satisfying audio cue. The knowledge that what just happened was genuinely difficult and the player pulled it off.

---

## Game Feel Targets

The following sections apply Steve Swink's three-level Game Feel framework — Micro, Meso, and Macro — to JuanSchott's specific design requirements. Swink defines Game Feel as "the tactile, kinesthetic sensation of interacting with a video game," and structures analysis across the input layer (what the player does), the response layer (what the game does back), and the experiential layer (what the player feels over time).

### Input Feel — Micro-Level (Swink's "Input" Layer)

Swink identifies the micro-layer as the domain of raw input processing: how the player's physical actions are captured, interpreted, and translated into game state. At this level, the goal is transparency. The player should never be aware that input processing is happening — it should feel as though their hands are directly manipulating the game world.

| Parameter | Target | Rationale |
|-----------|--------|-----------|
| Input latency budget | < 50ms from mouse click to muzzle flash | Competitive shooters demand sub-frame responsiveness. Above 50ms, players perceive delay and compensate with prediction, degrading Fluid Competence. |
| Mouse sensitivity | Configurable, default 0.002 (400 DPI equivalent) | Defaults target the competitive median. Full configurability respects individual preference and physical needs. |
| Input buffering (jump) | 6 frames (~100ms at 60fps) | Allows timing-tight jumps during fast movement without requiring frame-perfect input. Reduces frustration without reducing skill ceiling. |
| Animation priority | Player actions always complete to commitment | No animation canceling. Actions are designed to be fast enough that commitment never feels restrictive. This ensures predictability — a core tenet of Fluid Competence. |
| Dead zones | Minimal. Raw input respected. | Aim should feel 1:1. Any dead zone introduces a "mushy" sensation that violates the transparency goal. |

Additional micro-level considerations:

- **Input sampling rate** must match or exceed the display refresh rate. A 144Hz display must never present a frame where the game is using stale input data.
- **Analog movement** (WASD/stick) should support 8-directional digital input without analog gradient. Movement speed is binary — walk or sprint. This eliminates the "barely moving" dead zone problem that plagues analog stick implementations.
- **Scroll wheel** should not be bindable to jump. This is a deliberate design choice to prevent scroll-wheel jumping exploits that create an uneven skill floor.

### Responsiveness — Meso-Level (Swink's "Response" Layer)

The meso-level concerns the game's immediate feedback to player actions and game state changes. This is where the rubber of Game Feel meets the road of emotional design. Every response the game produces serves dual purpose: it communicates information and it generates sensation. Both must be considered simultaneously.

**Hit feedback (dealing damage):** Visual hit marker (small, clean, non-intrusive) plus audio ping plus subtle screen edge flash. No screen shake on hit. Screen shake violates Pillar 4 (Clean and Readable) by disrupting the player's ability to track targets during sustained combat. The hit marker must be small enough to not obscure the target but large enough to register in peripheral vision.

**Death feedback (player dies):** Quick fade to gray, respawn camera shows killer's position for 1 second, then fade into respawn. Total death-to-play time: 3–5 seconds. This window is long enough to communicate what happened and short enough to prevent emotional disengagement. The gray fade is deliberate — it signals finality without despair. The killer camera is educational, not punitive.

**Kill feedback (player scores a kill):** Satisfying audio cue plus clean HUD notification. No confetti, no obtrusive celebration, no slow-motion. The tone of JuanSchott's kill feedback is *respectful* — it acknowledges the achievement without diminishing the opponent. Over-celebratory kill feedback creates a toxic emotional asymmetry where winning feels glorified and losing feels humiliating.

**Damage taken feedback:** Screen edge vignette with red tint. Intensity scales with damage severity. Directional indicator showing damage source. No health bar animation — the vignette replaces the health bar as the primary "how hurt am I" signal. This keeps the HUD minimal and the information ambient rather than analytical.

**Camera behavior:** Locked to first-person at all times. No third-person moments, no cinematic cameras during gameplay. Camera bob is minimal and disable-able. The first-person lock is foundational to spatial immersion — it ensures that every visual the player receives is from their character's perspective, which is a prerequisite for Spatial Mastery.

### Emotional Experience — Macro-Level (Swink's "Context" Layer)

Swink's macro-level addresses the emotional arc of play over time — across a single engagement, a full match, and a multi-session play cycle. This is where individual moments of Game Feel accumulate into lasting emotional impressions.

**First 30 seconds of a match:** Tension. The player spawns, checks their loadout, and begins moving through the map. No enemies are visible yet, but the possibility of contact is omnipresent. Sound design carries this phase — footsteps echo, ambient environmental noise establishes the space, and the silence before combat is as informative as combat itself. The map should feel *occupied* even when empty.

**First kill of the match:** Relief and adrenaline. The extended tension of the opening phase pays off. The first kill establishes the emotional rhythm of the match — it transitions the player from "anticipation mode" to "engagement mode." The transition should feel like exhaling after holding your breath.

**Mid-match flow state:** Continuous flow. Moving, shooting, repositioning in a fluid loop. Time distortion occurs during good matches — players lose track of time because cognitive resources are fully allocated to the present moment. This is the target state for the majority of match duration. Map design, spawn logic, and game pacing all serve to sustain this state once entered.

**Clutch moment (last player alive in a team fight):** Intense focus. Sound design narrows — ambient noise reduces, footsteps become more prominent. HUD simplifies — non-essential elements fade. The game subtly adjusts its presentation to emphasize the 1vX situation without breaking the first-person commitment. This is the domain of Defiant Spectacle. A player who clutches a round should feel like they earned something extraordinary.

**Match win:** Warm satisfaction. Not euphoria — that is not JuanSchott's tone. The victory screen is clean, informative, and brief. Scoreboards are shown, MVP is acknowledged, and the player is guided toward the next queue. The emotional target is a quiet sense of competence: "I played well. I'd like to play well again."

**Match loss:** Understanding and determination. The loss screen presents the same information as the win screen — no punitive design, no shaming. The critical emotional target is that losses feel *educational*, not unfair. If a player can identify what went wrong — a bad position, a missed shot, a poor rotation — the loss is productive. If a player cannot identify what went wrong, the game has failed at clarity.

---

## Reference Games and Feel Targets

No game exists in a vacuum. The following table identifies specific reference games, what JuanSchott borrows from each, and — equally important — what JuanSchott deliberately rejects.

| Aspect | Reference Game | What We Take | What We Don't |
|--------|---------------|-------------|---------------|
| Movement fluidity | Titanfall 2 | Seamless movement-to-combat transitions. Wall-running momentum that accelerates rather than interrupts flow. The sense that movement IS combat. | Speed chaos — Titanfall 2's peak velocity can feel overwhelming. JuanSchott's movement ceiling is lower, trading speed for control. Titan mechanics are irrelevant. |
| Aim precision | CS2 / Valorant | Crisp, responsive aiming where first-shot accuracy matters. The mouse-to-crosshair relationship feels direct and trustworthy. | CS2's spray patterns (memorization-based skill). Valorant's agent abilities (combat modifiers). JuanSchott's aim skill is purely mechanical — no compensating factors. |
| Time-to-kill feel | Valorant | Low TTK where positioning wins engagements more often than raw aim. The dread of being one shot. | Agent abilities that modify TTK. Ultimate economy that creates asymmetrical engagements. JuanSchott's TTK is consistent and predictable. |
| Map spatial awareness | Quake 3 Arena | Vertical play, position timing, the sense that map knowledge is a weapon equal to aim. | Item pickups, arena-speed movement, spawn fragging. JuanSchott's maps are tactical, not athletic. |
| Clean visual feedback | Overwatch | Clear silhouettes, readable abilities, visual language that communicates game state at a glance. | Ultimate spectacles, visual noise, screen-filling effects that obstruct information. JuanSchott's feedback is understated. |
| Tactical pacing | Rainbow Six Siege | Pre-engagement tension, the value of information gathering, the weight of commitment to a position. | Destruction mechanics, glacial pace, operator gadget complexity. JuanSchott is faster than Siege but shares its respect for positioning. |

---

## Session Experience Mapping

The following table maps specific in-match moments to their intended emotional targets and the design mechanisms that achieve them. This serves as a verification checklist: if any moment's intended emotion is not being achieved, the corresponding "Achieved By" column identifies which systems to audit.

| Moment | Intended Emotion | Achieved By |
|--------|-----------------|-------------|
| Spawning | Focus and anticipation | Clean spawn protection, loadout confirmation visible, mental map of routes activated |
| Navigating empty map | Spatial awareness, planning | Ambient sound design, visible route options, no HUD distractions, environmental storytelling through map architecture |
| First enemy contact | Adrenaline spike | Audio cue (enemy footsteps or weapon sound before visual contact), sudden shift from ambient to combat soundscape |
| Winning a 1v1 duel | Satisfaction, validated competence | Clean kill confirm, visible damage taken (shows how close it was or how clean the play was) |
| Losing a 1v1 duel | Understanding | Clear death cam showing killer's position and actions, identifiable decision that led to the loss |
| Clutching a 1vX situation | Intense focus, flow state, Defiant Spectacle | Audio narrows, HUD simplifies, time perception shifts, pure skill expression under pressure |
| Match win | Warm satisfaction | Score screen with personal stats, clean UI, quick transition to next queue — no excessive celebration |
| Match loss | Determination | Clear feedback on team and personal performance, identifiable areas for improvement, immediate queue option |

---

## Accessibility and Feel

Accessibility is not a secondary consideration in JuanSchott's feel design — it is integral. A game that feels good only for a narrow demographic of players has failed at the most fundamental level.

**Redundant feedback channels:** All visual effects must have audio equivalents, and all audio cues must have visual equivalents. A player playing with sound off must receive equivalent information to a player with sound on, and vice versa. This is not a compromise — it is a design discipline that improves clarity for all players.

**Configurability:** The following must be configurable: mouse sensitivity, stick sensitivity, field of view, colorblind modes (Protanopia, Deuteranopia, Tritanopia), subtitle options, text size, and HUD opacity. Configurability is not "dumbing down" — it is allowing each player to calibrate the game to their perceptual and physical needs.

**Input complexity ceiling:** No mechanics require rapid repeated inputs. There is no manual bunny hopping, no wave dashing, no input sequences that test physical dexterity beyond the standard move-aim-shoot paradigm. Movement mechanics are hold-or-press. The skill ceiling lives in decision-making, timing, and spatial reasoning — not in physical execution speed.

**Comfort options:** Motion settings (camera bob, head bob, screen effects) must be individually toggleable. Photosensitivity considerations must inform all visual effect design — no strobing, flashing, or high-frequency visual patterns.

---

## Measurement and Validation

Experience goals are subjective by nature, but their achievement can be validated through structured playtesting:

- **Input feel validation:** Measured via high-speed camera input lag testing. Target: < 50ms confirmed on target hardware.
- **Responsiveness validation:** Player surveys using standardized Game Feel questionnaires, administered after 30-minute play sessions. Focus on "control" and "predictability" ratings.
- **Emotional arc validation:** Post-match emotional state surveys mapped against the Session Experience table above. Deviations flagged for design review.
- **Accessibility validation:** Playtesting with players who have varying visual, auditory, and motor abilities. Feedback channels audited for information parity.

---

## See Also

- `core_gameplay_loop.md` — Mechanical structure supporting these experience goals
- `movement_system.md` — Movement mechanics and their feel parameters
- `weapon_system.md` — Weapon feel, TTK, and feedback design
- `../03_art/ui_and_hud_design.md` — Visual feedback systems and HUD philosophy
- `../06_cross_cutting/audio_design.md` — Audio feedback, spatial sound, and soundscape design
