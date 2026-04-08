# Lesson 6: Camera Controls — OrbitControls & Mouse Button Remapping

## The Problem

So far, the camera is frozen where we placed it (`camera.position.set(8, 12, 18)`). You can't rotate around the scene to see the blocks from different angles — which is half the appeal of a 3D scene.

## Three.js Addons

Three.js ships with a large collection of **addons** (formerly called "examples") — pre-built utilities for common tasks. They're not part of the core library because not every project needs them.

To use them, we expand the import map:

```js
{
  "imports": {
    "three": "https://unpkg.com/three@0.170.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.170.0/examples/jsm/"
  }
}
```

The trailing `/` in `"three/addons/"` tells the browser: "any import starting with `three/addons/` should be fetched from that URL prefix." So:

```js
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
// Actually fetches from: https://unpkg.com/three@0.170.0/examples/jsm/controls/OrbitControls.js
```

Other popular addons include `GLTFLoader` (load 3D models), `EffectComposer` (post-processing), `FontLoader` (3D text), and many more.

## OrbitControls

`OrbitControls` handles orbiting, panning, and zooming:

```js
const controls = new OrbitControls(camera, renderer.domElement);
```

It takes two arguments:
1. **camera** — the camera it will manipulate
2. **domElement** — the HTML element to listen for mouse/touch events on

Once created, it takes over camera movement. You no longer call `camera.lookAt()` manually — the controls manage that based on their `.target` property.

### Default Mouse Mapping

| Button | Default action |
|--------|---------------|
| Left click + drag | Orbit (rotate around target) |
| Right click + drag | Pan (slide sideways) |
| Scroll wheel | Zoom (dolly in/out) |

### Our Remapping

We need left-click free for raycasting (clicking blocks), so we remap:

```js
controls.mouseButtons = {
  LEFT:   null,                // ← disabled, free for raycasting
  MIDDLE: THREE.MOUSE.PAN,
  RIGHT:  THREE.MOUSE.ROTATE, // ← orbit is now right-click
};
```

`THREE.MOUSE` is an enum: `{ LEFT: 0, MIDDLE: 1, RIGHT: 2, ROTATE: 0, DOLLY: 1, PAN: 2 }`. The `.mouseButtons` object maps physical buttons to logical actions.

## Damping (Inertia)

```js
controls.enableDamping = true;
controls.dampingFactor = 0.08;
```

With damping, the camera **drifts** after you release the mouse instead of stopping instantly. This feels much more natural and polished.

**Critical requirement:** When damping is enabled, you **must** call `controls.update()` in every animation frame. Without it, the camera freezes because the damping math never runs:

```js
function animate() {
  requestAnimationFrame(animate);
  controls.update();  // ← must be called every frame for damping
  renderer.render(scene, camera);
}
```

## Safety Limits

```js
controls.minDistance = 5;
controls.maxDistance = 60;
controls.maxPolarAngle = Math.PI / 2.1;
```

| Property | What it prevents |
|----------|-----------------|
| `minDistance` | Camera zooming *through* objects |
| `maxDistance` | Camera zooming so far out everything becomes a dot |
| `maxPolarAngle` | Camera going below the ground plane (flipping under the scene) |

`maxPolarAngle` is measured from the positive Y axis (straight up). `Math.PI / 2` = exactly horizontal. We use `Math.PI / 2.1` to stop just *above* horizontal, preventing the "underground" view.

## Context Menu Suppression

Right-click normally opens the browser's context menu ("Copy / Paste / Inspect Element"). We suppress it on the canvas:

```js
renderer.domElement.addEventListener('contextmenu', (e) => e.preventDefault());
```

This only affects the canvas — right-clicking elsewhere on the page (like the config panel) still works normally.

## Filtering Raycaster by Button

With right-click now taken by OrbitControls, we must make sure raycasting only fires on **left-click**:

```js
function onPointerDown(event) {
  if (event.button !== 0) return;  // 0 = left, 1 = middle, 2 = right
  // ... raycasting code ...
}
```

`event.button` is a standard DOM property available on all pointer/mouse events.

## The Target Point

```js
controls.target.set(0, 0, 0);
```

The target is the point in 3D space that the camera orbits **around** and **looks at**. Think of it as an invisible pivot. Moving the target shifts the center of rotation.

You could set it to, say, `(0, 3, 0)` to orbit around a point 3 units above ground — which would shift the camera angle upward.

## What You Should See

- **Left-click** a block → it spins (raycasting, unchanged)
- **Right-click + drag** → camera orbits around the scene
- **Scroll wheel** → zoom in/out
- **Middle-click + drag** → pan sideways
- Release after dragging → camera drifts smoothly (damping)
- Camera can't go underground or zoom infinitely

## Next Steps

Other interesting `OrbitControls` options to explore:
- `controls.autoRotate = true` — camera orbits automatically (set `autoRotateSpeed`)
- `controls.enablePan = false` — disable panning entirely
- `controls.minPolarAngle` — prevent looking straight down
- `controls.touches` — remap touch gestures (one-finger, two-finger, etc.)
