# UI and HUD Design

**Document ID:** ASTRA-ART-003
**Category:** Art & Visual Design
**Status:** Draft
**Last Updated:** 2026-05-05

---

## Table of Contents

1. [Design Philosophy](#1-design-philosophy)
2. [HUD Elements (During Gameplay)](#2-hud-elements-during-gameplay)
3. [Menu Systems](#3-menu-systems)
4. [UI Visual Style Guide](#4-ui-visual-style-guide)
5. [Technical Specifications](#5-technical-specifications)
6. [Accessibility](#6-accessibility)
7. [See Also](#7-see-also)

---

## 1. Design Philosophy

The UI exists to serve the player, never to impress them. Every pixel on screen must justify its presence by communicating something the player needs to know right now. If it does not, it is removed.

The following principles govern all UI and HUD decisions across the entire project. No exception is made for menus, in-game overlays, or spectator interfaces.

### Core Principles

**P1 — Minimal HUD.** Show only what is needed in the current moment. Hide everything else. The default state of the HUD is invisible. Elements appear when their information becomes relevant and disappear when it ceases to be.

**P2 — Zero visual clutter.** This aligns with Pillar 4 (Clean and Readable) as defined in the art direction document. The player's attention belongs on the game world and the opponent, not on chrome, ornaments, or decorative UI framing. If an element can be 1px thinner without losing legibility, make it thinner.

**P3 — 0.5-second read time.** Any individual HUD element must be fully comprehensible within half a second of a glance. This means no dense text blocks, no multi-step visual parsing, and no ambiguous iconography. If a designer cannot read it in 0.5 seconds during playtesting, it is redesigned.

**P4 — Consistent visual language.** All UI — menus, HUD, popups, tooltips, error messages — uses the same font family, the same color definitions, the same icon style, and the same spacing system. A player who has learned to read the ammo counter should instantly understand how to read the health bar. No element should feel like it belongs to a different application.

**P5 — The game world is the primary interface.** UI overlays must never fully occlude gameplay. All HUD elements use semi-transparent backgrounds. All menus either dim the game world or allow it to remain partially visible. The player should never feel disconnected from the match.

---

## 2. HUD Elements (During Gameplay)

All positional references assume a standard 16:9 display. Elements should anchor to their designated corner and scale proportionally on other aspect ratios. All HUD elements respect a safe zone margin of 5% from all screen edges.

### [UI-001] Crosshair

**Position:** Dead center of the screen viewport.

**Structure:** Four short lines arranged in a cross pattern with a central gap. No center dot. The gap in the center ensures the player's view of the target is never obstructed.

| Property        | Value                                  |
|-----------------|----------------------------------------|
| Line length     | 16px each                              |
| Central gap     | 4px                                    |
| Line thickness  | 1.5px                                  |
| Default color   | White (#FFFFFF) at 80% opacity         |
| Enemy hover     | Red (#FF3333)                          |
| Outline         | 0.5px dark outline for contrast on bright surfaces |

**Dynamic Behavior:**

- **Movement expansion:** Lines spread outward proportionally to player velocity. At maximum movement speed, the gap effectively doubles. This communicates accuracy penalty without requiring the player to look at a separate indicator.
- **Spread indicator:** Weapon spread stat directly maps to crosshair spread. Shotguns produce wide spread; precision weapons produce near-static crosshairs.
- **Hit confirmation:** On a successful hit, all four lines briefly flash to increased brightness (100% opacity, slight scale pulse). Duration: 100ms.
- **Headshot confirmation:** Distinct gold/amber flash (#FFD700) on headshot. Duration: 150ms. The different color and longer duration ensure the player can distinguish headshots from body shots without any ambiguity.

**Design Rationale:** The crosshair is the single most-looked-at UI element in a competitive shooter. It must communicate weapon state passively — the player should feel accuracy changes rather than having to consciously read them.

---

### [UI-002] Health Indicator

**Position:** Bottom-left corner, inset from safe zone.

**Layout:** A large numeric readout (e.g., "100") positioned above a thin horizontal bar. The number is the primary readout; the bar is a secondary visual cue for peripheral vision.

| Health Range  | Number Color | Bar Color   |
|---------------|-------------|-------------|
| 51–100        | White       | White       |
| 26–50         | Yellow      | Yellow      |
| 1–25          | Red         | Red         |
| 0             | Hidden      | Hidden      |

**Dynamic Behavior:**

- **Damage:** Instant number change. No animation, no tween, no fade. The number snaps to the new value immediately. This is a deliberate design choice — animated health drains introduce cognitive latency. The player needs to know their exact health now, not watch it descend.
- **Regeneration:** When health is regenerating, the number pulses subtly (opacity oscillation between 90% and 100%) at a 1.5-second cycle. The bar fills smoothly during regeneration. This creates a clear visual distinction between "taking damage" (instant, jarring) and "healing" (smooth, rhythmic).
- **Max health override:** If a player's max health is modified by an add-on, the display shows "[current] / [max]" format (e.g., "85 / 120"). Otherwise, only the current number is shown.

**Design Rationale:** Health is survival information. It must be instant, unambiguous, and readable from peripheral vision without the player looking directly at it.

---

### [UI-003] Ammo Counter

**Position:** Bottom-right corner, inset from safe zone. Mirrors the health indicator for visual balance.

**Format:** Varies by weapon type.
- **Hitscan / Projectile:** "[current] / [magazine]" — e.g., "12 / 30"
- **Beam (energy):** "[current energy] / [max energy]" — e.g., "156 / 200"
- **Melee:** No ammo display. The counter is hidden entirely.

| Ammo State               | Color  |
|--------------------------|--------|
| Above 25% of magazine    | White  |
| Below 25% of magazine    | Yellow |
| Empty (0)                | Red    |

**Reload Indicator:** During a reload, a thin circular progress arc sweeps around the ammo number, completing a full 360° circle when the reload finishes. This communicates reload progress without requiring a separate progress bar.

**Low ammo warning:** When ammo drops below 25%, the number subtly pulses at the same cadence as the low-health indicator. This creates a consistent "warning pulse" language across the HUD.

**Design Rationale:** Ammo is the second most critical survival stat. The mirrored positioning (bottom-left = health, bottom-right = ammo) creates a predictable spatial layout the player can internalize through muscle memory.

---

### [UI-004] Weapon Display

**Position:** Bottom-center, between health and ammo indicators.

**Layout:** Two weapon silhouette icons arranged horizontally: [Primary] [Secondary]. The currently equipped weapon is highlighted (full opacity, subtle glow outline). The unequipped weapon is dimmed (40% opacity).

**Swap Animation:** On weapon switch, the outgoing weapon icon shrinks to 0% and the incoming weapon icon scales from 0% to 100%. Duration: 200ms. Easing: ease-out. This is fast enough to not feel sluggish but visible enough to confirm the swap.

**No text labels.** Weapon names are not shown during gameplay. The silhouette icons are distinct enough to be identified at a glance. If a player needs to read the name of their weapon during a firefight, the icons have failed.

---

### [UI-005] Kill Feed

**Position:** Top-right corner, stacked vertically. Newest entry at the top.

**Format:** `[Player1] ▸ [Player2]` with a small weapon icon between the names.

| Element            | Style                                                      |
|--------------------|------------------------------------------------------------|
| Player names       | Color-coded by team allegiance                              |
| Kill arrow (▸)     | White, small                                                |
| Weapon icon        | 16px silhouette of the weapon used                          |
| Assist marker      | `[Player1] + [Player3] ▸ [Player2]` — plus sign between assisters |

**Behavior:**

- Maximum of 5 entries visible simultaneously.
- Each entry persists for 4 seconds, then fades out over 0.5 seconds.
- Oldest entries are pushed downward and off-screen by newer entries.
- The player's own kills are highlighted with a subtle background tint for immediate identification.

**Design Rationale:** The kill feed provides situational awareness — who is fighting whom, which weapons are in play, which players are alive. It is reference information, not critical information, so it is placed in the periphery and designed to be read passively.

---

### [UI-006] Minimap

**Position:** Top-left corner. Toggle-able via a keybind (default: M). Can be set to always-on in settings.

**Rendering:** Top-down schematic view of the current map. Walls are drawn as solid lines. Open spaces are empty. No textures, no gradients, no terrain detail. The minimap is a diagram, not a painting.

**Displayed Information:**

| Element                    | Representation                                       |
|----------------------------|------------------------------------------------------|
| Player position            | White directional arrow (points in facing direction) |
| Teammate positions         | Blue dots, labeled with player name on hover         |
| Last known enemy positions | Red dots that fade to transparent over 3 seconds     |
| Map boundaries             | Thin white outline                                   |

**Critical Design Rule — No Real-Time Enemy Tracking.** Enemies are never shown at their current position. Red dots appear only at the last known position where a teammate had line-of-sight to an enemy, and they fade after 3 seconds. The minimap provides inferred information, not omniscient surveillance. This is non-negotiable for competitive integrity.

**Size:** 15% of screen width, square aspect ratio. Background at 40% opacity. The player arrow is centered in the minimap; the map rotates around the player (player-centric rotation, not north-up).

---

### [UI-007] Score Display

**Position:** Top-center, below the safe zone margin.

**Format:** `[Team A Score] — [Time] — [Team B Score]`

- Time counts down from the match time limit (e.g., "12:45").
- Below the main line: objective text — "First to 30" or "First to 75" depending on game mode.
- When a team reaches match point (one kill away from winning), the score flashes briefly.

**Frag-based emphasis:** When a kill occurs that changes the score, the scoring team's number pulses once (scale 100% → 110% → 100% over 300ms). This provides feedback without being distracting.

---

### [UI-008] Damage Direction Indicator

**Position:** Screen edges — the indicator appears on the edge closest to the source of incoming damage.

**Visual:** A red arc or chevron shape rendered at the screen edge. The arc's position on the edge corresponds to the vertical angle of the damage source (left/right is mapped to the edge; up/down adjusts position along that edge).

| Property         | Value                              |
|------------------|------------------------------------|
| Duration         | 0.5 seconds                        |
| Fade             | Linear fade to transparent         |
| Intensity        | Proportional to damage taken (heavier hits = brighter, larger arc) |
| Color            | Red (#FF3333)                      |
| Maximum arcs     | 3 simultaneous (from different directions) |

**Design Rationale:** This is the player's primary spatial awareness tool for incoming threats. It must be fast, directional, and proportional. The 0.5-second duration is long enough to register consciously but short enough to not clutter the screen during sustained firefights.

---

### [UI-009] Add-On Status

**Position:** Above the health bar, in the bottom-left area.

**Layout:** Three small icons (32px each) in a horizontal row. Each icon represents one equipped add-on slot.

| State     | Visual Representation                                        |
|-----------|--------------------------------------------------------------|
| Passive   | Icon at 60% opacity, subtly lit. Always visible.             |
| Active    | Icon glows (brightness pulse to 100% opacity, soft outline). |
| Cooldown  | Circular sweep overlay darkens the icon proportionally to remaining cooldown time (similar to reload indicator). Sweeps clockwise from top. |

**Design Rationale:** Add-ons are secondary mechanics. Their status must be checkable without commanding attention. The passive/active/cooldown states use the same visual language established by the ammo reload indicator (circular sweep) and health regen (brightness pulse), reinforcing P4 (consistent visual language).

---

### [UI-010] Ability Ready Indicator

For add-ons with active abilities: when the ability comes off cooldown and is ready to use, the corresponding icon in [UI-009] pulses twice in rapid succession (200ms per pulse), then settles into the active glow state. This is a one-time notification — it does not repeat until the ability is used and comes off cooldown again.

This pulse serves the same function as a sound cue: it draws the eye to inform the player that a new option is available, then gets out of the way.

---

## 3. Menu Systems

All menus share the following structural conventions:
- Semi-transparent dark overlay (70% opacity black) over the game world or menu background.
- All text uses the project's standard UI font (see Section 4).
- Navigation uses both mouse click and keyboard (arrow keys + Enter).
- ESC always returns to the previous screen or closes the current overlay.

### Main Menu

The main menu is the player's first impression after launch. It sets the tone. It does not need to excite — it needs to not annoy.

**Layout:**
- Game title: top-center, large, static. No animation, no glow, no particle effects.
- Menu options: center of screen, stacked vertically. Options: **Play**, **Loadout**, **Codex**, **Settings**, **Quit**.
- Selected option: text brightens and a thin horizontal line appears beneath it. No bouncing, no scaling, no sound effects on hover.
- Background: slow-panning view of the Vimana interior. This doubles as environment art showcase — let the world sell itself.

**Deliberate omissions:** No animated characters, no particle effects, no 3D rotating weapons, no dynamic lighting. These compete for attention and increase load times without serving the player.

---

### [UI-011] Loadout Screen

**Layout:** Three-panel design.

| Panel  | Position | Content                                                                                 |
|--------|----------|-----------------------------------------------------------------------------------------|
| Left   | 30% width | Weapon selection. Primary slot and secondary slot. Click a slot, then click a weapon to assign it. Selecting a weapon reveals its stats below (damage, fire rate, range, special properties). |
| Center | 40% width | Player model preview. Shows the currently equipped weapons and add-ons on the character. Rotates slowly. Player can manually rotate by dragging. |
| Right  | 30% width | Add-on selection. Three slots. Click a slot, then click or drag an add-on to fill it. Shows add-on description and stats on selection. |

**Bottom bar:** Three loadout presets (Preset A, Preset B, Preset C). Click to load. Auto-save on any change. Presets are stored locally.

**Validation:** The loadout screen does not allow entering matchmaking with empty slots. If a required slot is unfilled, the "Play" button is disabled and the empty slot pulses to draw attention.

---

### [UI-012] Settings

Standard tabbed settings interface. Tabs across the top: **Video**, **Audio**, **Controls**, **Gameplay**, **Accessibility**.

**Video:**
- Resolution (dropdown, all supported resolutions)
- Display mode: Fullscreen / Borderless Windowed / Windowed
- Quality preset: Low / Medium / High / Ultra
- Individual toggles: shadows, anti-aliasing, motion blur, vsync
- FOV slider: 70°–120°, default 90°

**Audio:**
- Master volume (0–100%)
- Music volume (0–100%)
- SFX volume (0–100%)
- Voice (dialogue) volume (0–100%)
- Voice chat (microphone) volume: separate slider
- Voice chat output volume: separate slider

**Controls:**
- Full key rebinding for all actions. Click an action, press the desired key. Supports multi-key bindings.
- Mouse sensitivity slider (0.1–10.0, default 1.0)
- Mouse sensitivity Y-axis multiplier (0.5–2.0, default 1.0)
- Controller support: automatic detection. Separate sensitivity sliders for controller. Full button remapping.

**Gameplay:**
- Crosshair customization: color picker, size slider, style selector (cross / dot / circle / cross-dot)
- Damage numbers: on / off
- HUD opacity: 20%–100% slider
- Minimap: always-on / toggle / off
- Kill feed: on / off

**Accessibility:**
- Colorblind modes: Protanopia / Deuteranopia / Tritanopia / Off. Adjusts all color-dependent HUD elements (health, ammo, team colors, damage indicators) to use distinguishable alternatives.
- Subtitle options: on / off, font size (small / medium / large), background opacity
- HUD text size: small / medium / large
- Screen shake: 0%–100% (reduces or eliminates camera shake effects)
- Reduced motion: disables non-essential animations (crosshair pulse, kill feed transitions, menu transitions)

---

### Score Screen (End of Match)

Displayed after a match concludes. Shows the results before the player queues again.

**Content:**
- Match outcome header: "VICTORY" or "DEFEAT" in team color.
- Player table: columns for Player Name, Kills, Deaths, Assists, Damage Dealt. Sortable by clicking column headers.
- MVP highlight: the top-performing player (by a composite score of kills, assists, and damage) is called out with a subtle highlight row.
- "Continue" button: active only after a 5-second delay. This prevents instant-queue behavior and ensures the player actually sees their results. The button text shows a countdown during the delay (e.g., "Continue (3...)").

---

## 4. UI Visual Style Guide

All UI elements adhere to the following visual specifications. No deviations without art lead approval.

### Typography
- **Font family:** Clean sans-serif. System font stack or open-source alternative (e.g., Inter, IBM Plex Sans). No decorative or condensed variants.
- **Weight hierarchy:** Regular for body text, Medium for labels, Bold for numbers (health, ammo, score).
- **Sizes:** Base size 14px. HUD numbers scale to 24–32px. Menu text at 16–18px. Headers at 24px.

### Color
- All UI colors are drawn from the project's color language as defined in `visual_language.md`.
- Backgrounds: dark semi-transparent (#000000 at 40–70% opacity). Never fully opaque.
- Text: white (#FFFFFF) primary, gray (#AAAAAA) secondary.
- Accent colors: team colors, state colors (red/yellow/white for health/ammo), confirmation colors (green for positive, gold for headshot).

### Shape Language
- No rounded corners. All UI elements use angular/geometric shapes. This matches the Vimana interior aesthetic — angular architecture, sharp edges, engineered precision.
- Button corners: 0px radius (sharp rectangles). Panels: 0px radius. Progress bars: 0px radius.
- This is a deliberate aesthetic choice. Rounded corners feel approachable and consumer-friendly. Sharp corners feel purposeful and industrial. The Vimana is a weapon, not a living room.

### Iconography
- Style: simple line icons (strokes, not fills). 1.5px stroke width.
- Level of detail: minimal. An icon must be recognizable at 24px. If it requires more detail than that, it should be redesigned.
- Consistency: all icons use the same stroke weight, the same corner style (sharp), and the same visual density.

### Transitions and Animation
- Standard transition duration: 200ms. Easing: ease-out.
- No bouncing, no overshoot, no spring physics. Animations should feel mechanical and precise, matching the game's aesthetic.
- Element fade-in/fade-out: 150ms.

---

## 5. Technical Specifications

### Safe Zones and Anchoring

All HUD elements anchor to screen corners with a minimum 5% inset from all edges. On ultra-wide displays (21:9 and above), the HUD does not stretch to fill the full width — it maintains a 16:9 proportional layout centered on the display.

### Resolution Scaling

HUD elements use viewport-relative sizing. At 1080p, the base unit is 1px. At 4K, the base unit scales to 2px. All sizes defined in this document (e.g., "16px crosshair lines") refer to the 1080p baseline.

### Performance Budget

The HUD rendering budget is 0.5ms of frame time at 60fps. HUD elements must not be the cause of frame drops. If the HUD budget is exceeded, the first optimization step is to reduce animation complexity, not to reduce information density.

---

## 6. Accessibility

Accessibility is not a post-launch feature. The following are baseline requirements, not stretch goals.

- **Colorblind support** (defined in Settings) must be verified using color blindness simulation tools during QA.
- **All HUD information must be conveyed through at least two channels** (e.g., color AND position, icon AND text). No information relies on color alone.
- **Text scaling** must not break layout. All UI containers must accommodate the "Large" text size setting without overlapping or clipping.
- **Keyboard navigation** must reach every interactive element in every menu. Tab order follows logical reading order.
- **Screen reader compatibility** for menu text is required. HUD elements are exempt during active gameplay (they change too rapidly for screen reader output to be useful), but menu screens must expose text to accessibility APIs.

---

## 7. See Also

| Document                                        | Relationship                                                     |
|-------------------------------------------------|------------------------------------------------------------------|
| `art_direction.md`                              | Parent art direction document. Pillar 4 (Clean and Readable) directly governs this document. |
| `visual_language.md`                            | Color palette, material definitions, and visual tone. All UI colors reference this document. |
| `../02_game_design/health_and_damage.md`        | Health and damage mechanics. UI-002 and UI-008 display outputs of these systems. |
| `../02_game_design/addon_system.md`             | Add-on mechanics. UI-009 and UI-010 display add-on states defined by this system. |
