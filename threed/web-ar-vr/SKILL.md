---
name: web-ar-vr
description: - WebVR/WebXR immersive experiences in browser
category: threed
tags: [web-ar-vr]
---

## When to Use
- Web-based AR experiences (8th Wall, MindAR, AR.js)
- WebVR/WebXR immersive experiences in browser
- Product visualization in AR (try before you buy)
- Virtual tours and architectural visualization
- Cross-platform VR without native app development

## Core Concepts
- WebXR API: navigator.xr for VR/AR session management
- 8th Wall: commercial SLAM-based markerless AR
- AR.js: open-source marker-based AR (Hiro marker, custom markers)
- MR/VR hand tracking and controller input
- Camera passthrough for AR (WebXR hit-test, anchors)
- Performance budget: 60fps on mobile, 72fps+ on VR headsets
- HTTPS required: WebXR only works on secure origins

## Workflow
```bash
# Project setup (AR.js with A-Frame)
npm init -y
npm install aframe ar.js
# Or for 8th Wall: use their Niantic Studio platform
```

```html
<!-- AR.js marker-based AR example -->
<a-scene embedded arjs="sourceType: webcam; debugUIEnabled: false;">
  <a-marker preset="hiro">
    <a-box position="0 0.5 0" material="color: red;" scale="0.5 0.5 0.5">
      <a-animation attribute="rotation" to="0 360 0"
        dur="3000" repeat="indefinite"/>
    </a-box>
  </a-marker>
  <a-entity camera></a-entity>
</a-scene>

<!-- WebXR VR experience -->
<script>
navigator.xr.requestSession('immersive-vr', {
  optionalFeatures: ['local-floor', 'hand-tracking']
}).then(session => {
  renderer.xr.enabled = true;
  renderer.xr.setSession(session);

  const controller = renderer.xr.getController(0);
  controller.addEventListener('selectstart', onSelect);
  scene.add(controller);
});
</script>
```

```javascript
// 8th Wall: Place 3D model on surface via hit-test
import { XR8, XRExtras } from '@nic3/8thwall';

const run = () => {
  XR8.run({
    canvas: document.getElementById('camerafeed'),
    pipelines: [
      XRExtras.AlmostThere.pipeline(),
      XRExtras.Loading.pipeline(),
      XRExtras.XrController.pipeline(),
    ],
  });
};

// Hit-test for surface placement
const raycaster = new THREE.Raycaster();
const hitTestSource = await session.requestHitTestSource({
  space: viewerSpace,
});
const hit = hitTestSource.getResults();
if (hit.length > 0) {
  const pose = hit[0].getPose(referenceSpace);
  model.position.set(
    pose.transform.position.x,
    pose.transform.position.y,
    pose.transform.position.z
  );
}
```

## Key Patterns
```
# AR.js custom marker generation
# Print marker from https://jeromeetienne.github.io/AR.js/three.js/examples/marker-training/examples/generator.html
# Save as .patt file and reference in <a-marker type="pattern" url="marker.patt">

# WebXR hand tracking
session.addEventListener('handfound', (e) => {
  const hand = e.hand;
  const indexTip = hand.get('index-finger-tip');
  if (indexTip) {
    controller.position.copy(indexTip.position);
  }
});

# Performance: reduce draw calls for AR on mobile
# Target: < 50 draw calls, < 100K triangles
# Use GLB format with Draco compression
# Max texture size 1024x1024 for mobile AR
```

## Pitfalls
- HTTP blocked: WebXR requires HTTPS (use ngrok or self-signed for dev)
- iOS Safari has limited WebXR support — use WebARonARKit or 8th Wall
- Drift: SLAM tracking degrades in featureless environments (white walls)
- Battery drain: continuous camera + GPU at 60fps
- Orientation lock: AR needs fullscreen + locked portrait on mobile
- CORS errors loading models: serve from same origin or set headers

## Verification
- Test on iOS Safari and Android Chrome separately (different APIs)
- Use Chrome DevTools Remote Debugging for mobile AR
- Profile GPU frames on mobile: target 16ms per frame
- Test tracking quality: walk around marker/surface at different distances
- Validate WebXR session: `navigator.xr.isSessionSupported('immersive-vr')`