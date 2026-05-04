# Audio Design

## Document Metadata

| Field | Value |
|---|---|
| Document ID | CCD-001 |
| Version | 1.0 |
| Status | Draft |
| Owner | Audio Lead |
| Last Updated | 2026-05-05 |

---

## 1. Audio Philosophy

Every sound in JuanSchott communicates specific game information. There are no decorative sounds. If a player can hear it, it means something. This principle governs all audio decisions from the earliest prototype through final release.

Audio in a competitive arena shooter serves as a parallel information channel to the visual system. Players under cognitive load during firefights cannot process every visual cue on screen. Sound fills that gap. A footstep behind you, a reload click from behind cover, the distinct activation tone of a specific add-on — each of these sounds carries tactical data that a skilled player can act on.

This philosophy creates two inviolable rules for audio design:

1. **Every sound must be identifiable.** If two different game events produce the same or similar sounds, one of them must be changed. Ambiguity in audio is a design defect.
2. **No sound exists without purpose.** Ambient beds, atmospheric loops, and environmental audio are permitted only when they serve the player's spatial awareness or immersion without masking critical gameplay sounds. Even ambient sounds must communicate something — where you are, what surface you are near, what part of the Vimana you are in.

Sounds that cannot be justified under these rules are cut. There is no audio padding.

---

## 2. Spatial Audio System

JuanSchott uses Bevy's built-in spatial audio for all 3D positional sound. Every gameplay-relevant sound — footsteps, weapon fire, ability activations, damage cues — is positioned accurately in 3D space relative to the listener (the player's camera).

### 2.1 Positional Accuracy

Sound sources emit from the world-space position of the originating entity. A footstep comes from the player's feet. A weapon's muzzle flash sound comes from the weapon's barrel position. An add-on activation sound originates from the player's center of mass. This positioning allows players to determine direction and approximate distance of events they cannot see.

### 2.2 Distance Falloff

All spatial sounds implement linear distance falloff. Each sound category has a defined audible range. Beyond that range, the sound is inaudible. This prevents information overload and ensures that audio cues remain meaningful — if you hear sprinting footsteps, the source is within 20 meters, guaranteed.

| Sound Category | Maximum Audible Range |
|---|---|
| Walk footstep | 8m |
| Sprint footstep | 20m |
| Slide | 15m |
| Wall-run | 12m |
| Weapon fire (pistol) | 40m |
| Weapon fire (rifle) | 50m |
| Weapon fire (shotgun) | 25m |
| Weapon reload | 12m |
| Ability activation | 15-25m (varies by add-on) |
| Jump / Double-jump | 10m |
| Landing | 8m |

### 2.3 Directional Cues

Players receive directional audio information through standard stereo or surround sound setups. HRTF support is a future consideration but is not in scope for launch. The minimum requirement is accurate left/right panning and front/back differentiation. Vertical differentiation (above/below) is supported but acknowledged as less precise on stereo systems.

---

## 3. Sound Categories

### 3.1 Movement Sounds

Movement is the most frequent source of audio events in any match. The movement sound system must be rich enough to communicate what a player is doing, where they are, and how fast they are moving, without becoming noise.

#### Footsteps

Footsteps are the primary movement audio. They communicate player position, speed, and current surface material.

- **Walk footstep**: Triggered every 0.45 seconds during walking movement. Surface-dependent sound variation (see Section 6). Audible range: 8 meters. Moderate volume.
- **Sprint footstep**: Triggered every 0.28 seconds during sprinting. Louder than walk footsteps by approximately 6dB. Audible range: 20 meters. Sprinting is the loudest sustained movement state and the primary way enemies detect approaching players.
- **Surface variation**: Different surface materials on the Vimana produce different footstep timbres. The current surface material is determined by a raycast downward from the player's position. Surface types include: metal grating, solid metal panel, composite flooring, rubberized surface, and zero-g padding. Each surface has a unique footstep sound set (3-4 variations per surface to avoid repetition fatigue).

#### Jump and Landing

