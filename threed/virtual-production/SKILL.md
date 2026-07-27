---
name: virtual-production
description: - Virtual sets for live broadcasts and events
category: threed
tags: [virtual-production]
---

## When to Use
- Real-time filming with LED volumes (ICVFX) for film/TV
- Virtual sets for live broadcasts and events
- Previsualization and real-time previz for film production
- Real-time compositing with camera tracking
- Mixed reality performances with live actors and virtual environments

## Core Concepts
- LED Volume: curved LED wall displays real-time 3D backgrounds
- ICVFX (In-Camera Visual Effects): captures final pixels in-camera
- Camera tracking: real-time position/rotation fed to render engine
- Genlock: synchronizing camera frame rate with LED refresh rate
- Color science: ACES workflow, LED color calibration, white balance
- Disguise / Pixotope: media servers for LED wall content management
- nDisplay: UE5 multi-display rendering for LED walls
- Frame interpolation: reducing LED flicker and moiré patterns

## Workflow
```
# Unreal Engine virtual production setup
1. Build LED stage: panels, curved wall, ceiling
2. Calibrate cameras: lens distortion, intrinsics, extrinsics
3. Set up camera tracking (OptiTrack, Mo-Sly, Stype)
4. Configure nDisplay cluster for LED wall rendering
5. Build virtual set in UE5 with Nanite/Lumen
6. Configure frustum rendering (inner frustum for camera, outer for ambient)
7. Color calibrate: match LED output to camera exposure
8. Rehearse: test lighting, reflections, parallax
9. Shoot: monitor with live composite overlay
```

nDisplay configuration:
```json
{
  "cluster": {
    "nodes": [
      {"id": "node_0", "ip": "192.168.1.10", "role": "primary"},
      {"id": "node_1", "ip": "192.168.1.11", "role": "backup"},
      {"id": "node_2", "ip": "192.168.1.12", "role": "node"}
    ]
  },
  "display": {
    "render_surfaces": [
      {
        "id": "inner_frustum",
        "resolution": [3840, 2160],
        "viewport": {"x": 0, "y": 0, "width": 1920, "height": 1080}
      }
    ]
  },
  "camera": {
    "tracking": {
      "provider": "OptiTrack",
      "interval": 1000000
    }
  }
}
```

```python
# Python: camera tracking data relay
import socket
import json
import struct

class CameraTrackerRelay:
    # Receives tracking data from OptiTrack Motive and sends to UE5
    
    def __init__(self, motive_host, ue5_host, ue5_port=11111):
        self.motive = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.motive.bind((motive_host, 3964))
        self.ue5 = (ue5_host, ue5_port)
    
    def process_frame(self):
        data, addr = self.motive.recvfrom(1024)
        # Parse Motive data (rigid body transform)
        rigid_body = struct.unpack('fffffff', data[:28])
        x, y, z, qx, qy, qz, qw = rigid_body
        
        # Convert to UE5 coordinate system (Z-up)
        transform = {
            "position": {"x": x * 100, "y": -z * 100, "z": y * 100},
            "rotation": {"x": qx, "y": -qz, "z": qy, "w": qw},
        }
        
        # Send to UE5 via LiveLink
        self.send_to_ue5(json.dumps(transform).encode())
    
    def send_to_ue5(self, data):
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect(self.ue5)
        sock.send(data)
        sock.close()
```

## Key Patterns
```
# LED wall brightness calibration
# Measure with colorimeter at center and edges
# Target: 100 nits center, 80 nits edges (uniformity)
# Adjust via Disguise color correction per panel

# Frustum rendering setup (UE5)
# Inner frustum: matches camera lens (85% of LED wall)
# Outer frustum: ambient lighting (15% surrounding)
# Parallax: camera position drives frustum movement
# Update rate: 60fps minimum, 120fps for high-speed cameras

# Camera settings for ICVFX
Shutter speed:  1/48 (24fps) or 1/60 (30fps)
ISO:            800-1600 (adjust for LED brightness)
ND filter:      0.6-1.2 (control LED exposure)
White balance:  Match LED color temperature (5600K typical)
Genlock:        Free-run with genlock sync to LED refresh
```

## Pitfalls
- Moiré patterns from camera sensor + LED pixel grid — use defocus or pixel offset
- Color shift: LED panels have different gamut than camera — ACES conversion needed
- Latency: camera tracking delay causes parallax errors — keep < 1 frame
- LED refresh rate vs camera shutter: 30fps needs ≥120Hz LED to avoid banding
- Thermal: LED walls generate heat — cooling system required for extended shoots
- Budget: LED stage setup costs $100K+/day — validate ROI vs traditional VFX

## Verification
- Camera tracking accuracy: <1mm position error, <0.1° rotation error
- Color calibration: Delta E < 2 across LED wall surface
- Latency test: measure tracking-to-display delay (<16ms for 60fps)
- Visual QA: compare in-camera composite vs final VFX plate
- Stress test: 8-hour continuous shoot for thermal and stability