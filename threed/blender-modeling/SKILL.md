---
name: blender-modeling
description: - Hard-surface modeling (weapons, vehicles, mechanical parts)
category: threed
tags: [blender-modeling]
---

## When to Use
- Modeling 3D assets: characters, props, environments, architecture
- Hard-surface modeling (weapons, vehicles, mechanical parts)
- Organic modeling (creatures, characters, terrain features)
- Preparing meshes for game engines or rendering pipelines
- Retopology workflows for high-poly to low-poly conversion

## Core Concepts
- Mesh topology: quads vs tris vs n-gons; edge flow for deformation
- Modifiers stack order matters (Array → Mirror → Subdivision)
- UV mapping: seams, islands, overlap avoidance, texel density
- Normals: face normals, vertex normals, auto-smooth, custom split normals
- Scale and units: Blender default is meters; match target engine scale
- Non-destructive workflow: keep modifiers stackable, use shape keys

## Workflow
1. Block out with cubes/spheres at reference scale
2. Refine proportions with proportional editing (O key)
3. Add loop cuts (Ctrl+R) for supporting edges before subdivision
4. Mirror modifier for symmetry; apply only when modeling asymmetry
5. UV unwrap: mark seams (Ctrl+E → Mark Seam), then U → Unwrap
6. Check for flipped normals: Mesh → Normals → Recalculate Outside
7. Export: File → Export → glTF 2.0 (.glb) for web/game engines

## Key Patterns
```
# Python scripting for batch operations
import bpy
obj = bpy.context.active_object
bpy.ops.object.modifier_add(type='MIRROR')
obj.modifiers["Mirror"].use_clip = True

# Apply all modifiers before export
for mod in obj.modifiers:
    bpy.ops.object.modifier_apply(modifier=mod.name)

# Set texel density uniformly
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.uv.select_all(action='SELECT')
bpy.ops.uv.average_islands_scale()
```

Blender config for consistent units:
```python
# Project settings script
bpy.context.scene.unit_settings.system = 'METRIC'
bpy.context.scene.unit_settings.scale_length = 1.0
bpy.context.scene.render.fps = 30
```

## Pitfalls
- Forgetting to apply scale before export (causes wrong size in engine)
- N-gons cause shading artifacts on subdivision — triangulate for export
- Blender Y-up vs engine Z-up: always set correct forward/up in export
- Overlapping UVs cause baked lighting errors — check UV Sync Select
- Huge vertex counts from unapplied subdivision — use decimate modifier

## Verification
- Import glTF into glTF Viewer (https://gltf-viewer.donmccurdy.com/) for validation
- Check wireframe mode (Z → Wireframe) for bad topology
- Use Mesh → Clean Up → Degenerate Dissolve to fix zero-area faces
- Run `bpy.ops.mesh.normals_make_consistent(inside=False)` to fix normals
- Validate with Blender's built-in 3D printing toolbox (overhang, thickness)