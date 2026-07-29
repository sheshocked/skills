---
name: glsl-custom-shaders
description: Develop vertex/fragment GLSL shaders, mapping texture buffers, coordinate spaces, and custom rendering loops.
category: threed
tags: [glsl, shaders, fragment-shader, vertex-shader, webgl, graphics]
---

# GLSL Custom Shaders Development Masterclass

## When to Use
Use to build custom visual effects, dynamic background gradients, or stylized animations in WebGL (Three.js / React Three Fiber) that run directly on the GPU.

## Prerequisites
- WebGL execution container.

## Workflow
1. Establish a vertex shader mapping vertex positions to screen coordinates.
2. Build a fragment shader computing color values for each pixel.
3. Pass dynamic variables (uniforms, attributes) from JavaScript.

## Key Patterns

### Custom Fragment Shader Code (procedural_gradient.frag)
```glsl
// Custom fragment shader calculating dynamic gradient waves
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform float u_time;

void main() {
    // Normalize coordinates (0.0 to 1.0)
    vec2 st = gl_FragCoord.xy / u_resolution.xy;

    // Generate wave structures
    float wave_red = sin(st.x * 3.0 + u_time) * 0.5 + 0.5;
    float wave_green = cos(st.y * 2.0 - u_time * 1.5) * 0.5 + 0.5;
    float wave_blue = sin((st.x + st.y) * 4.0 + u_time * 0.5) * 0.5 + 0.5;

    // Output final color structure
    gl_FragColor = vec4(wave_red, wave_green, wave_blue, 1.0);
}
```

## Pitfalls
- **Precision mismatches:** WebGL drivers on Android/iOS devices crash if floats lack explicit precision definitions. Always include `precision mediump float;` at the top of fragment files.
- **GPU utilization spikes:** High-frequency trig calculations (like `sin` or `cos` nested inside loops) crash mobile GPUs. Cache calculations or pass pre-computed textures.

## Verification
- Bind shaders to WebGL materials and check for layout compiling warnings.
- Run profile audit tests on target hardware devices.
