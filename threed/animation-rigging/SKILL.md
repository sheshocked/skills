---
name: animation-rigging
description: - Facial animation and blend shape setup
category: threed
tags: [animation-rigging]
---

## When to Use
- Character animation for games, film, or web
- Facial animation and blend shape setup
- Rigging props (doors, vehicles, mechanical parts)
- Retargeting animations between different character rigs
- Creating procedural/expression-based animation systems

## Core Concepts
- Armature: hierarchy of bones (joints) driving mesh deformation
- Weight painting: how much each bone influences nearby vertices
- IK (Inverse Kinematics): move hand → arm bones solve automatically
- FK (Forward Kinematics): each bone rotated individually
- Constraints: track-to, copy-rotation, limit-rotation, IK
- Blend shapes / shape keys: morph targets for facial expressions
- Animation curves: F-curves control timing and easing between keyframes

## Workflow
```
# Rigging pipeline
1. Model character in T-pose or A-pose
2. Create armature: add bones in Edit Mode
3. Parent mesh to armature with Armature Deform (automatic weights)
4. Test in Pose Mode: move bones, check deformation
5. Weight paint corrections for bad areas (fingers, shoulders, spine)
6. Add IK constraints for limbs (legs, arms)
7. Add constraints for spine, head tracking
8. Create control rig: user-friendly handles around the character
```

## Key Patterns
```python
# Blender Python: create armature programmatically
import bpy

bpy.ops.object.armature_add(enter_editmode=True)
armature = bpy.context.object.data

# Add bones
bones = {
    'root': (0, 0, 0),
    'spine': (0, 0, 0.8),
    'chest': (0, 0, 1.2),
    'head': (0, 0, 1.6),
    'upper_arm_L': (0.3, 0, 1.1),
    'lower_arm_L': (0.6, 0, 1.1),
    'hand_L': (0.9, 0, 1.1),
}

for name, (x, y, z) in bones.items():
    bone = armature.edit_bones.new(name)
    bone.head = (x, y, z)
    bone.tail = (x, y, z + 0.15)
    if name.startswith('upper_arm'):
        bone.parent = armature.edit_bones['chest']

bpy.ops.object.mode_set(mode='POSE')

# IK constraint on leg
pbone = bpy.context.object.pose.bones['lower_leg_L']
ik = pbone.constraints.new('IK')
ik.target = bpy.data.objects['IK_target_L']
ik.chain_count = 2  # affects upper_leg and lower_leg

# Auto-weight painting
bpy.ops.object.mode_set(mode='WEIGHT_PAINT')
# Use Weights → Automatic Weights from Bones
```

Animation retargeting setup:
```python
# Blender: retarget from Rigify to Mixamo
# 1. Import both rigs
# 2. Use Animation → Bake Action on source
# 3. NLA Editor: add strip with baked animation
# 4. Use "Copy Transforms" constraint to map bones
# Bone mapping: Mixamo:Hips → Rigify:torso
#                Mixamo:Spine → Rigify:spine.001
#                Mixamo:LeftArm → Rigify:upper_arm.L
```

## Pitfalls
- Automatic weights fail on complex geometry — manually weight paint
- IK/FK switching needs a snap system or the animator loses control
- Overlapping bone influence causes double-bending — use weight constraints
- Blend shapes need consistent topology between base and target meshes
- Animation baking removes non-linear editing ability — backup before baking
- Export to game engines: apply all transforms before export, check bone orientation

## Verification
- Test full range of motion for each joint: extreme poses reveal weight issues
- Export to target engine and verify bone names match expected skeleton
- Check for mesh stretching, pinching, and volume loss during deformation
- Play animation at 1x speed: check for jitter or unnatural motion
- Compare bone orientations: Blender Z-up vs engine Y-up differences