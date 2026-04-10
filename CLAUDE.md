# CLAUDE.md — Project Context for Claude Code

## What This Is

A single-file Three.js generative art scene (`index.html`) built as a learning exercise.
Inspired by Ezekiel Aquino's "Opus no.1". Reference video in `references/`.

## Architecture

Everything lives in `index.html` (~1000 lines):
- CSS: full-screen canvas + slide-out config panel
- HTML: toggle button + config panel (sliders, palette select, action buttons)
- JS module: all Three.js logic inline

Key code structure:
- `CONFIG` object — single source of truth for all tweakable values
- `PALETTES` dictionary — preset color arrays
- `generateBlocks()` — teardown + rebuild pattern (disposes geometry/materials, recreates Group)
- `createControls(cam, target)` — factory that disposes old OrbitControls and creates new ones
- Dual cameras: `perspCamera`, `orthoCamera`, `let activeCamera` as mutable pointer
- `animate()` loop: controls.update() -> pan clamping -> rotation updates -> render
- Raycasting on left-click only (right-click reserved for OrbitControls orbit)

## Conventions

- **Single HTML file** — no splitting into separate JS/CSS files
- **Heavily commented code** — inline comments explain Three.js concepts for learners
- **Lesson docs in `lessons/`** — one markdown file per step, tutorial voice
- **One commit per feature step** — commit messages prefixed with `step-N:`
- **MeshBasicMaterial** — flat unlit colors, no lighting setup
- **Three.js via CDN import map** — no npm, no bundler
- **No external UI libraries** — plain HTML controls for educational clarity

## When Adding Features

1. Create the feature in `index.html`
2. Add inline comments explaining new Three.js concepts
3. Write a companion `lessons/NN-topic.md` in tutorial voice
4. Commit with message format: `step-N: short description`

## Important Patterns

- `mesh.userData` stores per-object state (spinning, spinAxis, spinSpeed)
- OrbitControls must be recreated (not reused) when swapping cameras
- Both cameras must be updated on window resize, not just the active one
- Pan is clamped per-frame after `controls.update()` using `THREE.MathUtils.clamp()`
- `controls.dispose()` is critical before creating new controls (prevents ghost listeners)
- `event.stopPropagation()` on the config panel prevents clicks from reaching the canvas

## Current State

8 steps completed. Possible next features:
- Hover highlight (raycasting on pointermove)
- Lighting + shadows (switch to MeshStandardMaterial)
- More shape types (spheres, cones, torus)
- Post-processing effects (bloom, film grain)
- Scroll-driven animation
- Instanced rendering for performance at scale

## Refactoring to React

When porting this single-file scene into a React app:

### Component Structure
```
<SceneProvider>          <- React context holding CONFIG, activeCamera, scene ref
  <Canvas3D />           <- mounts renderer, owns the animate() loop via useEffect
  <ControlPanel />       <- replace inline HTML with React state + controlled inputs
  <CameraToggle />       <- button that calls context method to swap cameras
</SceneProvider>
```

### Key Patterns

- **useRef for Three.js objects** — Scene, Renderer, cameras, and controls are mutable singletons. Store them in `useRef`, not `useState`. React re-renders would destroy and recreate them otherwise.
- **useEffect for lifecycle** — Initialize renderer/scene/cameras in a `useEffect` with an empty dep array. Return a cleanup function that calls `renderer.dispose()`, `controls.dispose()`, and removes event listeners.
- **Don't let React own the canvas** — Attach the renderer to a `<div ref={mountRef}>` via `mountRef.current.appendChild(renderer.domElement)`. Don't use a JSX `<canvas>` — the renderer creates its own.
- **CONFIG becomes React state or context** — Slider changes call `setConfig(...)`, and a `useEffect` watching config triggers `generateBlocks()`. This replaces the manual `wireSlider()` pattern.
- **animate() lives outside React's render cycle** — Start `requestAnimationFrame` in a `useEffect`. The loop reads from refs, not state, to avoid stale closures. If you need state values inside the loop, sync them to a ref: `configRef.current = config`.
- **Teardown/rebuild stays the same** — `generateBlocks()` still disposes old meshes and creates new ones. Call it imperatively from a `useEffect` that depends on config, not from render.
- **Event handlers need care** — Raycasting pointer events should be attached to `renderer.domElement`, not via React's `onClick`. React's synthetic events give wrong coordinates for NDC conversion and don't respect the Three.js canvas lifecycle.

### Consider react-three-fiber (R3F)

[react-three-fiber](https://docs.pmnd.rs/react-three-fiber) is the standard React renderer for Three.js. It replaces most of the manual wiring above:

```jsx
<Canvas camera={{ position: [8, 12, 18], fov: 60 }}>
  <Block position={[x, 0, z]} color="#e63946" spinAxis="x" />
  <OrbitControls makeDefault mouseButtons={{ LEFT: null, RIGHT: 2 }} />
</Canvas>
```

- Three.js objects become JSX (`<mesh>`, `<boxGeometry>`, `<meshBasicMaterial>`)
- `useFrame()` replaces `requestAnimationFrame` — runs every frame with access to state
- `useThree()` gives access to scene, camera, renderer from any component
- Automatic disposal — R3F disposes geometry/materials when components unmount
- `@react-three/drei` provides ready-made `<OrbitControls>`, camera helpers, etc.

**Trade-off:** R3F adds abstraction. For learning Three.js fundamentals, the vanilla approach in this project is better. For production React apps, R3F avoids a whole class of lifecycle bugs.

### What NOT to Do

- **Don't call `renderer.render()` from a React effect** — it belongs in the rAF loop only
- **Don't store meshes in React state** — it triggers re-renders on every frame update and serialization issues with Three.js objects
- **Don't use `useState` for animation values** (spinning, spinSpeed) — updates at 60fps would thrash React's reconciler. Keep animation state in `userData` or refs
- **Don't split the animate loop across components** — one central loop reads all refs. Multiple competing rAF loops cause frame timing issues
