---
name: unreal-engine-blueprints
description: - Rapid prototyping of game mechanics and interactions
category: threed
tags: [unreal-engine-blueprints]
---

## When to Use
- Visual scripting for gameplay logic without writing C++
- Rapid prototyping of game mechanics and interactions
- Level design with Blueprints: triggers, doors, elevators
- UI creation with UMG (Unreal Motion Graphics)
- Multiplayer game setup with replication nodes
- Cinematic sequences with Sequencer

## Core Concepts
- Blueprint Class: visual script containers for game logic
- Event Graph: execution flow triggered by events (BeginPlay, Tick)
- Components: Scene, Static Mesh, Camera, Audio, Light
- Variables: Local, Instance, Class; types: Float, Bool, Vector, Object
- Components of execution: white wires for flow, colored for data
- C++ can extend Blueprints via UFUNCTION(BlueprintCallable)
- UE5: Nanite (virtualized geometry), Lumen (global illumination)

## Workflow
```
# Project structure
Content/
  Blueprints/
    Characters/        # BP_PlayerCharacter, BP_Enemy
    Props/             # BP_Door, BP_Switch, BP_Crate
    Weapons/           # BP_Rifle, BP_Bullet
    UI/                # WBP_MainMenu, WBP_HUD
  Levels/
  Materials/
  Maps/
```

1. Create Blueprint Class → parent (Character, Actor, Pawn)
2. Event Graph: BeginPlay for initialization
3. Add components: mesh, collision, camera
4. Wire input events (Enhanced Input System in UE5)
5. Add timers with SetTimerByEvent for delayed actions
6. Test with Play in Editor (PIE) — multiple clients for multiplayer

## Key Patterns
```
# Enhanced Input (UE5)
// In BP_EventGraph
BeginPlay → Get Player Controller → Get Enhanced Input Local Player
  → Get Subsystem → Add Mapping Context → Input Action Move

# Trigger overlap detection
On Component Begin Overlap (BoxCollision):
  → Cast to BP_PlayerCharacter
  → if valid: Add Impulse (upward) → Set Timer by Function Name ("OpenDoor")

# AI: Behavior Tree task
// C++ exposes to Blueprint
UCLASS(Blueprintable)
class AEnemyAI : public ACharacter
{
    UFUNCTION(BlueprintCallable)
    void MoveToTarget(AActor* Target);
};

// BP: Select → Find closest player → Move To (AI MoveTo node)

# Replication for multiplayer
// BP_Actor
Replicates = true
On Rep (Variable Name): Update visuals when replicated
Set Rep: On Rep Notify
```

Blueprint function for door with key check:
```
Event → Get Player Inventory → Has Item ("Key")?
  True: Play Animation (DoorOpen) → Set Collision (No Collision)
  False: Play Sound (Locked) → Print String ("Need a key!")
```

## Pitfalls
- Tick events are expensive — use timers, delegates, or event dispatchers instead
- Blueprint-to-Blueprint calls add overhead for hot paths — migrate to C++
- Circular references between Blueprints cause packaging errors
- Replication: only set variables on server; use RPCs for client→server
- Nanite doesn't support skeletal meshes — use LOD for skinned characters
- Blueprint debugging: breakpoints freeze the game — use Print String for production

## Verification
- Compile all Blueprints: right-click Content folder → Compile All
- Check for blueprint compilation errors in Message Log
- Use Session Frontend → Profiler for frame time analysis
- Package for target platform: File → Package Project
- Test with -game -server -log for dedicated server validation