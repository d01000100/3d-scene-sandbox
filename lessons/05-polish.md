# Lesson 5: Polish — Palettes, Resize, Pause/Resume

## Palette System

We store preset palettes as a dictionary of color arrays:

```js
const PALETTES = {
  mondrian: ['#e63946', '#f4a261', ...],
  pastel:   ['#ffd6e0', '#ffdfba', ...],
  neon:     ['#ff006e', '#fb5607', ...],
};
```

Switching palettes swaps `CONFIG.palette` and regenerates:

```js
paletteSelect.addEventListener('change', () => {
  CONFIG.palette = PALETTES[paletteSelect.value];
  generateBlocks();
});
```

This reuses the same `generateBlocks()` teardown/rebuild pattern from Step 2.

### Color Swatches

The preview row creates a `<div>` per color with inline `background`:

```js
CONFIG.palette.forEach((color) => {
  const swatch = document.createElement('div');
  swatch.style.background = color;
  container.appendChild(swatch);
});
```

This is a common UI pattern for color palette pickers.

## Window Resize Handling

When the browser window is resized, two things break:
1. The canvas pixel dimensions are wrong (scene is cropped or stretched)
2. The camera's aspect ratio is wrong (objects look squished)

The fix:

```js
window.addEventListener('resize', () => {
  renderer.setSize(window.innerWidth, window.innerHeight);
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
});
```

**Why `updateProjectionMatrix()`?** The camera stores its projection as a 4x4 matrix (math that converts 3D coordinates to 2D screen coordinates). Changing `.aspect` doesn't automatically recompute this matrix — you must call the update method explicitly. This is a common Three.js gotcha.

## Pause / Resume

The simplest approach — a boolean flag checked in the animation loop:

```js
let paused = false;

function animate() {
  requestAnimationFrame(animate);  // always schedule next frame
  if (!paused && blocksGroup) {
    // update rotations...
  }
  renderer.render(scene, camera);  // always render
}
```

**Key detail:** We still call `renderer.render()` even when paused. This ensures the scene remains visible. We only skip the rotation updates.

The toggle button swaps the flag and its own label:

```js
paused = !paused;
btnPause.textContent = paused ? 'Resume' : 'Pause';
```

## Design Decisions Recap

| Decision | Why |
|----------|-----|
| `MeshBasicMaterial` | Flat colors, no lighting needed, fastest material |
| Single HTML file | Zero setup, portable, good for learning |
| CSS `transform` for panel | GPU-accelerated, no layout thrashing |
| `userData` for spin state | Co-located with each mesh, auto-cleans on remove |
| CDN import map | No npm/bundler, works in any modern browser |
| Manual controls (no lil-gui) | Educational — understand the DOM↔3D bridge |

## Where to Go Next

Now that you have the fundamentals, here are paths to explore:

### Easy additions:
- **Hover highlight:** Use raycasting in `pointermove` to change a block's color when hovered
- **Camera orbit:** Add mouse-drag orbit using `OrbitControls` (a Three.js addon)
- **More shapes:** Mix `SphereGeometry`, `ConeGeometry`, `TorusGeometry` into the grid

### Intermediate:
- **Lighting + shadows:** Switch to `MeshStandardMaterial`, add a `DirectionalLight`
- **Post-processing:** Add bloom, film grain, or vignette with Three.js `EffectComposer`
- **Scroll-driven animation:** Tie block rotations or camera position to scroll position

### Advanced:
- **Instanced rendering:** For thousands of blocks, use `InstancedMesh` (one draw call)
- **Physics:** Add cannon-es or rapier for realistic block tumbling
- **Shaders:** Write custom GLSL shaders for unique visual effects

## The Complete Architecture

```
index.html
├── CSS: full-screen canvas + config panel styles
├── HTML: toggle button + config panel (sliders, palette select, buttons)
└── JS (module):
    ├── CONFIG object (all tweakable values)
    ├── PALETTES dictionary
    ├── Scene / Camera / Renderer setup
    ├── generateBlocks() — creates grid of meshes in a Group
    ├── Raycaster — click detection → toggle spin on random axis
    ├── animate() — rotation loop + render
    ├── Resize handler
    └── Panel wiring (wireSlider, palette change, regenerate, pause)
```

Each piece connects to the next through the `CONFIG` object — the single source of truth.
