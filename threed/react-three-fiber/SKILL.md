---
name: react-three-fiber
description: - Teams familiar with React ecosystem (hooks, state, components)
category: threed
tags: [react-three-fiber]
---

## When to Use
- React-based 3D applications and interactive product viewers
- Teams familiar with React ecosystem (hooks, state, components)
- Declarative 3D scene construction
- Integration with React UI (overlays, controls, panels)
- Rapid prototyping of 3D experiences with hot-reload

## Core Concepts
- R3F is a React renderer for Three.js: JSX describes 3D scenes
- `<Canvas>` wraps the Three.js renderer, camera, and render loop
- All Three.js objects available as JSX elements: `<mesh>`, `<boxGeometry>`
- Hooks: useFrame (animation loop), useThree (renderer/camera/scene)
- drei: helper components (OrbitControls, Environment, useGLTF, Text)
- drei's `<Preload>` manages asset loading with Suspense boundaries

## Workflow
```bash
npm create vite@latest my-r3f-app -- --template react-ts
cd my-r3f-app
npm install three @react-three/fiber @react-three/drei @types/three
```

```tsx
// App.tsx
import { Canvas } from '@react-three/fiber';
import { OrbitControls, Environment, ContactShadows } from '@react-three/drei';
import { Suspense } from 'react';
import { Scene } from './Scene';

function App() {
  return (
    <Canvas shadows camera={{ position: [0, 2, 5], fov: 50 }} dpr={[1, 2]}>
      <Suspense fallback={<Loading />}>
        <Scene />
        <Environment preset="studio" />
        <ContactShadows position={[0, -0.5, 0]} opacity={0.4} scale={10} blur={2} />
        <OrbitControls makeDefault />
      </Suspense>
    </Canvas>
  );
}

// Scene.tsx
import { useFrame } from '@react-three/fiber';
import { useGLTF } from '@react-three/drei';
import { useRef } from 'react';

export function Scene() {
  const { scene } = useGLTF('/models/product.glb');
  const meshRef = useRef<THREE.Mesh>(null);

  useFrame((state) => {
    if (meshRef.current) {
      meshRef.current.rotation.y = state.clock.elapsedTime * 0.2;
    }
  });

  return <primitive ref={meshRef} object={scene} scale={1.5} castShadow />;
}
```

## Key Patterns
```
# State-driven material changes
import { useState } from 'react';
import * as THREE from 'three';

function Product() {
  const [color, setColor] = useState('#ff6b35');
  const [material, setMaterial] = useState('matte');

  return (
    <mesh castShadow>
      <sphereGeometry args={[1, 64, 64]} />
      <meshStandardMaterial
        color={color}
        roughness={material === 'matte' ? 0.8 : 0.1}
        metalness={material === 'metallic' ? 0.9 : 0}
      />
    </mesh>
  );
}

# Responsive 3D with useThree
function ResponsiveScene() {
  const { size, camera } = useThree();
  useEffect(() => {
    const aspect = size.width / size.height;
    (camera as THREE.PerspectiveCamera).aspect = aspect;
    camera.updateProjectionMatrix();
  }, [size]);
  return null;
}

# Performance: React.memo for static objects
const StaticModel = React.memo(function StaticModel({ url }: { url: string }) {
  const { scene } = useGLTF(url);
  return <primitive object={scene} />;
});
```

## Pitfalls
- JSX objects don't auto-dispose on unmount — use `useEffect` cleanup or `dispose` prop
- `<primitive>` with GLTF scene must be manually disposed: `scene.traverse(c => c.dispose?.())`
- Re-renders cause re-creation of Three.js objects if not memoized
- drei's `<Text>` uses troika — heavy on first load, precompute with font prop
- useFrame callbacks accumulate if components remount — track cleanup

## Verification
- Use `@react-three/fiber` dev overlay for frame time display
- Check React DevTools for unnecessary re-renders in Canvas
- Profile with Chrome Performance tab — look for GC spikes
- Test Suspense boundaries show loading states
- Verify with `renderer.info.render` for draw call counts