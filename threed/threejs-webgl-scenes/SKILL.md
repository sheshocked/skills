---
name: threejs-webgl-scenes
description: - Data visualization with 3D charts, maps, and dashboards
category: threed
tags: [threejs-webgl-scenes]
---

## When to Use
- Building interactive 3D web experiences and product configurators
- Data visualization with 3D charts, maps, and dashboards
- Virtual showrooms, architectural walkthroughs
- Browser-based games and interactive demos
- Any web project requiring WebGL rendering without full game engine

## Core Concepts
- Scene graph: Scene → Mesh → Geometry + Material hierarchy
- Camera types: PerspectiveCamera (realistic), OrthographicCamera (flat)
- Render loop: requestAnimationFrame-based continuous rendering
- Lighting: AmbientLight (base), DirectionalLight (sun), PointLight (bulb)
- Raycasting for mouse interaction with 3D objects
- Renderer: WebGLRenderer with antialias, shadows, toneMapping

## Workflow
```bash
# Project setup
npm create vite@latest my-3d-app -- --template vanilla
cd my-3d-app && npm install three @types/three
```

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// Scene setup
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a2e);

const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(0, 2, 5);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.2;
document.body.appendChild(renderer.domElement);

// Controls
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.05;

// Lighting
const ambientLight = new THREE.AmbientLight(0x404040, 0.5);
scene.add(ambientLight);
const dirLight = new THREE.DirectionalLight(0xffffff, 1.0);
dirLight.position.set(5, 10, 7.5);
dirLight.castShadow = true;
dirLight.shadow.mapSize.set(2048, 2048);
scene.add(dirLight);

// Render loop
function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}
animate();

// Handle resize
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
```

## Key Patterns
```
# Load GLTF model
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
const loader = new GLTFLoader();
loader.load('/models/scene.glb', (gltf) => {
  scene.add(gltf.scene);
  gltf.scene.traverse((child) => {
    if (child.isMesh) child.castShadow = true;
  });
});

# Raycasting for click interaction
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();
renderer.domElement.addEventListener('click', (e) => {
  mouse.x = (e.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(e.clientY / window.innerHeight) * 2 + 1;
  raycaster.setFromCamera(mouse, camera);
  const intersects = raycaster.intersectObjects(scene.children);
  if (intersects.length > 0) console.log(intersects[0].object.name);
});

# Performance: InstancedMesh for many identical objects
const geometry = new THREE.BoxGeometry(0.5, 0.5, 0.5);
const material = new THREE.MeshStandardMaterial({ color: 0x44aa88 });
const instancedMesh = new THREE.InstancedMesh(geometry, material, 10000);
const dummy = new THREE.Object3D();
for (let i = 0; i < 10000; i++) {
  dummy.position.set(Math.random()*100, Math.random()*50, Math.random()*100);
  dummy.updateMatrix();
  instancedMesh.setMatrixAt(i, dummy.matrix);
}
scene.add(instancedMesh);
```

## Pitfalls
- Not disposing geometries/materials on unmount causes WebGL memory leaks
- Shadow maps are expensive — only enable for key lights
- render() without requestAnimationFrame runs at monitor refresh, not 60fps fixed
- Z-fighting between coplanar meshes — offset one slightly or use polygonOffset
- Loading large GLTFs blocks main thread — use DRACO compression or split models

## Verification
- Check FPS with `renderer.info.render` and stats.js panel
- Use Chrome DevTools → Performance for frame budget analysis
- Test on mobile: touch controls, lower resolution, memory limits
- Validate GLTF with https://gltf-transform.dev/ CLI tools
- Run Lighthouse for web performance scores