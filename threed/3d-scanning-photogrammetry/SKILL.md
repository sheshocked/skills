---
name: 3d-scanning-photogrammetry
description: - Cultural heritage preservation and digital archiving
category: threed
tags: [3d-scanning-photogrammetry]
---

## When to Use
- Creating 3D models from real-world objects or environments
- Cultural heritage preservation and digital archiving
- Creating realistic props for games/film from physical objects
- Quality inspection and measurement of manufactured parts
- Generating textures from real-world materials (scan → PBR)

## Core Concepts
- Photogrammetry: reconstruct 3D from multiple overlapping photos
- SfM (Structure from Motion): camera pose estimation from images
- MVS (Multi-View Stereo): dense point cloud from camera poses
- Texture mapping: projecting photos onto reconstructed mesh
- Mesh cleaning: filling holes, removing noise, smoothing
- Camera requirements: 50-200 overlapping images, 70-80% overlap
- Lighting: diffuse (overcast) or cross-polarized for texture quality

## Workflow
1. Capture photos: consistent lighting, 70-80% overlap, all angles
2. Import into photogrammetry software (Meshroom, RealityCapture, Metashape)
3. Align photos → generate sparse point cloud
4. Dense reconstruction → generate dense point cloud
5. Surface reconstruction → mesh
6. Texture projection → textured mesh
7. Clean mesh: fill holes, reduce polygons, fix normals
8. Export for target use (game engine, 3D print, web viewer)

## Key Patterns
```bash
# AliceVision Meshroom CLI (free, open-source)
# Install via AppImage or Docker
docker run -v $(pwd):/data alicevision/meshroom_compute \
  --input /data/photos \
  --output /data/output \
  --config default

# Export command after processing
python -c "
import json
with open('output/structureFromMotion.abc') as f:
    data = json.load(f)
print(data['outputs']['cameras'])
"

# MeshLab batch processing (Python)
# Clean and decimate scanned mesh
import pymeshlab
ms = pymeshlab.MeshSet()
ms.load_new_mesh('scan_raw.ply')
ms.compute_normal_per_face()
ms.meshing_remove_faces_outside_core()
ms.meshing_decimation_quadric_edge_collapse(targetfacenum=50000)
ms.save_current_mesh('scan_cleaned.ply')
```

Photo capture checklist:
```
Camera settings:
  - Fixed ISO (100-400)
  - Fixed white balance
  - Fixed aperture (f/8 - f/11)
  - No flash (diffuse natural light)
  - RAW format preferred

Coverage requirements:
  - 360° horizontal coverage (minimum 4 angles per side)
  - Top-down views for flat surfaces
  - Close-ups for detail areas
  - Avoid reflective/transparent surfaces (or use polarizing filter)
  - Min 50 photos, optimal 100-200 for medium objects
```

## Pitfalls
- Moving objects during capture cause reconstruction artifacts
- Specular highlights break SfM matching — diffuse lighting essential
- Thin structures (wires, chains) rarely scan well — manual cleanup needed
- Scale reference: include a calibration object or known dimension
- Texture stretching on steep surfaces — capture from more angles
- File sizes: raw scans can be 100MB+ — optimize for delivery

## Verification
- Overlay scan on reference measurements: check scale accuracy (<1mm)
- Visual inspection in MeshLab for holes, spikes, and artifacts
- Compare scan dimensions with caliper measurements
- Test decimated mesh in target engine for polygon budget compliance
- Generate normal map from high-poly to low-poly for game use