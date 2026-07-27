---
name: pbr-materials-texturing
description: - Texturing 3D models with realistic surface properties
category: threed
tags: [pbr-materials-texturing]
---

## When to Use
- Creating physically-based rendering materials for games/film
- Texturing 3D models with realistic surface properties
- Building material libraries for consistent visual quality
- Substance Designer/Painter material creation
- Setting up shader networks in Blender, Unity, or Unreal

## Core Concepts
- PBR maps: Base Color, Normal, Roughness, Metallic, AO, Emissive
- Base Color (albedo): diffuse color without lighting information
- Normal Map: simulates surface detail via tangent-space normals
- Roughness: 0 = mirror, 1 = diffuse (inversely related to glossiness)
- Metallic: 0 = dielectric, 1 = metal (affects F0 and color behavior)
- AO (Ambient Occlusion): precomputed soft shadowing in crevices
- Texel density: consistent pixels-per-meter across assets (1024px/m typical)

## Workflow
```
# Substance Painter → Game Engine pipeline
1. Bake mesh maps: AO, Curvature, World Space Normal, Position, Thickness
2. Create base materials: fill layers with color/roughness/metallic
3. Add edge wear using curvature-based generators
4. Add dirt/dust in crevices using AO-based generators
5. Paint detail layers manually for unique features
6. Export maps in target engine's format
```

Map export formats by engine:
```
Unity:    Base Color (sRGB), Normal (Linear), MaskMap (R=AO, G=Metal, B=Rough)
Unreal:   Base Color (sRGB), Normal (Linear), ORM (R=AO, G=Rough, B=Metal)
Web/GL:   Separate PNGs or KTX2 compressed
```

## Key Patterns
```python
# Substance Designer Python: create material graph
import sd

app = sd.getApp()
doc = app.newDoc()
graph = doc.getGraph('MyMaterial')

# Create PBR outputs
output_basecolor = graph.newNode('sbs::compositing/output')
output_basecolor.setIdentifier('basecolor')
output_normal = graph.newNode('sbs::compositing/output')
output_normal.setIdentifier('normal')

# Add noise for surface variation
noise = graph.newNode('sbs::procedural/noise/gaussian_noise')
noise.getParameter('scale').setVal(50.0)
noise.getParameter('disorder').setVal(0.8)

# Connect noise to base color with color correction
color_correct = graph.newNode('sbs::processing/color/color_correction')
color_correct.getParameter('hue_offset').setVal(0.1)
noise.connectTo(color_correct)
color_correct.connectTo(output_basecolor)

app.log('Material graph created with procedural noise')
```

```python
# Blender: PBR material node setup via Python
import bpy

mat = bpy.data.materials.new(name="PBR_Metal")
mat.use_nodes = True
nodes = mat.node_tree.nodes
links = mat.node_tree.links

# Clear defaults
for node in nodes:
    nodes.remove(node)

# Create Principled BSDF
bsdf = nodes.new('ShaderNodeBsdfPrincipled')
bsdf.inputs['Base Color'].default_value = (0.8, 0.1, 0.1, 1.0)
bsdf.inputs['Metallic'].default_value = 0.9
bsdf.inputs['Roughness'].default_value = 0.2

# Connect to output
output = nodes.new('ShaderNodeOutputMaterial')
links.new(bsdf.outputs['BSDF'], output.inputs['Surface'])

# Add texture coordinate for procedural textures
tex_coord = nodes.new('ShaderNodeTexCoord')
mapping = nodes.new('ShaderNodeMapping')
mapping.inputs['Scale'].default_value = (5, 5, 5)
links.new(tex_coord.outputs['Object'], mapping.inputs['Vector'])

# Connect to roughness via noise
noise = nodes.new('ShaderNodeTexNoise')
noise.inputs['Scale'].default_value = 20.0
links.new(mapping.outputs['Vector'], noise.inputs['Vector'])
links.new(noise.outputs['Fac'], bsdf.inputs['Roughness'])

# Assign to active object
bpy.context.object.data.materials.append(mat)
```

## Pitfalls
- sRGB vs Linear: Base Color in sRGB, everything else in Linear (correct gamma)
- Tangent space: Blender and Unity use OpenGL normals (Y+), Unreal uses DirectX (Y-)
- Metallic workflow: metal maps must be binary (0 or 1) — grays cause artifacts
- Texture size mismatches between diffuse and normal maps cause aliasing
- Seams visible in reflections: fix UV seams or use triplanar mapping
- 8-bit precision causes banding on smooth gradients — use 16-bit or BC6H

## Verification
- Validate normal map direction in engine (blue channel should be dominant)
- Check PBR materials under different lighting conditions
- Use roughness/metallic debug views in engine to verify maps
- Compare texel density across all scene assets with visualization tool
- Test on multiple monitors: gamma differences reveal color space issues