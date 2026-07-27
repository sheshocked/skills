---
name: vfx-particles
description: - UI effects: particles on button clicks, transitions
category: threed
tags: [vfx-particles]
---

## When to Use
- Game effects: explosions, fire, smoke, magic, debris
- UI effects: particles on button clicks, transitions
- Environmental effects: rain, snow, dust, fog
- Trail effects: projectiles, sword swings, vehicle exhaust
- Post-process effects: screen flash, distortion, color grading

## Core Concepts
- Particle systems: emitter → individual particles → death
- Emitter types: point, cone, sphere, mesh surface, volume
- Particle lifecycle: birth → update (velocity, gravity) → death
- Sub-emitters: particles spawn other particles on death/collision
- Billboard rendering: camera-facing quads for efficiency
- GPU particles: compute shader-driven for 100K+ particle counts
- Curl noise: organic swirling motion for fire and smoke

## Workflow
```
# Unity VFX Graph (GPU-accelerated)
1. Create VFX Graph asset
2. Add Initialize Context (spawn rate, lifetime, velocity)
3. Add Update Context (gravity, curl noise, drag)
4. Add Output Context (billboard, mesh, or lit mesh)
5. Connect to scene
6. Tune with VFX Graph window (timeline preview)
```

```hlsl
// VFX Graph HLSL: custom particle behavior
// Curl noise for organic fire motion
float3 curlNoise(float3 p) {
    const float e = 0.1;
    float3 dx = float3(e, 0.0, 0.0);
    float3 dy = float3(0.0, e, 0.0);
    float3 dz = float3(0.0, 0.0, e);

    float3 x0 = curl(p - dx); float3 x1 = curl(p + dx);
    float3 y0 = curl(p - dy); float3 y1 = curl(p + dy);
    float3 z0 = curl(p - dz); float3 z1 = curl(p + dz);

    float x = (y1.z - y0.z) - (z1.y - z0.y);
    float y = (z1.x - z0.x) - (x1.z - x0.z);
    float z = (x1.y - x0.y) - (y1.x - y0.x);

    return normalize(float3(x, y, z)) / (2.0 * e);
}
```

## Key Patterns
```csharp
// Unity: GPU particle system setup
// VFX Graph: Create → Visual Effect Graph
// In Inspector: Set spawn rate, lifetime, size over lifetime

// Trail effect for sword swing
VFXGraph trailEffect;
void OnSwing()
{
    var vfx = Instantiate(trailEffect).GetComponent<VisualEffect>();
    vfx.SetVector3("StartPos", swordStart.position);
    vfx.SetVector3("EndPos", swordEnd.position);
    vfx.SetFloat("Speed", 1.0f);
    vfx.SendEvent("OnPlay");
    Destroy(vfx.gameObject, 2.0f);
}
```

```javascript
// Three.js: lightweight particle system
class ParticleSystem {
  constructor(scene, count = 1000) {
    const geometry = new THREE.BufferGeometry();
    const positions = new Float32Array(count * 3);
    const velocities = new Float32Array(count * 3);
    const lifetimes = new Float32Array(count);

    for (let i = 0; i < count; i++) {
      positions[i*3] = (Math.random() - 0.5) * 0.1;
      positions[i*3+1] = Math.random() * 0.5;
      positions[i*3+2] = (Math.random() - 0.5) * 0.1;
      velocities[i*3] = (Math.random() - 0.5) * 0.02;
      velocities[i*3+1] = 0.02 + Math.random() * 0.03;
      lifetimes[i] = Math.random();
    }

    geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    const material = new THREE.PointsMaterial({
      color: 0xff6600,
      size: 0.05,
      transparent: true,
      blending: THREE.AdditiveBlending,
    });

    this.mesh = new THREE.Points(geometry, material);
    this.velocities = velocities;
    this.lifetimes = lifetimes;
    this.count = count;
    scene.add(this.mesh);
  }

  update(dt) {
    const pos = this.mesh.geometry.attributes.position.array;
    for (let i = 0; i < this.count; i++) {
      this.lifetimes[i] -= dt * 0.5;
      if (this.lifetimes[i] <= 0) {
        pos[i*3] = (Math.random() - 0.5) * 0.1;
        pos[i*3+1] = 0;
        pos[i*3+2] = (Math.random() - 0.5) * 0.1;
        this.lifetimes[i] = 1.0;
      }
      pos[i*3] += this.velocities[i*3] * dt;
      pos[i*3+1] += this.velocities[i*3+1] * dt;
      pos[i*3+2] += this.velocities[i*3+2] * dt;
    }
    this.mesh.geometry.attributes.position.needsUpdate = true;
  }
}
```

## Pitfalls
- Overdraw: thousands of transparent particles kill fill rate — use fewer, larger particles
- Sorting: transparent particles need depth sorting — expensive, use additive blending
- GPU particles need compute shader support — fallback to CPU on older devices
- Particle counts in stats are misleading — check actual triangle/fragment counts
- Sub-emitters multiply particle count exponentially — limit nesting depth
- Screen-space vs world-space: screen particles don't respect scene lighting

## Verification
- Profile particle count and draw calls per effect
- Test on lowest target device for performance
- Visual QA: check particle motion matches reference (fire rises, debris falls)
- Ensure particles don't clip through geometry (collision testing)
- Memory check: large particle pools consume GPU memory — monitor allocations