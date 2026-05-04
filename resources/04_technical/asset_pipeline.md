# Asset Pipeline — JuanSchott

**Document Version:** 1.0  
**Last Updated:** 2026-05-05  
**Status:** Active  

---

## 1. Overview

JuanSchott's asset pipeline is a linear workflow: assets are authored in Blender, exported as glTF 2.0 files (binary `.glb`), and loaded directly by Bevy at runtime. There are no intermediate baking steps, no texture compression pipelines, and no custom asset processors. The pipeline is designed for speed of iteration — artists can modify a Blender file, export, and see the result in-game within seconds using Bevy's built-in asset hot-reloading.

The pipeline's simplicity is a direct consequence of the game's visual design: flat colors with no textures. Without texture assets, the pipeline has no need for DDS/KTX compression, mipmap generation, or texture packing tools. The only assets flowing through the pipeline are 3D meshes, skeletal animations, material color values, and collider definitions.

---

## 2. Pipeline Overview

```
Blender (.blend)  ──export──>  glTF 2.0 (.glb)  ──load──>  Bevy Runtime
     │                              │                            │
     │                              │                            ├─ Mesh + Material spawning
     │                              │                            ├─ Collider generation (Avian)
     │                              │                            └─ Animation clip loading
     │                              │
     │                              └─ Asset hot-reload watcher (development only)
     │
     └─ Source files in blender/ directory, tracked in Git (LFS)
```

The pipeline has two modes:

- **Development mode.** Bevy's asset server watches the `assets/` directory for changes. When an artist exports an updated `.glb` file, Bevy detects the filesystem change and hot-reloads the asset in the running game. No restart required.
- **Release mode.** Assets are packed into the game binary or distributed alongside it. No filesystem watching.

---

## 3. Blender Export Settings

All assets are exported from Blender using the built-in glTF 2.0 exporter. The export settings are standardized across all asset types to ensure consistency.

### 3.1 Export Configuration

| Setting | Value | Rationale |
|---|---|---|
| Format | Binary glTF (.glb) | Smallest file size, single-file output, fastest load time |
| Apply Modifiers | Yes | Ensures subsurf, mirror, and other modifiers are baked into the exported mesh |
| Export Selected Only | Yes | Prevents accidental export of helper objects, reference geometry, or unused collections |
| Cameras | No | Cameras are defined in Bevy, not in Blender |
| Lights | No | Lights are defined in Bevy, not in Blender |
| Materials | Yes | Export material color values for mapping in code |
| Animations | Yes | Export skeletal animations as glTF animation clips |
| Skins | Yes | Export armature/skinning data for animated models |
| Tangents | No | Not needed — no normal maps |
| Images | No | No textures are used |
| Compression | None | glb is already compact; compression adds load-time decompression overhead |

### 3.2 Export Procedure

1. Open the source `.blend` file.
2. Select the objects to export (the relevant collection or objects).
3. File → Export → glTF 2.0 (.glb).
4. Configure settings per the table above.
5. Export to the appropriate `assets/` subdirectory using the naming convention.

### 3.3 Why No Cameras or Lights in Export

Cameras and lights are configured entirely in Bevy code. The game's camera setup (main camera, view model camera, UI camera) has specific rendering requirements (render layers, FOV, post-processing) that cannot be expressed through glTF. Similarly, the game's lighting is tuned per-map in code to ensure consistency with the shadow mapping configuration. Exporting cameras and lights from Blender would create confusion — they would be imported as Bevy entities and conflict with the code-defined ones.

---

## 4. Naming Conventions

All objects in Blender and all exported files follow strict naming conventions. Consistent naming is critical for automated asset loading, collider generation, and spawn-point detection.

### 4.1 File Naming

Files are named using `snake_case` and are prefixed by asset type:

| Asset Type | Pattern | Example |
|---|---|---|
| Map | `map_[name].glb` | `map_arena_01.glb`, `map_bridge.glb` |
| Player Model | `model_player_[variant].glb` | `model_player_default.glb`, `model_player_heavy.glb` |
| Weapon Model | `weapon_[name].glb` | `weapon_rifle.glb`, `weapon_pistol.glb` |
| Projectile Model | `model_projectile_[name].glb` | `model_projectile_bullet.glb` |
| View Model | `model_viewmodel_[weapon].glb` | `model_viewmodel_rifle.glb` |
| Prop/Environment | `model_prop_[name].glb` | `model_prop_crate.glb`, `model_prop_pillar.glb` |

### 4.2 Object Naming Within Blender

Objects inside Blender files follow `snake_case` naming. This is important because Bevy's glTF loader preserves object names as entity names, which are used for programmatic lookup.