- **Jump**: Short upward whoosh. Audible range: 8m. Communicates vertical movement initiation.
- **Double-jump**: Energy boost sound. Distinct from regular jump — includes a synthetic "charge" element that communicates the expenditure of a limited resource. Audible range: 10m.
- **Landing**: Impact sound scaled by fall distance. A short drop produces a quiet tap; a maximum-height drop produces a heavy impact. Audible range scales from 4m (short drop) to 8m (maximum drop).

#### Slide

The slide produces a loud, sustained scraping/friction sound. Duration matches the slide duration (up to 1.2 seconds). Louder than sprinting. Audible range: 15 meters. The slide is a deliberate trade-off — you gain speed and a lower profile but announce your position to nearby enemies.

#### Wall-Run

Wall-running produces a continuous surface-scraping sound. Softer than sliding but sustained. Audible range: 12 meters. The wall-run sound includes a subtle rhythm change as the duration timer approaches expiry, giving the wall-running player (if attentive) an audio cue that the wall-run is about to end.

#### Wall Jump

A sharp lateral push-off sound. Shorter and punchier than a regular jump. Communicates a direction change. Audible range: 8m.

### 3.2 Weapon Sounds

Every weapon type has a unique and immediately identifiable firing sound. This is critical for competitive play — a player must be able to identify what weapon an opponent is using from audio alone.

#### Firing Sounds

- **Pistol**: Sharp, snappy crack. Short decay. Designed to sound precise and controlled.
- **Rifle**: Sustained burst or rapid individual shots (depending on fire mode). Fuller, more aggressive than the pistol. The rapid-fire variant has a distinctive rhythmic quality.
- **Shotgun**: Heavy, concussive blast. Widest frequency spread of any weapon. Unmistakable.
- **Beam**: Continuous energy hum with an overlay "searing" tone. The beam sound persists as long as the trigger is held. Has a faint doppler-like waver as the beam sweeps across targets.
- **Projectile launcher**: Deep thump on firing, followed by the projectile's travel sound (a low hum or whistle depending on projectile type).

#### Reload

Each weapon has a distinct reload sound sequence. Reload sounds are shorter than the actual reload time — the sound plays at the start of the reload, not throughout. This communicates to nearby enemies that the reloading player is vulnerable.

- **Magazine ejection**: Short mechanical click.
- **New magazine insertion**: Slightly louder click-snap.
- **Chamber/slide action**: Distinct per weapon type.

Audible range: 12 meters for the full reload sequence.

#### Empty Magazine

A distinct, dry "click" when the trigger is pulled on an empty magazine. This sound is intentionally different from the reload sound to communicate that the player attempted to fire but could not. Useful for opponents who hear it — the target is completely out of ammo, not just reloading.

#### Impact Sounds

- **Impact on player body**: A distinct "hit confirmation" sound heard by the shooter. Different from surface impacts. This is the primary feedback mechanism for confirming hits, especially at range where visual hit markers may be missed.
- **Impact on surface/environment**: Varied by surface material. Provides spatial awareness — you can hear where missed shots are landing.

### 3.3 Damage and Health Sounds

#### Taking Damage

When a player takes damage, they hear a directional audio cue. The cue is panned to indicate the direction the damage came from. This is critical for survival — players must be able to react to threats they cannot see. The damage cue is a sharp, synthetic "sting" that cuts through other audio.

The damage cue scales in intensity with the amount of damage taken. A small chip (10 HP) produces a subtle cue. A large hit (50+ HP) produces a much more prominent cue. A headshot produces the most intense damage cue, combined with a brief high-frequency ring.

#### Health Regeneration

- **Regen start**: A soft, rising tone that communicates "healing has begun." Played once when the 3-second regen delay expires and regeneration starts.
- **Regen in progress**: A faint, sustained hum during active regeneration. Quiet enough not to mask other sounds.
- **Regen complete**: A short, satisfying "full" tone when HP reaches maximum.

#### Death

A brief, distinctive "shutdown" sound. Not dramatic or lengthy — the player needs to get back into the match. The death sound communicates finality without being punishing.

#### Respawn

A "reinitialization" sound when the player respawns. Conveys the sci-fi concept of materialization. Distinct from all other sounds so players nearby can hear that someone has spawned.

