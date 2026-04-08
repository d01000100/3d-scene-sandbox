# Lesson 8: Toggling Between Perspective and Orthographic Cameras

## The Two Camera Types

Three.js has two main cameras that see the world very differently:

### PerspectiveCamera
Objects **shrink with distance**, just like your eyes. Defined by a field of view (FOV) angle and an aspect ratio. This is what most 3D scenes use.

```
      /  far plane  \
     / ------------- \
    / /             \ \
   / /               \ \
  [camera]            [near plane]
```

### OrthographicCamera
**No perspective** — a box 10 meters away looks the same size as one 1 meter away. Defined by a rectangular viewing box (`left`, `right`, `top`, `bottom`). This gives a flat, isometric, architectural look.

```
  |  far plane      |
  | --------------- |
  | |             | |
  | |             | |
  [camera]        [near plane]
  (parallel rays — no convergence)
```

## Setting Up the Orthographic Camera

```js
const ORTHO_SIZE = 15;
const aspect = window.innerWidth / window.innerHeight;

const orthoCamera = new THREE.OrthographicCamera(
  -ORTHO_SIZE * aspect,   // left boundary
   ORTHO_SIZE * aspect,   // right boundary
   ORTHO_SIZE,            // top boundary
  -ORTHO_SIZE,            // bottom boundary
  0.1,                    // near clipping plane
  1000,                   // far clipping plane
);
```

**`ORTHO_SIZE`** controls how much of the scene is visible. A larger value zooms out, a smaller value zooms in. We multiply by `aspect` on the horizontal axis to prevent stretching.

## The `activeCamera` Pattern

Instead of a single `const camera`, we have:

```js
const perspCamera = new THREE.PerspectiveCamera(...);
const orthoCamera = new THREE.OrthographicCamera(...);
let activeCamera = perspCamera;  // mutable pointer
```

Everything that previously referenced `camera` now uses `activeCamera`:
- `renderer.render(scene, activeCamera)`
- `raycaster.setFromCamera(pointer, activeCamera)`
- Resize handler updates both cameras
- Reset button moves `activeCamera`

## Why OrbitControls Must Be Recreated

OrbitControls binds to a specific camera in its constructor and stores internal state tied to that camera's projection math. There's no public API to swap the camera. So on toggle:

```js
// 1. Copy position so the view doesn't jump
newCamera.position.copy(activeCamera.position);

// 2. Save the orbit target
const currentTarget = controls.target.clone();

// 3. Swap
activeCamera = newCamera;

// 4. Recreate controls for the new camera
controls = createControls(activeCamera, currentTarget);
```

The `createControls` factory handles disposal of old controls, creation of new ones, and restoring all settings (damping, mouse buttons, limits).

### The Factory Function

```js
function createControls(cam, target) {
  if (controls) controls.dispose();  // clean up old listeners

  const ctrl = new OrbitControls(cam, renderer.domElement);
  ctrl.mouseButtons = { LEFT: null, MIDDLE: THREE.MOUSE.PAN, RIGHT: THREE.MOUSE.ROTATE };

  if (target) ctrl.target.copy(target);

  // Different zoom limits per camera type
  if (cam.isPerspectiveCamera) {
    ctrl.minDistance = 5;
    ctrl.maxDistance = 60;
  } else {
    ctrl.minZoom = 0.3;
    ctrl.maxZoom = 4;
  }

  return ctrl;
}
```

**Key detail:** `controls.dispose()` is critical. Without it, the old controls would keep listening for mouse events on the canvas, creating ghost interactions.

## Zoom Works Differently Per Camera

| Camera type | Zoom mechanism | Limit properties |
|------------|---------------|-----------------|
| Perspective | Moves camera closer/farther (dolly) | `minDistance`, `maxDistance` |
| Orthographic | Scales the viewing box (changes `.zoom`) | `minZoom`, `maxZoom` |

OrbitControls handles this automatically — you just set the right limit properties.

## Resize Handler — Both Cameras

We update **both** cameras on resize, not just the active one. This ensures the inactive camera is ready if the user toggles mid-resize:

```js
window.addEventListener("resize", () => {
  const w = window.innerWidth;
  const h = window.innerHeight;
  const newAspect = w / h;

  renderer.setSize(w, h);

  // Perspective: update aspect ratio
  perspCamera.aspect = newAspect;
  perspCamera.updateProjectionMatrix();

  // Orthographic: recalculate viewing box boundaries
  orthoCamera.left   = -ORTHO_SIZE * newAspect;
  orthoCamera.right  =  ORTHO_SIZE * newAspect;
  orthoCamera.top    =  ORTHO_SIZE;
  orthoCamera.bottom = -ORTHO_SIZE;
  orthoCamera.updateProjectionMatrix();
});
```

## `.isPerspectiveCamera` — Type Checking in Three.js

Three.js objects have boolean type flags:
- `camera.isPerspectiveCamera` → `true` for PerspectiveCamera
- `camera.isOrthographicCamera` → `true` for OrthographicCamera
- `object.isMesh` → `true` for Mesh

This is preferred over `instanceof` checks because it works across module boundaries and is more readable.

## What You Should See

- Click **"Switch to Orthographic"** → the scene flattens into an isometric view
- All controls still work (orbit, pan, zoom, click blocks)
- Click **"Switch to Perspective"** → depth returns
- The view doesn't jump on toggle — position and target are synced
- Window resize works correctly in both modes