| Object Type | Pattern | Example |
|---|---|---|
| Mesh objects | `mesh_[description]` | `mesh_floor_main`, `mesh_wall_north` |
| Armatures | `arm_[description]` | `arm_player`, `arm_weapon_rifle` |
| Empty objects (spawn points) | `spawn_[team]_[index]` | `spawn_team_a_01`, `spawn_team_b_03` |
| Empty objects (misc markers) | `marker_[description]` | `marker_flag_capture`, `marker_powerup_spawn` |
| Collections | `[category]_[name]` | `geometry_walls`, `geometry_floors` |

### 4.3 Spawn Point Markers

Spawn points are defined in Blender as empty objects with specific names. The game code scans the loaded glTF scene for entities whose names match the pattern `spawn_team_[a|b]_[0-9]+`. Each matching entity's transform becomes a spawn point in the game.

For free-for-all modes, spawn points use the pattern `spawn_ffa_[0-9]+`.

---

## 5. Material Assignment in Blender

Materials in JuanSchott are flat colors. There are no textures, no normal maps, and no roughness/metallic maps. Materials are assigned in Blender using Principled BSDF nodes with specific parameter constraints.

### 5.1 Principled BSDF Configuration

| Parameter | Value | Notes |
|---|---|---|
| Base Color | Per-material color value | The primary surface color. Defined per material in `material_and_lighting.md`. |
| Metallic | 0.0 or 1.0 | No partial metallic values. Metal surfaces use 1.0; all others use 0.0. |
| Roughness | Per-material value | Typically 0.7–1.0 for non-metal, 0.3–0.5 for metal. Matches the stepped lighting shader's expectations. |
| Emission | 0.0 (default) | Only used for emissive elements (projectiles, pickups). Set to a non-zero color for glowing objects. |
| Normal | Default (no map) | No normal maps are used. |

### 5.2 Material Naming Convention

Materials in Blender are named using the pattern `mat_[color_or_purpose]`:

- `mat_red_team`
- `mat_blue_team`
- `mat_concrete_light`
- `mat_metal_dark`
- `mat_emissive_orange`

### 5.3 Material Mapping in Code

