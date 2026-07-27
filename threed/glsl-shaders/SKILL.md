---
name: glsl-shaders
description: - Performance-critical per-pixel calculations
category: threed
tags: [glsl-shaders]
---

## When to Use
- Custom visual effects: water, fire, dissolves, toon shading
- Performance-critical per-pixel calculations
- Post-processing effects (bloom, blur, color grading)
- Particle system custom rendering
- Any visual effect too complex for standard material properties

## Core Concepts
- Vertex shader: transforms vertices from model → clip space
- Fragment shader: determines final color of each pixel
- Uniforms: constant values passed from CPU to GPU (matrices, time, textures)
- Varyings: interpolated data passed from vertex → fragment shader
- GLSL types: vec2/3/4, mat4, sampler2D, float, int
- Coordinate spaces: local → world → view → clip → screen

## Workflow
1. Start with a minimal vertex/fragment shader pair
2. Declare uniforms for parameters you'll animate
3. Implement vertex transformation (pass UVs and normals as varyings)
4. Write fragment shader starting with solid color
5. Add texture sampling, then lighting calculations
6. Integrate with Three.js ShaderMaterial or RawShaderMaterial
7. Test on multiple GPUs (integrated vs discrete)

## Key Patterns
```glsl
// Vertex shader
uniform mat4 modelViewMatrix;
uniform mat4 projectionMatrix;
uniform float uTime;
attribute vec3 position;
attribute vec2 uv;
varying vec2 vUv;

void main() {
  vUv = uv;
  vec3 pos = position;
  pos.y += sin(pos.x * 3.0 + uTime) * 0.1; // wave displacement
  gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
}

// Fragment shader
precision highp float;
varying vec2 vUv;
uniform float uTime;
uniform sampler2D uTexture;

void main() {
  vec4 texColor = texture2D(uTexture, vUv + vec2(0.0, uTime * 0.05));
  float vignette = smoothstep(0.8, 0.4, length(vUv - 0.5));
  gl_FragColor = vec4(texColor.rgb * vignette, 1.0);
}
```

```javascript
// Three.js integration
const material = new THREE.ShaderMaterial({
  vertexShader: vertexSrc,
  fragmentShader: fragmentSrc,
  uniforms: {
    uTime: { value: 0 },
    uTexture: { value: new THREE.TextureLoader().load('noise.png') },
  },
  transparent: true,
  side: THREE.DoubleSide,
});

// In animation loop
material.uniforms.uTime.value = clock.elapsedTime;
```

```glsl
// Post-processing fullscreen quad fragment shader
uniform sampler2D tDiffuse;
uniform vec2 uResolution;
varying vec2 vUv;

void main() {
  vec4 color = texture2D(tDiffuse, vUv);
  // Chromatic aberration
  float offset = 0.003;
  float r = texture2D(tDiffuse, vUv + vec2(offset, 0.0)).r;
  float b = texture2D(tDiffuse, vUv - vec2(offset, 0.0)).b;
  gl_FragColor = vec4(r, color.g, b, 1.0);
}
```

## Pitfalls
- `precision mediump float` on mobile causes banding on large coordinates
- Missing `gl_Position` assignment = invisible geometry
- Texture sampling outside [0,1] wraps by default — set wrap mode explicitly
- Uniform names must match exactly between JS and GLSL — no validation
- Shader compilation errors are runtime-only — check `gl.getShaderInfoLog()`

## Verification
- Compile shaders and check: `gl.getShaderInfoLog(fragmentShader)`
- Use Spector.js browser extension to inspect WebGL calls
- Test on mobile (Mali, Adreno GPUs have different precision behavior)
- Profile with GPU timing queries for bottleneck detection
- Visual regression: screenshot shader output at key frames, compare