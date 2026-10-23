# XR Training Labs | NFCC Ladder Drill Simulator (WebXR)

Single-file WebXR training application demonstrating UK Fire & Rescue Service National Fire Chiefs Council (NFCC) standard 13.5m / 10.5m extension ladder drills. Engineered for standalone execution on Meta Quest 3 / 3S headsets and desktop browsers.

## Core Architecture

* **Zero-Build Single-File Execution**: Contained within a single `index.html` file utilizing A-Frame 1.6.0, Three.js, and Tailwind CSS via CDN.
* **Procedural Fallback Geometry**: Boots instantaneously with zero external asset dependencies using procedurally generated meshes for the ladder sections, 4-storey drill tower, and Type-B water ladder appliance.
* **Dynamic GLB/GLTF Asset Swapper (`swappable-asset`)**: Hot-swaps procedural primitives with high-fidelity `.glb` / `.gltf` models at runtime via local file upload (`URL.createObjectURL`), remote URI, or static manifest configuration.
* **Automatic Bounding-Box Normalization**: Computes `THREE.Box3` bounds upon model load, scales meshes proportionally to NFCC metric dimensions, and aligns the bottom-center pivot to `(0, 0, 0)` to preserve hinge kinematics and extension vectors.
* **Meta Quest 3 WebXR Optimization**: Configured with hardware foveated rendering (`foveationLevel: 2`), high refresh rate targeting (`highRefreshRate: true`), physically correct lighting, and Google Draco mesh decompression.
* **Dual-Interface Control**:
  * **2D Glassmorphism HUD**: Desktop/tablet instructor overlay for telemetry monitoring, stage triggering, and runtime GLB injection.
  * **3D Diegetic Console**: In-world spatial UI operable via Quest 3 Touch Plus laser raycasters or hand-tracking pinch gestures.

---

## NFCC Drill State Machine

The simulation enforces a 5-stage operational sequence aligned with UK Foundation Training guidelines:

| Stage | Command | Mechanical State | Telemetry / Spatial Action |
| :--- | :--- | :--- | :--- |
| **1. Ground** | `"GROUND LADDER"` | Heel at `Z = -1.8m` (2.2m from tower face), `2.0°` pitch, housed (`5.5m`). | Validates 4:1 footprint zone and overhead hazard clearance. |
| **2. Pitch** | `"PITCH LADDER"` | Heel pivot rotates to `75.0°` working angle (`-15°` local X). | Simulates under-running while No. 1 braces the heel blocks. |
| **3. Extend** | `"EXTEND... WELL!"` | Fly carriage translates `+3.1m` along local Y axis (`8.4m` head height). | Positions head rounds 3 full rungs above the `6.8m` 2nd-floor sill; locks pawls. |
| **4. Climb** | `"CLIMB & LEG LOCK"` | Pitch `75.0°`, Extension `8.4m`. | Translates camera rig to `(0, 5.4, -3.1)` to simulate climber POV at the target window sill. |
| **5. Make Up** | `"MAKE UP"` | Retracts fly to `0.15m`, pitches down to `2.0°`. | Returns camera rig to yard POV `(0, 0, 4.5)` and houses equipment. |

---

## GLB Asset Integration & Specifications

### 1. Runtime Swapping (UI Drawer)
Open the **GLB Asset Swapper** drawer in the 2D HUD to upload local `.glb` files or bind remote URLs for each of the four scene slots. Blob URLs are automatically revoked upon replacement or reset to prevent WebGL memory leaks on mobile chipsets.

### 2. Static Manifest Configuration
To hardcode production 3D models for deployment, populate `window.XR_ASSET_MANIFEST` inside `index.html`:

```javascript
window.XR_ASSET_MANIFEST = {
  'ladder-base': {
    url: './assets/nfcc_135_base.glb',
    targetSize: { x: 0.55, y: 5.5, z: 0.22 },
    autoScale: true
  },
  'ladder-fly': {
    url: './assets/nfcc_135_fly.glb',
    targetSize: { x: 0.46, y: 5.5, z: 0.18 },
    autoScale: true
  },
  'drill-tower': {
    url: './assets/fsc_drill_tower.glb',
    targetSize: { x: 4.5, y: 12.5, z: 4.5 },
    autoScale: true
  },
  'fire-appliance': {
    url: './assets/scania_wtl_appliance.glb',
    targetSize: { x: 2.55, y: 3.3, z: 8.2 },
    autoScale: true
  }
};
