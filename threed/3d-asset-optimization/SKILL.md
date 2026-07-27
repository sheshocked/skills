---
name: 3d-asset-optimization
description: - Texture atlas baking to minimize draw calls
category: threed
tags: [3d-asset-optimization]
---

## When to Use
- Reducing polygon count for mobile/web VR targets
- Texture atlas baking to minimize draw calls
- LOD (Level of Detail) setup for streaming environments
- Preparing assets for real-time rendering constraints
- Reducing memory and GPU bandwidth consumption

## Core Concepts
- Triangle budget: mobile (50K-100K), web (100K-500K), PC (1M+)
- Draw calls: each material swap = new draw call; atlasing reduces this
- Texture memory: 1K×1K RGBA8 = 4MB; use compression (BCn, ASTC, ETC2)
- LOD levels: typically 3-4 meshes with decreasing poly count
- Texture streaming: only load visible mip levels
- Vertex formats: position + normal + UV = 32 bytes/vertex minimum

## Workflow
```bash
# Blender to game-ready pipeline
# 1. High-poly sculpt → decimate for low-poly
# 2. Bake normals/ambient occlusion from high to low
# 3. Export low-poly + baked textures

# Using gltf-transform CLI for optimization
npm install -g @gltf-transform/cli
gltf-transform optimize input.glb output.glb --compress draco
gltf-transform resize input.glb output.glb --width 1024 --height 1024
gltf-transform dedup input.glb output.glb
gltf-transform prune input.glb output.glb
```

```
# Poly count targets per asset type
Prop (crate, barrel):    50-200 triangles
Weapon (sword, gun):     200-800 triangles
Character (game):        3,000-15,000 triangles
Vehicle:                 1,000-5,000 triangles
Environment piece:       100-1,000 triangles
Full scene (mobile):     50,000-100,000 triangles
```

## Key Patterns
```
# Blender Python: batch decimate
import bpy
for obj in bpy.context.scene.objects:
    if obj.type == 'MESH':
        mod = obj.modifiers.new('Decimate', 'DECIMATE')
        mod.ratio = 0.5  # reduce to 50%
        bpy.context.view_layer.objects.active = obj
        bpy.ops.object.modifier_apply(modifier='Decimate')

# Texture atlas creation in Blender
# UV → Merge by Distance (merge overlapping UV islands)
# Bake diffuse/normal/AO from individual materials to atlas
bpy.ops.object.bake(type='DIFFUSE')

# LOD distance thresholds (common values)
# LOD0: 0-10m (full detail)
# LOD1: 10-30m (50% triangles)
# LOD2: 30-80m (25% triangles)
# LOD3: 80m+ (impostor/billboard)

# Mesh combining for fewer draw calls
Unity:
  Static batch:勾选 Static checkbox
  Dynamic batch: <300 verts, single material
  GPU Instancing: same mesh + material, different transforms
```

## Pitfalls
- Over-optimization: losing silhouette at distance, need to test at actual viewing distance
- Texture compression artifacts on normals: use BC5/ATC for normal maps
- Mipmapping: must be enabled or textures shimmer at distance
- UV seam baking creates visible seams — fix with bake margin and padding
- Draco compression adds decode time on load — test load time vs size tradeoff
- Atlas bleeding: textures near atlas edges bleed into adjacent textures

## Verification
- RenderDoc capture: check draw calls, state changes, texture sizes
- Blender Statistics panel: vertex/face count per object
- GPU profiling: texture memory, vertex buffer sizes
- Compare load times before/after optimization on target platform
- Visual QA: screenshot comparisons at all LOD distances