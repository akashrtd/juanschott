# Phase 5 — Content

**Version Target:** v0.5 (Beta)  
**Estimated Duration:** 8–10 weeks  
**Status:** Planning  
**Dependencies:** Phase 4 (v0.4) complete  

---

## 1. Overview

Phase 5 is the final phase of the JuanSchott roadmap. It takes the fully functional, visually polished multiplayer game and fills it with the content needed for a playable beta release. This includes three maps (one per game mode), the complete weapon roster, the complete add-on roster, fully defined game mode rules with scoring and win conditions, the in-game HUD, sound design, the menu system, and a basic seasonal content framework.

By the end of Phase 5, the game must support a complete session flow: the player launches the game, navigates the main menu, enters matchmaking or starts a local match, plays through a complete game with scoring and a win condition, sees the results on a score screen, and returns to the menu. This is the minimum viable product for closed beta testing.

This phase is content-heavy but also systems-heavy. The HUD, menu system, sound engine integration, and game mode rules are all new systems that must be built alongside the content they present.

---

## 2. Goals

- Build 3 playable maps, one designed for each game mode
- Complete the full weapon roster
- Complete the full add-on roster
- Implement game mode rules with scoring and win conditions
- Build the in-game HUD
- Integrate sound design
- Create the menu system
- Establish a basic seasonal content framework

---

## 3. Task Breakdown

### 3.1 Map Design and Construction

**Task 5.1: Map design principles**

Establish design principles for all JuanSchott maps:
- **Movement-friendly:** All maps must support the full movement suite (sprint, slide, double-jump, wall-run). Every area must be reachable using advanced movement. Verticality is essential.
- **Readable:** Map layout must be quickly learnable. Landmarks at key intersections. Distinct visual themes per area of the map to aid navigation.
- **Balanced:** Symmetric or carefully balanced asymmetric layouts for competitive fairness. No spawn-camping positions. No unreachable camping spots.
- **Mode-appropriate:** Each map is designed for a specific mode's pacing and player count.

**Task 5.2: Map 1 — Arena (Deathmatch / Duel)**

Design and build the arena map:
- **Scale:** Tight and intimate. Designed for 1v1 or 2v2. Sight lines are short. Engagement distance is close-to-medium.
- **Layout:** A central open area with surrounding corridors, platforms, and vertical routes. Multiple paths between any two points. At least 3 vertical layers (ground, mid, high).
- **Movement features:** Walls suitable for wall-running along corridors. Platforms requiring double-jump to reach. A long sight line for the slide mechanic. Ramps and drops for vertical flow.
- **Visual theme:** Inner temple courtyard. Open sky above. Surrounded by temple architecture with alien material accents. Central feature (a glowing alien monolith or similar landmark).

**Task 5.3: Map 2 — Complex (Team Deathmatch / Skirmish)**

Design and build the complex map:
- **Scale:** Medium. Designed for 3v3 or 4v4. Multiple engagement areas. Longer sight lines than the arena.
- **Layout:** An interconnected complex of rooms and hallways. 2–3 major engagement zones connected by corridors. Flanking routes. A central control point or chokepoint. Vertical spaces with multiple access points.
- **Movement features:** Multi-level rooms with wall-run surfaces. Gaps and platforms requiring precise movement. Open areas for slide-based traversal.
- **Visual theme:** Temple interior transitioning to alien underground. Contrasting warm (temple) and cool (alien) lighting zones. Distinct visual identity per room for navigation.

**Task 5.4: Map 3 — Nexus (Objective Mode)**

Design and build the nexus map:
- **Scale:** Large. Designed for objective-based play with team coordination. Longer travel times between areas.
- **Layout:** Symmetric layout with two team sides and a central objective area. Forward spawn points or respawn routes. Chokepoints and alternate paths. The objective area is the map's focal point.
- **Movement features:** Open approaches (high risk) and covered routes (slower but safer). Wall-run surfaces along the central area for dynamic flanking. Vertical perches for ranged fire support.
- **Visual theme:** Grand temple exterior with alien sky. Dramatic scale. The objective area glows with alien energy. Weather or atmospheric effects (light rain, floating particles) for ambiance.

**Task 5.5: Map production pipeline**

For each map:
1. Gray-box layout in Blender (simple geometry, no art — pure gameplay testing)
2. Playtest the gray-box for flow, balance, and movement support
3. Iterate on layout based on playtest feedback
4. Final art pass: replace gray geometry with themed models, apply toon shader materials, set lighting
5. Add collision geometry (manual collider placement for clean physics)
6. Final playtest with full art and lighting