### 3.4 Ability Sounds (Add-Ons)

Each add-on has a unique activation sound, deactivation sound, and cooldown-ready ping.

- **Activation sound**: Played when the add-on is activated. Must be identifiable per add-on type. Communicates to nearby enemies that an ability is in use. Audible range varies by add-on (15-25m).
- **Deactivation sound**: Played when the add-on's active duration expires or the player deactivates it manually. Shorter and quieter than activation.
- **Cooldown ready ping**: Heard only by the player who owns the add-on. A short, clear "ready" tone. This is the audio equivalent of a UI indicator — the player should not need to look at their HUD to know their add-on is available.

### 3.5 UI Sounds

UI sounds are non-spatial. They play at a consistent volume regardless of game state. They exist only in menus or as overlay notifications during gameplay.

- **Menu interactions**: Open, close, navigate between menu items.
- **Button clicks**: Confirmation sound for menu selections.
- **Score updates**: Played when the score changes. Different tones for your team scoring vs. the enemy team scoring.
- **Kill feed notifications**: A brief sound when you eliminate an opponent. Distinct from score updates.
- **Match timer warnings**: Audio cues at 60 seconds remaining and 10 seconds remaining in a round.

### 3.6 Ambient Sounds

Ambient audio serves spatial awareness, not mood.

- **Map ambience**: Very quiet, atmospheric loops unique to each map section of the Vimana. These loops are subtle enough that they never mask gameplay sounds. They communicate "where am I" — different areas of the map have different ambient tones.
- **Vimana ambient hum**: A persistent, very low-frequency hum present on all maps. Represents the Vimana's operating systems. Nearly subliminal at normal volume. Provides a sense of place — you are inside a massive machine.

---

## 4. Sound Design Aesthetic

The audio aesthetic for JuanSchott is **clean, synthetic, and slightly alien**. No realistic gun recordings. No foley-captured footsteps. All sounds are designed — synthesized, processed, and sculpted to fit the science fiction setting of the Vimana.

This aesthetic serves the functional philosophy: designed sounds are easier to make distinct from one another than realistic sounds. When every sound is crafted from scratch, there is no risk of two weapons sounding similar because they were both sourced from similar real-world recordings.

Key characteristics of the JuanSchott sound palette:

- **Clean transients**: Attack portions of sounds are sharp and well-defined. This aids identification and cuts through the mix.
- **Controlled decay**: No long reverb tails except where the map environment specifically warrants it. Most sounds decay quickly to make room for the next sound.
- **Synthetic textures**: Metallic resonances, energy-based tones, processed mechanical sounds. The Vimana is a machine — everything sounds like it belongs in or on a machine.
- **Frequency separation**: Each sound category occupies a defined frequency range. Weapon sounds live in the midrange. Footsteps in the low-mid. Damage cues in the high-mid. This separation ensures sounds do not mask each other.

---

## 5. Audio Mixing Priorities

When multiple sounds compete for the player's attention, they are mixed according to a strict priority hierarchy:

| Priority | Category | Rule |
|---|---|---|
| 1 (Highest) | Damage taken | Always audible. Duck all other sounds briefly. |
| 2 | Weapon fire (own) | Always audible. Player must hear their own gun. |
| 3 | Weapon fire (enemy) | High priority for spatial awareness. |
| 4 | Ability sounds | Important tactical information. |
| 5 | Footsteps | Important but lower priority than combat. |
| 6 | Reload and weapon actions | Useful but situational. |
| 7 (Lowest) | Ambient | Fades or ducks when higher-priority sounds play. |

Critical game information is always audible. If a player is taking damage, that sound ducks everything else momentarily. If a player is firing their weapon, ambient sounds reduce in volume. The mix dynamically adjusts so the most important information is never masked.

### Ducking Rules

- Damage cue: ducks all other sounds by 6dB for 0.15 seconds.
- Own weapon fire: ducks ambient by 8dB, footsteps by 3dB during firing.
- Enemy weapon fire nearby: ducks ambient by 4dB.

---

## 6. Footstep Surface System