The exported glTF file contains material definitions with the Principled BSDF parameters. When Bevy loads the glTF, the materials are initially loaded as standard PBR materials. A Bevy system then iterates over loaded materials, matches them by name to `CustomMaterial` definitions (the game's custom flat-color shader), and replaces them.

The mapping is defined in a code registry:

```rust
fn get_custom_material(blender_material_name: &str) -> CustomMaterial {
    match blender_material_name {
        "mat_red_team" => CustomMaterial { base_color: Color::srgb(0.85, 0.15, 0.15), ..default() },
        "mat_concrete_light" => CustomMaterial { base_color: Color::srgb(0.82, 0.80, 0.76), ..default() },
        // ... etc.
        _ => CustomMaterial { base_color: Color::srgb(0.5, 0.5, 0.5), ..default() },
    }
}
```

This approach decouples the Blender material (which is used for preview purposes during authoring) from the runtime material (which uses the custom shader). Artists see a reasonable approximation in Blender's viewport, while the game uses the exact flat-color shader.

---

## 6. Collider Generation

JuanSchott uses the Avian physics engine for collision detection. Colliders are generated using two strategies depending on the entity type.

### 6.1 Static Map Geometry — Trimesh Colliders

Static map geometry (floors, walls, ramps, platforms) uses triangle-mesh colliders generated directly from the visual mesh. Avian's `Collider::trimesh_from_mesh` function converts a Bevy `Mesh` asset into a `TrimeshCollider` at load time.

**Procedure:**

1. Load the map glTF file.
2. For each mesh entity tagged as collision geometry (by Blender object name convention: `collision_[description]` or simply all mesh objects in the map), extract the `Mesh` handle.
3. Call `Collider::trimesh_from_mesh(&mesh)` to generate the collider.
4. Attach the collider to the entity as a `Collider` component.

**Limitations and considerations:**

- Trimesh colliders cannot be used for dynamic (moving) bodies in Avian. They are static-only.
- Trimesh colliders are more expensive than primitive colliders (cuboid, capsule) for collision queries. However, for a single static map, the performance impact is negligible because the broadphase culling eliminates most collision pairs.
- The visual mesh and collision mesh can be the same object (no separate collision geometry needed for simple maps). For complex maps with decorative geometry, a separate low-poly collision mesh can be used.

### 6.2 Dynamic Entities — Primitive Colliders

Dynamic entities (players, projectiles, pickups, moving platforms) use primitive collider shapes:

| Entity Type | Collider Shape | Dimensions |
|---|---|---|
| Player | `Collider::capsule(radius, height)` | radius: 0.3m, height: 1.4m |
| Projectile | `Collider::sphere(radius)` | radius: 0.05m |
| Pickup | `Collider::cuboid(x, y, z)` | varies by pickup type |
| Moving Platform | `Collider::cuboid(x, y, z)` | matches visual dimensions |

Primitive colliders are defined in code at entity spawn time. They are not exported from Blender. The collider dimensions are tuned per entity type and documented in the game's code, not in Blender.

### 6.3 Collision Layers

Avian's collision layers are used to prevent unnecessary collision checks:

| Layer | Contents | Collides With |
|---|---|---|
| Layer 0 | Static map geometry | Layer 1, Layer 2 |
| Layer 1 | Players | Layer 0, Layer 1 |
| Layer 2 | Projectiles | Layer 0, Layer 1 |
| Layer 3 | Pickups (sensors) | Layer 1 |

---

## 7. Map Composition

Each map is a single glTF file containing all static geometry for that map. The composition strategy is designed for simplicity and fast load times.

### 7.1 Map File Structure

A map glTF file contains:

- **Geometry meshes.** All static world geometry: floors, walls, ramps, platforms, pillars, and decorative elements.
- **Spawn point empties.** Empty objects with names matching the spawn point naming convention. These are loaded as transform-only entities.
- **Marker empties.** Additional markers for gameplay elements: pickup spawn locations, capture zone centers, and other positional data.

The map file does **not** contain:

- Players, weapons, or projectiles (spawned dynamically in code).
- Lights or cameras (configured in code).
- Materials as textures (all materials are flat colors processed in code).

### 7.2 Map Loading Procedure

1. **Load glTF.** Bevy's asset server loads `assets/maps/map_[name].glb`.
2. **Spawn scene.** The glTF scene is spawned as a child of the map root entity.
3. **Process spawn points.** A system scans for entities with names matching `spawn_*` and registers their transforms in a `SpawnPoints` resource.
4. **Process markers.** A system scans for entities with names matching `marker_*` and registers their positions.
5. **Generate colliders.** For each mesh entity in the map, generate a trimesh collider (unless the entity is marked as non-collidable via name prefix `decor_`).
6. **Replace materials.** Map all Principled BSDF materials to `CustomMaterial` instances using the code material registry.

### 7.3 Map Blender File Organization

Each map has a single `.blend` source file stored in `blender/maps/`. The Blender file is organized into collections:

| Collection | Contents |
|---|---|
| `geometry` | All visible mesh objects (floors, walls, platforms) |
| `collision` | Collision mesh objects (if separate from visual geometry) |
| `spawn_points` | Empty objects for player spawn locations |
| `markers` | Empty objects for gameplay markers |
| `reference` | Reference images, dimensions, and notes (not exported) |

---

## 8. Asset Hot-Reloading

Bevy's built-in asset hot-reloading is the primary iteration tool for artists. It enables rapid visual feedback during development.

### 8.1 How It Works

Bevy's asset server uses the `notify` crate to watch the `assets/` directory for filesystem changes. When a file is modified, the asset server reloads the asset and notifies all systems that hold handles to it.

**Workflow for artists:**

1. Run the game in development mode (`cargo run`).
2. Open the Blender source file.
3. Make changes to the model or material.
4. Export the updated `.glb` file to the same path in `assets/`.
5. Bevy detects the file change and hot-reloads the asset.
6. The updated model/material appears in the running game within 1–2 seconds.

### 8.2 Hot-Reload Scope

The following assets support hot-reloading:

| Asset Type | Hot-Reload Support | Notes |
|---|---|---|
| glTF models (.glb) | Yes | Mesh geometry and skeletal data are reloaded |
| Materials (via glTF) | Partial | Material parameters update, but shader swaps require restart |
| Animations (via glTF) | Yes | Animation clips reload automatically |
| Bevy asset files (.ron) | Yes | Config files reload |

### 8.3 Limitations

- Collider geometry is generated from the mesh at load time. Hot-reloading a mesh does not automatically regenerate its collider. For collision changes, a restart is required.
- Spawn point positions update on hot-reload (since they come from the glTF scene), but the `SpawnPoints` resource must be refreshed by the spawn-point system.
- Entity hierarchy changes (adding or removing objects in the glTF) may require a restart to properly rebuild the scene graph.

---

## 9. Animation Export

Skeletal animations are exported from Blender as glTF animation clips. JuanSchott uses these for player model animations (run, jump, idle, weapon-specific poses) and any other animated elements.

### 9.1 Animation Types

| Animation | Scope | Usage |
|---|---|---|
| Player movement | Full skeleton (arms, legs, torso) | Locomotion state machine |
| Weapon animations | Arms + weapon skeleton | Weapon-specific actions (reload, fire, equip) |
| View model animations | Arms + weapon skeleton | First-person weapon animations |
| Environmental | Object-level transforms | Doors, platforms, animated props |

### 9.2 Export Configuration

Animations are exported with the standard glTF export settings. Additional animation-specific settings:

| Setting | Value |
|---|---|
| Animation Mode | Actions (export each NLA action as a separate clip) |
| Frame Range | Keyframes only (optimize) |
| Sampling Rate | 24 fps (game's animation rate) |
| Always Sample Animations | No (preserve keyframe interpolation) |

### 9.3 Animation Loading in Bevy

Bevy's glTF loader extracts animation clips from the glTF file and stores them as `AnimationClip` assets. The game code accesses them via the glTF scene's `animations` field.

**Playback:** Bevy's `AnimationPlayer` component is attached to the animated entity. The game code triggers animations by name:

```rust
fn play_animation(
    player: &mut AnimationPlayer,
    clips: &AnimationClips,
    animation_name: &str,
    transition_duration: f32,
) {
    if let Some(clip) = clips.get(animation_name) {
        player.play_with_transition(clip, transition_duration);
    }
}
```

### 9.4 Animation Naming Convention

Animation clips in Blender are named using the pattern `anim_[entity]_[action]`:

- `anim_player_idle`
- `anim_player_run`
- `anim_player_jump`
- `anim_weapon_rifle_fire`
- `anim_weapon_rifle_reload`
- `anim_viewmodel_rifle_equip`

---

## 10. Blender File Organization

Source files are organized in the `blender/` directory at the project root. This directory is tracked in Git using Git LFS (Large File Storage) due to the binary nature of `.blend` files.

### 10.1 Directory Structure

```
blender/
├── maps/
│   ├── map_arena_01.blend
│   ├── map_bridge.blend
│   └── map_complex.glb          (exported reference, optional)
├── models/
│   ├── player/
│   │   ├── model_player_default.blend
│   │   └── model_player_heavy.blend
│   ├── weapons/
│   │   ├── weapon_rifle.blend
│   │   ├── weapon_pistol.blend
│   │   └── weapon_shotgun.blend
│   ├── viewmodels/
│   │   ├── model_viewmodel_rifle.blend
│   │   └── model_viewmodel_pistol.blend
│   └── props/
│       ├── model_prop_crate.blend
│       └── model_prop_pillar.blend
└── templates/
    └── material_template.blend   (shared material definitions for reference)
```

### 10.2 Git LFS Configuration

`.gitattributes` tracks `.blend` files through Git LFS:

```
*.blend filter=lfs diff=lfs merge=lfs -text
*.glb filter=lfs diff=lfs merge=lfs -text
```

Exported `.glb` files are stored in the `assets/` directory, which is also tracked in Git LFS. This ensures that the build can reproduce the exact same assets without requiring Blender to be installed.

### 10.3 One File Per Asset

Each distinct asset type has its own `.blend` file. This convention prevents merge conflicts when multiple artists work on different assets simultaneously. A single `.blend` file per map, per weapon, and per player model variant means that two artists can work on different maps without ever touching the same file.

---

## 11. Quality Assurance

### 11.1 Export Validation Checklist

Before committing an exported `.glb` file, artists verify:

- [ ] File exports without Blender warnings or errors.
- [ ] File size is reasonable (maps: 100KB–2MB; models: 10KB–200KB).
- [ ] No cameras or lights are present in the export (verify in a glTF viewer like `gltf-viewer.donmccurdy.com`).
- [ ] Materials have correct color values (verify in glTF viewer).
- [ ] Animations play correctly (verify in glTF viewer).
- [ ] Spawn points have correct names and positions.
- [ ] Mesh normals are correct (no inverted faces).

### 11.2 Automated Validation (Future)

A CI step can validate exported `.glb` files using the `gltf` Rust crate to:

- Check for presence of cameras/lights (should be none).
- Verify naming conventions on all nodes.
- Confirm material count matches expectations.
- Validate triangle counts against performance budgets.

---

## 12. Summary

JuanSchott's asset pipeline is intentionally minimal: Blender to glTF to Bevy, with no intermediate steps. The flat-color, texture-free art style eliminates the need for texture compression, mipmap generation, and material baking. Asset hot-reloading provides fast iteration for artists. Standardized naming conventions enable automated collider generation and spawn-point extraction. The pipeline's simplicity reduces both artist onboarding time and the risk of asset-related bugs at runtime.
