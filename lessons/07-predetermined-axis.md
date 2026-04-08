# Lesson 7: Pre-Assigned Rotation Axes

## What Changed

Previously, each block picked a **new random axis** every time you clicked it. Now, each block gets a **fixed rotation axis assigned at generation time** — and it always spins on that same axis when clicked. The axis is also limited to just **X and Z** (no Y).

## The Code Change

### At generation time (`generateBlocks`):

```js
// pick() is our existing helper: picks a random item from an array
mesh.userData.spinAxis = pick(["x", "z"]);
```

This runs inside the nested `for` loop, so every block gets its own random axis stored in `userData` before any click ever happens.

### At click time (`onPointerDown`):

```js
// Before — picked a new random axis every click:
const axes = ["x", "y", "z"];
hitMesh.userData.spinAxis = axes[Math.floor(Math.random() * 3)];

// After — just sets the speed, axis is already assigned:
hitMesh.userData.spinSpeed = CONFIG.rotationSpeed;
```

The click handler no longer touches `spinAxis` at all. It only toggles `spinning` on/off and sets the speed.

## The Principle: Front-Loading State

This is a common pattern in generative art and game development — **assign all properties at creation time**, not at interaction time. Benefits:

- **Deterministic behavior:** Given the same random seed, you'd get the same scene
- **Inspectable state:** You can read any block's axis before it's ever clicked
- **Simpler interaction code:** Click handlers do less work
- **Easier to serialize:** If you wanted to save/load a scene, all state is already on the mesh

## What You Should See

- Click any block — it spins on either X or Z (never Y)
- Click it off, then on again — **same axis** every time
- Hit "Regenerate" — blocks get **new** random axes (because `generateBlocks` re-runs)