The Vimana's interior surfaces are categorized into material types. Each material produces a distinct footstep timbre. The material is determined by a downward raycast from the player entity.

| Surface Material | Sound Character | Maps/Regions |
|---|---|---|
| Metal Grating | Hollow, resonant, slight rattle | Catwalks, maintenance sections |
| Solid Metal Panel | Flat, solid, slightly muted | Primary walkways, platforms |
| Composite Flooring | Dense, slightly organic undertone | Hub areas, transition zones |
| Rubberized Surface | Soft, muted, low-pitch | Training areas, respawn zones |
| Zero-G Padding | Nearly silent, soft pad | Zero-gravity sections |

Each surface has 3-4 footstep variations that play randomly to avoid repetitive "machine gun" footsteps. The variation set is the same for walk and sprint; only timing, volume, and range change.

Sprint footsteps are louder than walk footsteps by approximately 6dB. This is a deliberate gameplay balance mechanism: sprinting gets you there faster but tells everyone nearby where you are.

---

## 7. Music Policy

Music is **not in scope for launch**. JuanSchott is a competitive arena shooter. During gameplay, there is no music. The audio channel belongs entirely to gameplay information.

Future considerations for music, post-launch:

- **Main menu music**: Themed per season. Atmospheric, not intrusive.
- **Season event stingers**: Short musical motifs for season transitions or special events.
- **Victory/defeat themes**: Brief musical cues at match end. Under 5 seconds.

Music will never play during active gameplay. The competitive focus of JuanSchott requires that the audio channel remain dedicated to game information at all times during a match.

---

## 8. Technical Implementation Notes

### Audio Engine

All audio is implemented through Bevy's built-in audio system. Bevy uses `rodio` as the audio backend, which supports WAV, Ogg Vorbis, and FLAC formats. All JuanSchott audio assets are stored as Ogg Vorbis for quality-to-size ratio.

### Sound Asset Organization

```
assets/audio/
  footsteps/
    metal_grating/     (walk_01.ogg through walk_04.ogg, sprint_01.ogg through sprint_04.ogg)
    solid_metal/       (same structure)
    composite/         (same structure)
    rubber/            (same structure)
    zerog_pad/         (same structure)
  weapons/
    pistol/            (fire.ogg, reload.ogg, empty.ogg, impact_body.ogg, impact_surface.ogg)
    rifle/             (same structure)
    shotgun/           (same structure)
    beam/              (activate.ogg, sustain_loop.ogg, deactivate.ogg, impact_body.ogg)
    projectile/        (fire.ogg, travel_hum.ogg, impact_body.ogg, impact_surface.ogg)
  damage/
    hit_light.ogg
    hit_medium.ogg
    hit_heavy.ogg
    hit_headshot.ogg
    regen_start.ogg
    regen_loop.ogg
    regen_complete.ogg
    death.ogg
    respawn.ogg
  abilities/
    [per_addon]/       (activate.ogg, deactivate.ogg, cooldown_ready.ogg)
  ui/
    menu_open.ogg
    menu_close.ogg
    button_click.ogg
    score_friendly.ogg
    score_enemy.ogg
    kill_confirm.ogg
    timer_60s.ogg
    timer_10s.ogg
  ambient/
    vimana_hum.ogg
    [per_map_region]/  (ambient_loop.ogg)
```

### Performance Budgets

- Maximum simultaneous spatial audio sources: 32.
- Maximum simultaneous ambient sources: 4.
- All sounds are mono for spatial playback except ambient loops (stereo).
- UI sounds are mono, played without spatial positioning.
- Audio memory budget: 64MB loaded at any time.

---

## 9. Open Questions

| ID | Question | Status |
|---|---|---|
| AQ-1 | Should footstep audible ranges differ per map region? | Deferred to playtesting |
| AQ-2 | HRTF support timeline? | Post-launch |
| AQ-3 | Should damage direction cues be more prominent than current spec? | Pending playtest feedback |
| AQ-4 | Ambient sound ducking thresholds — are current values correct? | Pending playtest feedback |
| AQ-5 | Should the beam weapon's sustain loop be audible to the target player specifically? | Under discussion |