**Estimated effort:** 25–35 hours (across all three maps)

---

### 3.2 Weapon Roster

**Task 5.6: Weapon roster definition**

Define the full weapon roster. Each weapon must have:
- Name, type (hitscan, projectile, beam), damage, fire rate, range, spread, projectile speed (if applicable), special mechanics
- Add-on slot configuration (how many slots, which types)
- Visual identity (color, shape, VFX color)
- Role in the weapon ecosystem (close-range, long-range, area denial, burst damage, sustained damage)

Target: 6–8 weapons covering all archetypes:
- **Pistol (hitscan):** Reliable sidearm. Moderate damage, moderate fire rate. 1 add-on slot.
- **Rifle (hitscan):** Precision weapon. High damage, low fire rate, low spread. 2 add-on slots.
- **SMG (hitscan):** Spray weapon. Low damage per hit, high fire rate, moderate spread. 2 add-on slots.
- **Shotgun (hitscan):** Close-range burst. Multiple pellets per shot, high spread, devastating at close range. 1 add-on slot.
- **Rocket Launcher (projectile):** Explosive projectile. High damage, slow fire rate, area of effect. 2 add-on slots.
- **Grenade Launcher (projectile):** Bouncing projectile with timed detonation. Area denial. 2 add-on slots.
- **Beam Rifle (beam):** Continuous damage beam. Moderate damage per second, limited range. 2 add-on slots.
- **Melee (special):** Instant close-range attack. High damage, no ammo. Always available as a fallback. 0 add-on slots.

**Task 5.7: Weapon implementation**

