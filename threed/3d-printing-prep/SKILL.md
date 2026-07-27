---
name: 3d-printing-prep
description: - Fixing mesh issues: non-manifold, inverted normals, gaps
category: threed
tags: [3d-printing-prep]
---

## When to Use
- Preparing 3D models for FDM, SLA, or SLS printing
- Fixing mesh issues: non-manifold, inverted normals, gaps
- Optimizing models for printability and strength
- Converting game/film assets to printable format
- Designing parts with tolerances for assembly

## Core Concepts
- Manifold meshes: watertight, no holes, no self-intersections
- Wall thickness: minimum 0.8mm for FDM, 0.3mm for SLA
- Overhangs: >45° from vertical needs support material
- Layer height: 0.2mm (fast) to 0.05mm (high detail) for FDM
- Print orientation: affects strength, surface finish, support needs
- Infill: 15-20% for decorative, 40-60% for functional parts
- Tolerances: 0.2mm gap for FDM, 0.1mm for SLA (fitting parts)

## Workflow
```bash
# Blender to STL pipeline
# 1. Export as STL: File → Export → STL (.stl)
# 2. Check manifold: import into Meshmixer or PrusaSlicer

# Mesh repair with Python (trimesh)
pip install trimesh numpy
```

```python
import trimesh

# Load and inspect mesh
mesh = trimesh.load('model.stl')
print(f"Watertight: {mesh.is_watertight}")
print(f"Volume: {mesh.volume} mm³")
print(f"Triangles: {len(mesh.faces)}")
print(f"Bounds: {mesh.bounds}")

# Fix non-manifold edges
if not mesh.is_watertight:
    mesh.fill_holes()
    mesh.fix_normals()
    print(f"After repair - Watertight: {mesh.is_watertart}")

# Export fixed mesh
mesh.export('model_fixed.stl')

# Scale for printing (Blender exports in meters, STL needs mm)
scale_factor = 1000  # meters to mm
mesh.apply_scale(scale_factor)
```

Slicer configuration:
```
# PrusaSlicer / Cura settings (FDM)
Layer height:          0.2mm (standard) / 0.1mm (fine)
First layer height:    0.3mm (better adhesion)
Wall count:            3 (for strength)
Top/bottom layers:     4
Infill:                20% (decorative) / 40% (functional)
Infill pattern:        Gyroid (balanced strength)
Support:               Tree supports (less material)
Support threshold:     45° overhang
Print speed:           60mm/s (standard) / 40mm/s (quality)
Temperature:           210°C (PLA) / 240°C (PETG)
Bed temperature:       60°C (PLA) / 80°C (PETG)
Retraction:            0.8mm @ 40mm/s (direct drive)
```

## Key Patterns
```
# Blender Python: prepare mesh for printing
import bpy

bpy.ops.import_mesh.stl(filepath='model.stl')
obj = bpy.context.active_object

# Scale to mm (if exported from Blender in meters)
obj.scale = (1000, 1000, 1000)
bpy.ops.object.transform_apply(scale=True)

# Check volume and area
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='SELECT')
bpy.ops.mesh.print3d_clean()
stats = bpy.ops.mesh.print3d_statistics()
print(stats)

# Hollow mesh for resin printing (save material)
bpy.ops.object.modifier_add(type='SOLIDIFY')
obj.modifiers["Solidify"].thickness = -2.0  # 2mm wall inward
obj.modifiers["Solidify"].offset = -1
bpy.ops.object.modifier_apply(modifier='Solidify')

# Add drainage holes for resin
bpy.ops.mesh.primitive_cylinder_add(radius=1.5, depth=5, location=(0, 0, -5))
hole = bpy.context.active_object
bpy.context.view_layer.objects.active = obj
bpy.ops.object.modifier_add(type='BOOLEAN')
obj.modifiers["Boolean"].operation = 'DIFFERENCE'
obj.modifiers["Boolean"].object = hole
bpy.ops.object.modifier_apply(modifier='Boolean')
bpy.data.objects.remove(hole)
```

## Pitfalls
- Non-manifold geometry fails slicing: always check watertightness
- Thin walls break during printing: add minimum thickness modifier
- Overhangs without supports create spaghetti: use 45° threshold
- Scale mismatch: Blender meters vs STL mm vs slicer mm
- Resin printing needs drainage holes: trapped resin ruins prints
- Support removal: design supports to be easy to remove (breakaway points)

## Verification
- Open in PrusaSlicer/Cura: no slicing errors or red areas
- Check print preview: verify layer adhesion and support placement
- Print test piece: 20mm calibration cube for dimensional accuracy
- Measure printed part: compare dimensions to digital model (<0.2mm tolerance)
- Test assembly: parts should fit with designed tolerances