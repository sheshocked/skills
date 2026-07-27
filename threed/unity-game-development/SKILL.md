---
name: unity-game-development
description: - Rapid prototyping with Play Mode testing
category: threed
tags: [unity-game-development]
---

## When to Use
- Building 2D/3D games for PC, console, mobile, or web
- Rapid prototyping with Play Mode testing
- AR/VR applications with XR Interaction Toolkit
- Simulation and training applications
- Any interactive 3D project needing physics, audio, UI, and scripting

## Core Concepts
- GameObjects + Components: entity-component architecture
- Transform: position, rotation (quaternion), scale
- MonoBehaviour: custom C# scripts attached to GameObjects
- Prefabs: reusable GameObject templates with overrides
- Scenes: distinct levels or menus loaded/unloaded at runtime
- Physics: Rigidbody, Collider, Joint, PhysicsMaterial
- Render Pipeline: URP (mobile/mid) vs HDRP (high-end)

## Workflow
```bash
# Unity Hub install (CLI)
unity-hub --headless install --version 2022.3.20f1 --module android

# Create project from template
unity-hub --headless create --project-path ./MyGame --template 3d
```

```
# Project structure
Assets/
  Scripts/           # C# MonoBehaviours
    Player/
      PlayerMovement.cs
      PlayerHealth.cs
  Prefabs/
    Enemies/
  Scenes/
    MainMenu.unity
    Level01.unity
  Materials/
  Art/
    Models/
    Textures/
ProjectSettings/
  ProjectSettings.asset    # Build settings, quality, input
  InputManager.asset       # Input axes definitions
```

## Key Patterns
```csharp
// Player movement with Input System
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    [SerializeField] private float speed = 5f;
    [SerializeField] private float jumpForce = 8f;
    private Rigidbody rb;
    private bool isGrounded;

    void Start() => rb = GetComponent<Rigidbody>();

    void Update()
    {
        float h = Input.GetAxisRaw("Horizontal");
        float v = Input.GetAxisRaw("Vertical");
        Vector3 move = new Vector3(h, 0, v).normalized * speed;
        rb.linearVelocity = new Vector3(move.x, rb.linearVelocity.y, move.z);

        if (Input.GetButtonDown("Jump") && isGrounded)
            rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
    }

    void OnCollisionEnter(Collision col)
    {
        if (col.gameObject.CompareTag("Ground")) isGrounded = true;
    }
}

// Object pooling for projectiles
public class ObjectPool : MonoBehaviour
{
    [SerializeField] private GameObject prefab;
    [SerializeField] private int poolSize = 20;
    private Queue<GameObject> pool = new Queue<GameObject>();

    void Awake()
    {
        for (int i = 0; i < poolSize; i++)
        {
            var obj = Instantiate(prefab, transform);
            obj.SetActive(false);
            pool.Enqueue(obj);
        }
    }

    public GameObject Get()
    {
        var obj = pool.Dequeue();
        obj.SetActive(true);
        pool.Enqueue(obj);
        return obj;
    }
}

// ScriptableObject for game data
[CreateAssetMenu(fileName = "EnemyData", menuName = "Game/EnemyData")]
public class EnemyData : ScriptableObject
{
    public float health = 100f;
    public float speed = 3f;
    public int damage = 10;
    public GameObject modelPrefab;
}
```

## Pitfalls
- Update() runs every frame — move heavy logic to FixedUpdate() for physics
- Find() and FindWithTag() are slow in Update — cache references in Start()
- DontDestroyOnLoad causes memory leaks if objects accumulate across scenes
- Prefab instances with missing scripts show "Missing (Mono Script)" — reassign
- URP vs HDRP materials are NOT compatible — switching requires rework
- Serialization: [SerializeField] private fields serialize; public ones are editor-only

## Verification
- Build for target platform: File → Build Profiles → Select Platform
- Profile with Unity Profiler: CPU/GPU frames, memory allocation
- Use Frame Debugger (Window → Analysis) to inspect draw calls
- Test on lowest target device early (mobile: check draw calls < 100)
- Addressables for asset loading: validate with Window → Asset Management