Implement each weapon from the roster:
- Create weapon data assets with all stats
- Implement firing mechanics for each weapon type (leveraging Phase 2's weapon system)
- Create placeholder view models for each weapon (geometry consistent with the art style)
- Wire up the weapon inventory system to support the full roster
- Balance-tune each weapon's constants against the others

**Task 5.8: Weapon balance pass**

Conduct a systematic balance pass:
- Each weapon should have a clear niche where it excels
- No weapon should be strictly better than another in all situations
- TTK should be consistent with design targets (1–2 seconds for focused fire)
- Test all weapon + add-on combinations for broken interactions
- Document balance rationale for each weapon's stats

**Estimated effort:** 15–20 hours

---

### 3.3 Add-On Roster

**Task 5.9: Add-on roster definition**

Define the full add-on roster. Target: 10–15 add-ons across slot types:
- **Scope slot:** Red dot sight, holographic sight, thermal sight
- **Barrel slot:** Suppressor (reduce visibility), extended barrel (increase range), choked barrel (reduce spread)
- **Magazine slot:** Extended magazine (increase capacity), quick magazine (faster reload), high-caliber magazine (increase damage)
- **Grip slot:** Ergonomic grip (reduce switch time), heavy grip (reduce recoil), lightweight grip (increase movement speed while aiming)

**Task 5.10: Add-on implementation**

Implement each add-on:
- Define stat modifications for each add-on
- Implement any special mechanics (e.g., thermal sight highlights players through walls briefly)
- Ensure all add-ons stack correctly when combined
- Test all valid add-on combinations per weapon

**Task 5.11: Add-on balance pass**

Balance the add-on ecosystem:
- Each add-on should provide a meaningful trade-off, not a pure upgrade
- No add-on combination should make a weapon dominant in all situations
- Add-ons that increase damage should come with a downside (reduced fire rate, increased spread, etc.)
- Playtest with competitive loadouts and validate variety in effective builds

**Estimated effort:** 10–14 hours

---

### 3.4 Game Mode Rules

**Task 5.12: Game mode framework**

Create a game mode framework that supports multiple modes:
- `GameMode` trait/enum defining: scoring rules, win conditions, time limits, spawn rules, team configuration
- `MatchState` resource tracking: scores, timer, current phase (warmup, active, overtime, ended)
- Mode-specific logic runs as Bevy systems that read `MatchState` and apply mode rules

**Task 5.13: Deathmatch / Duel mode**

Implement the deathmatch/duel mode:
- Free-for-all or 1v1
- Scoring: 1 point per kill. -1 point per suicide.
- Win condition: first to N kills (default: 20) or highest score when time expires (default: 10 minutes)
- Overtime: if tied at time expiration, sudden death (first kill wins)
- Map: Arena

**Task 5.14: Team Deathmatch / Skirmish mode**

Implement the team deathmatch mode:
- Two teams (2v2, 3v3, or 4v4)
- Scoring: team score = sum of individual kills. Team loses 1 point per team kill (if friendly fire is enabled).
- Win condition: first team to N kills (default: 50) or highest score when time expires (default: 10 minutes)
- Overtime: same as deathmatch
- Map: Complex

**Task 5.15: Objective mode**

Implement the objective mode:
- Two teams attack and defend a central objective (single-point capture or similar)
- Attacking team wins by capturing the objective within the time limit
- Defending team wins by preventing capture until time expires
- Roles swap after each round (best of 3 or best of 5)
- Map: Nexus

**Task 5.16: Match flow system**

Implement the full match lifecycle:
- **Pre-match:** Players load in, select loadouts, ready up
- **Countdown:** 5-second countdown before the match starts
- **Active:** Game mode rules in effect, scoring active
- **Round end:** When a round ends (time or score), brief intermission
- **Match end:** When the match is decided (best-of-N rounds or score limit), show results
- **Post-match:** Score screen, return to lobby

**Estimated effort:** 15–20 hours

---

### 3.5 HUD Implementation

**Task 5.17: HUD framework**

Create the HUD rendering framework:
- HUD elements are Bevy UI components rendered on a UI camera
- The HUD camera renders on top of the game camera
- HUD layout is resolution-independent (anchor-based positioning)
- HUD supports per-player display (different HUD for each player in split-screen, if applicable)

**Task 5.18: Health display**

Implement the health indicator:
- Current health as a number or bar
- Visual feedback when taking damage (flash, shake)
- Visual feedback when health is low (red tint, pulse)
- Regen state indicator (optional)

**Task 5.19: Ammo/weapon display**

Implement the weapon and ammo indicator:
- Current weapon name and icon
- Ammo count (if applicable — some weapons may not use ammo)
- Weapon switch indicators (show available weapons, highlight current)
- Reload indicator (if reload mechanic exists)

**Task 5.20: Crosshair**

Implement the crosshair:
- Center-screen crosshair that expands with weapon spread
- Dynamic: the crosshair visually widens during fire (spread), movement (movement spread), and airtime (inaccuracy)
- Color changes on enemy target (hitscan weapons: crosshair turns red when aimed at an entity with health)
- Crosshair style should be consistent with the game's clean aesthetic

**Task 5.21: Score display**

Implement the score display:
- Current score for each player/team
- Kill feed (last 3–5 kills shown with weapon icons)
- Match timer
- Game mode objective indicator (for objective mode: capture progress)

**Task 5.22: Damage indicators**

Implement directional damage indicators:
- When the player takes damage, show an arrow or flash on the screen edge indicating the damage source direction
- The indicator fades after 1–2 seconds
- Intensity reflects damage amount (small damage = small indicator, heavy damage = prominent indicator)

**Estimated effort:** 15–20 hours

---

### 3.6 Sound Design

**Task 5.23: Sound engine integration**

Integrate Bevy's audio system:
- Support simultaneous playback of multiple sound effects
- Support 3D spatial audio for positional sounds (weapon fire from other players, impacts)
- Support 2D non-spatial audio for UI sounds and the local player's weapon sounds
- Volume control per category (master, SFX, music, voice)

**Task 5.24: Weapon sound effects**

Create or acquire sound effects for all weapons:
- Fire sound per weapon type (distinct and recognizable)
- Reload sound (if applicable)
- Weapon switch sound
- Empty clip / out of ammo sound
- Impact sounds: surface hit, player hit, headshot (distinct from surface)
- Priority: fire sounds and impact sounds first (gameplay-critical), reload and switch sounds second

**Task 5.25: Movement sound effects**

Create or acquire sound effects for movement:
- Footsteps (walking, sprinting — different cadence)
- Jump and double-jump
- Land (from various heights — louder for higher falls)
- Slide start and slide end
- Wall-run contact
- Wall-jump launch

**Task 5.26: UI sound effects**

Create or acquire sound effects for UI interactions:
- Menu navigation (hover, select, back)
- Loadout selection (equip add-on, confirm loadout)
- Match countdown beeps
- Kill notification
- Match end fanfare

**Task 5.27: Ambient sound (stretch goal)**

If time permits, add ambient sound to maps:
- Map-specific ambient loops (wind, machinery, water, etc.)
- Environmental sound effects (doors, platforms, ambient activity)
- This is a lower priority than gameplay sounds

**Estimated effort:** 12–16 hours

---

### 3.7 Menu System

**Task 5.28: Main menu**

Create the main menu screen:
- Title and logo
- Play button (enters matchmaking flow)
- Local play button (starts a local match or listen server)
- Settings button (opens settings screen)
- Quit button
- Background: 3D scene or stylized static art consistent with the game's aesthetic

**Task 5.29: Settings screen**

Create a settings screen with:
- Video settings: resolution, fullscreen/windowed, quality preset (low/medium/high)
- Audio settings: master volume, SFX volume, music volume
- Controls: sensitivity, invert Y, key rebinding (basic)
- Network: server browser or direct connect (for listen server play)
- Settings persist to a local configuration file

**Task 5.30: Loadout selection screen (production version)**

Upgrade the Phase 3 loadout screen to production quality:
- Full visual design matching the game's UI aesthetic
- Weapon models previewed in 3D
- Add-on visual indicators
- Loadout saving (persist to local file)
- Smooth animations for selection and confirmation

**Task 5.31: Score screen**

Create the post-match score screen:
- Per-player stats: kills, deaths, assists (if applicable), damage dealt, accuracy
- Per-team stats (for team modes)
- Match result (win/loss, final score)
- MVP highlight
- Return to lobby button
- Play again button (re-queue for matchmaking)

**Task 5.32: Pause menu**

Create an in-game pause menu:
- Resume
- Settings (access settings mid-game)
- Quit to menu (with confirmation)
- Accessible via Escape key
- Pauses the game in local play; does not pause in online multiplayer

**Estimated effort:** 15–20 hours

---

### 3.8 Seasonal Framework

**Task 5.33: Season data structure**

Define the seasonal content framework:
- A "season" is a time-bounded content update (e.g., 8–12 weeks)
- Each season can introduce: new maps, new weapons, new add-ons, balance changes, cosmetic rewards
- Season metadata: season number, start date, end date, content list

**Task 5.34: Season content loading**

Implement the system to load season-specific content:
- Content is organized by season ID
- The game loads the current season's content on startup
- Previous seasons' content can be retained or rotated out
- This is a data-driven system — adding a new season means adding data files, not code

**Task 5.35: Battle pass or progression (basic)**

Implement a basic progression system:
- Players earn experience from matches (kills, wins, objectives)
- A progression track unlocks rewards at level thresholds
- Rewards are cosmetic (titles, color variants, model accessories)
- The system is simple and extensible for future expansion
- Progression data is stored locally for the beta (server-side progression is post-launch)

**Estimated effort:** 8–12 hours

---

## 4. Task Summary

| Task Group | Tasks | Estimated Hours |
|-----------|-------|----------------|
| Maps | 5.1, 5.2, 5.3, 5.4, 5.5 | 25–35 |
| Weapon Roster | 5.6, 5.7, 5.8 | 15–20 |
| Add-On Roster | 5.9, 5.10, 5.11 | 10–14 |
| Game Mode Rules | 5.12, 5.13, 5.14, 5.15, 5.16 | 15–20 |
| HUD | 5.17, 5.18, 5.19, 5.20, 5.21, 5.22 | 15–20 |
| Sound Design | 5.23, 5.24, 5.25, 5.26, 5.27 | 12–16 |
| Menu System | 5.28, 5.29, 5.30, 5.31, 5.32 | 15–20 |
| Seasonal Framework | 5.33, 5.34, 5.35 | 8–12 |
| **Total** | **35 tasks** | **115–157 hours** |

At 12–16 productive hours per week, this maps to the 8–10 week estimate. This is the most content-intensive phase. Parallelization across contributors is essential.

---

## 5. Parallelization Guide

This phase benefits heavily from parallel workstreams:

- **Map 1, Map 2, and Map 3** can be built simultaneously by different contributors. Each follows the same gray-box → playtest → art pass pipeline independently.
- **Weapon roster and add-on roster** can be implemented by one contributor while **game mode rules** are implemented by another.
- **Sound design** is entirely independent of code development and can be sourced or produced in parallel.
- **HUD and menu system** share the UI framework but can be split: HUD gameplay elements by one contributor, menu screens by another.
- **Seasonal framework** is a backend system that can be developed independently of all visual content.

With 3–4 active contributors, the phase can be compressed significantly from the linear estimate.

---

## 6. Acceptance Criteria

Phase 5 is complete when **all** of the following are true:

1. **Maps:** Three maps are playable with full art and lighting. Each map supports its intended game mode. All movement mechanics work across all maps. No map has exploitable geometry or physics bugs.

2. **Weapons:** The full weapon roster (6–8 weapons) is implemented and balanced. Each weapon has a distinct role. No weapon is useless or overpowered. TTK is consistent with design targets.

3. **Add-Ons:** The full add-on roster (10–15 add-ons) is implemented. Add-ons provide meaningful choices. No add-on combination creates a dominant meta-build.

4. **Game Modes:** All three game modes (deathmatch, team deathmatch, objective) are fully playable with scoring, win conditions, and match flow (pre-match → countdown → active → end → score screen).

5. **HUD:** All HUD elements are functional: health, ammo, crosshair, score, kill feed, damage indicators. The HUD is readable and does not obstruct gameplay.

6. **Sound:** All gameplay-critical sounds are present: weapon fire, impacts, footsteps, jumps, UI navigation. Sounds are spatially accurate for other players' actions.

7. **Menus:** Main menu, settings, loadout selection, score screen, and pause menu are all functional. The menu flow supports a complete session from launch to match to score screen and back.

8. **Seasonal Framework:** The seasonal content system is functional. A new season can be defined via data files. The basic progression system tracks experience and unlocks rewards.

9. **Complete Game Loop:** A player can launch the game, navigate menus, enter a match, play to completion, see results, and return to the menu. This works for both online and local play.

10. **Performance:** All content (maps, weapons, VFX, sounds) runs at target framerates. No content addition has degraded performance below Phase 4 targets.

---

## 7. Test Protocol

### 7.1 Full Session Test

1. Launch the game from a fresh install.
2. Navigate the main menu. Adjust settings.
3. Enter matchmaking (or start a local match).
4. Select a loadout with specific weapons and add-ons.
5. Play a complete match to the win condition.
6. View the score screen.
7. Return to the menu and start a second match with a different mode and map.
8. Verify the entire flow is seamless with no crashes, no soft-locks, and no missing UI.

### 7.2 Content Coverage Test

1. Play one full match on each map with each game mode.
2. Use every weapon in the roster during matches.
3. Equip and test every add-on.
4. Verify all sounds play correctly (weapon fire, footsteps, impacts, UI).
5. Verify all HUD elements display correctly during gameplay.
6. Verify all menu screens function correctly.

### 7.3 Balance Playtest

1. Conduct at least 10 hours of competitive playtesting across all modes.
2. Track kill/death ratios per weapon, per add-on loadout.
3. Identify any dominant strategies or weapons.
4. Adjust balance constants based on data.
5. Re-test after adjustments.

---

## 8. Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Map design requires extensive iteration | High | Medium | Gray-box playtesting catches layout problems early. Do not invest art time in layouts that have not been playtested. |
| Weapon balance takes longer than expected | Medium | Medium | Prioritize getting all weapons functional first. Balance can be tuned continuously. Use data from playtesting, not intuition. |
| Sound design is under-resourced | Medium | High | Use placeholder sounds from free libraries for the beta. Replace with custom sounds post-beta. Gameplay sounds (fire, impact) are non-negotiable. |
| Scope creep in game mode features | Medium | High | Define exact mode rules before implementation. Resist adding features during development. Document all decisions. |
| Menu system is more complex than anticipated | Low | Medium | Use Bevy's UI system straightforwardly. Avoid fancy animations or custom widgets for v0.5. Polish post-beta. |
| Seasonal framework is over-engineered | Medium | Low | Keep it minimal for v0.5. Data-driven content loading and a simple progression track. Everything else is post-launch. |

---

## 9. Exit Criteria — Beta Milestone

Upon completing Phase 5 and meeting all acceptance criteria:

1. Tag the repository as `v0.5`.
2. Build distributable binaries for all target platforms (Windows primary, Linux secondary).
3. Conduct a full regression test across all game modes, maps, weapons, and add-ons.
4. Distribute to the closed beta testing group.
5. Begin collecting feedback for the post-beta iteration cycle.

**The game is now in Beta.** All features specified in the roadmap are implemented. The focus shifts from development to testing, balance refinement, and community feedback integration.

---

## 10. Post-Beta Preview

After v0.5, the project enters a phase outside this roadmap. Anticipated activities include:

- **Closed beta testing:** 4–8 weeks of community testing with feedback collection
- **Balance iteration:** Data-driven balance adjustments based on beta play data
- **Bug fixing:** Triage and resolve issues reported by beta testers
- **Performance optimization:** Address framerate issues on specific hardware configurations
- **Content expansion:** Additional maps, weapons, and add-ons based on community demand
- **Anti-cheat integration:** Server-side validation hardening and client integrity checks
- **Infrastructure scaling:** Server capacity for open beta and launch
- **Launch preparation:** Marketing, distribution platform setup, v1.0 milestone definition

These will be planned in a dedicated post-beta document once v0.5 is achieved and the game's actual state is assessed against the original vision.
