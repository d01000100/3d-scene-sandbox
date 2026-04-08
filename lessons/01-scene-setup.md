# Lesson 1: Scene, Camera, Renderer

## The Three Pillars

Every Three.js application needs exactly three objects to show anything on screen:

```
Scene + Camera + Renderer = pixels on screen
```

### Scene
The scene is a **container** (technically a scene graph — a tree of objects). You `.add()` things to it: meshes, lights, groups, etc. It has no visual representation of its own.

```js
const scene = new THREE.Scene();
scene.background = new THREE.Color('#1a1a1a');
```

### Camera
The camera defines **what part of the scene** is visible. Three.js has two main camera types:

| Camera | Behavior | Use case |
|--------|----------|----------|
| `PerspectiveCamera` | Objects shrink with distance (like real life) | Most 3D scenes |
| `OrthographicCamera` | No perspective — objects stay the same size regardless of distance | 2D games, UI overlays, architectural views |

We use `PerspectiveCamera` because we want the 3D blocks to have depth.

**Key parameters:**
- **FOV (60):** The vertical angle the camera can see. Lower = more "zoomed in", higher = more fisheye.
- **Aspect ratio:** Must match the canvas or things look stretched.
- **Near/Far planes:** Objects outside this range are clipped (invisible). Keep near > 0 (never 0).

### Renderer
The renderer takes the scene + camera and **draws actual pixels** using WebGL (GPU-accelerated).

```js
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);   // adds the <canvas>
```

**Important settings:**
- `antialias: true` — smooths jagged edges. Slight performance cost, but looks much better.
- `setPixelRatio` — matches Retina/HiDPI displays. Without it, your scene looks blurry on MacBooks.

## Coordinate System

Three.js uses a **right-handed coordinate system:**

```
        Y (up)
        |
        |
        +------ X (right)
       /
      Z (toward you)
```

- Positive X = right
- Positive Y = up
- Positive Z = toward the viewer

When we set `camera.position.set(5, 8, 12)`, we place it 5 units right, 8 units up, and 12 units toward us. Then `camera.lookAt(0, 0, 0)` points it at the origin.

## Meshes = Geometry + Material

A **Mesh** is the basic visible object:

```js
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);
```

- **Geometry** defines the shape (vertices, triangles). `BoxGeometry(w, h, d)` is a rectangular box.
- **Material** defines how the surface looks. `MeshBasicMaterial` is the simplest — flat color, no lighting needed.

We chose `MeshBasicMaterial` because the Opus reference uses **flat, unlit colors**. No shadows, no reflections — just bold blocks of color.

## The Render Loop

A single `renderer.render(scene, camera)` would draw one frame and stop. For animation, we loop:

```js
function animate() {
  requestAnimationFrame(animate);  // schedule next frame
  renderer.render(scene, camera);  // draw this frame
}
animate();  // start the loop
```

`requestAnimationFrame` syncs with the monitor's refresh rate (usually 60fps). It pauses when the tab is hidden, saving battery.

## What You Should See

A single red box on a dark background, viewed from above-right. It's static — no animation yet. That comes in Step 3.

## Next Step
→ [Lesson 2: Generative Grid](./02-generative-grid.md) — Fill the scene with a grid of colorful blocks.
