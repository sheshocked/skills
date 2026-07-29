---
name: blender-procedural-modeling
description: Headless asset generation inside Blender using python bpy module and dynamic procedural meshes math.
category: threed
tags: [blender, bpy, procedural-modeling, headless, scripts, python]
---

# Blender Headless Procedural Modeling Masterclass

## When to Use
Use to build server-side procedural mesh rendering engines, dynamically generating game assets or customized 3D profiles in background environments.

## Prerequisites
- Blender compiled and installed on execution servers.

## Workflow
1. Write Blender python scripting workflows utilizing the native `bpy` module.
2. Initialize vertex, face, and UV maps structures dynamically using math algorithms.
3. Build shaders configurations and apply texture arrays.
4. Run Blender headlessly (`blender -b -P script.py`) to output GLTF/FBX packages.

## Key Patterns

### Python Procedural Mesh Generator (generate_mesh.py)
```python
import bpy
import bmesh
import math

def create_procedural_gear(teeth_count=12, radius=1.0, depth=0.2):
    # 1. Clean existing meshes
    bpy.ops.object.select_all(action='DESELECT')
    bpy.ops.object.select_by_type(type='MESH')
    bpy.ops.object.delete()

    # 2. Create mesh container
    mesh = bpy.data.meshes.new("ProceduralGearMesh")
    obj = bpy.data.objects.new("ProceduralGear", mesh)
    bpy.context.collection.objects.link(obj)

    # 3. Build geometry using BMesh
    bm = bmesh.new()
    
    # Generate gear vertices
    vertices = []
    angle_step = (2 * math.pi) / (teeth_count * 2)
    for i in range(teeth_count * 2):
        angle = i * angle_step
        # Alternate radius to create teeth
        r = radius if i % 2 == 0 else radius * 0.8
        x = math.cos(angle) * r
        y = math.sin(angle) * r
        vertices.append(bm.verts.new((x, y, 0)))

    # Extrude mesh to add depth
    bm.verts.ensure_lookup_table()
    geom = bmesh.ops.extrude_discrete_faces(bm, faces=[bm.faces.new(vertices)])
    for face in geom['faces']:
        bmesh.ops.translate(bm, vec=(0, 0, depth), verts=face.verts)

    # 4. Finalize and export
    bm.to_mesh(mesh)
    bm.free()

    # Export to GLTF
    bpy.ops.export_scene.gltf(filepath="/tmp/procedural_gear.gltf", export_format='GLTF_EMBEDDED')

if __name__ == "__main__":
    create_procedural_gear(teeth_count=16, radius=1.5, depth=0.3)
```

## Pitfalls
- **Context requirements:** Some `bpy.ops` commands require an active UI window layout. Bypass these limitations by using `bmesh` modules directly to manipulate meshes mathematically without UI dependency.
- **Memory leaks in background loops:** Blender maintains file caches in RAM. Call `bpy.ops.wm.read_factory_settings()` periodically inside loops to purge cache.

## Verification
- Run: `blender -b -P generate_mesh.py`.
- Verify the exported file is written to `/tmp/procedural_gear.gltf` and loads correctly in online GLTF viewers.
