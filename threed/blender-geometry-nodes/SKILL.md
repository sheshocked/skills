---
name: blender-geometry-nodes
description: - Parameterized asset generation (change inputs → new variant)
category: threed
tags: [blender-geometry-nodes]
---

## When to Use
- Procedural modeling: fences, staircases, repeated structures, foliage
- Parameterized asset generation (change inputs → new variant)
- Scattering objects across surfaces (grass, rocks, debris)
- Creating complex shapes without manual modeling
- Real-time geometry modification in Blender 3.x+ (Geometry Nodes)

## Core Concepts
- Node tree replaces traditional modifier approach with visual programming
- Geometry types: Mesh, Curve, Point Cloud, Instances, Volume
- Sampling: Point, Edge, Face, Corner (each has separate attributes)
- Attributes store per-element data (position, color, custom data)
- Instances vs real geometry: Instances are lightweight until realized
- Named attributes pass data between nodes: `Named Attribute` node

## Workflow
1. Add Geometry Nodes modifier to object
2. Start with Group Input → Group Output
3. Use Set Position for vertex displacement
4. Use Distribute Points on Faces for scattering
5. Use Instance on Points to place objects at scatter points
6. Control with inputs in the modifier panel for non-destructive params
7. Realize Instances only when needed for boolean or physics operations

## Key Patterns
```
# Random rotation and scale on scattered objects
Distribute Points on Faces → Instance on Points
  → Instance: your object
  → Rotation: random (Vector Math → Combine XYZ with Random Value)
  → Scale: Random Value (Float, min 0.5, max 1.5)

# Align instances to surface normal
Align Euler to Vector node:
  Input: Normal of distributed face
  Axis: Z
  This rotates each instance to sit flat on the surface

# Parameterized fence generation
Curve Line → Resample Curve (count driven by curve length)
  → Instance on Points (post): vertical bars
  → Instance on Points (post): horizontal rails
  Set input "rail_count" in modifier panel

# Vertex color-driven material assignment
Capture Attribute (store named attribute "material_id")
  → Use in Shader Editor with Attribute node
```

Geometry Nodes setup for procedural terrain:
```
Grid → Set Position (noise displacement via Noise Texture * Combine XYZ)
  → Subdivide Mesh (level: 2)
  → Set Position (second noise layer for detail)
  → Shade Smooth
```

## Pitfalls
- Realized instances lose the instance data — backup before realizing
- Heavy point counts (>1M) slow viewport: use viewport display limits
- Named attributes must match exact names — typos silently fail
- Curve-based approaches need "Curve to Mesh" before export
- Frame range matters: cache simulations with Cache node for animation

## Verification
- Check Statistics overlay (N panel → Viewport) for vertex/face counts
- Use Spreadsheet editor to inspect attribute values at each element
- Test with extreme parameter values to find breaking points
- Export to game engine to verify instances converted correctly
- Compare file size before/after: instances should be lighter