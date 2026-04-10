# 3D Scenes — Generative Block Grid

A self-contained Three.js learning project that recreates the style of [Ezekiel Aquino's "Opus no.1"](https://ezekielaquino.com/information) — colorful 3D cuboids arranged in a grid that rotate on random axes when clicked.

## Quick Start

Open `index.html` in any modern browser. No build tools, no npm, no server required.

## What It Does

- Grid of colored 3D boxes with randomized heights and widths
- Click any block to spin it on its pre-assigned axis (X or Z)
- Right-click + drag to orbit, scroll to zoom, middle-click to pan
- Toggle between perspective and orthographic cameras
- Configurable via a slide-out control panel (block count, size ranges, speed, palettes)
- Multiple color palettes: Mondrian, Pastel, Monochrome, Neon, Earth

## Learning Path

The project was built incrementally. Each step has a companion lesson doc in `lessons/`:

| Step | Lesson | Concepts |
|------|--------|----------|
| 1 | [Scene Setup](lessons/01-scene-setup.md) | Scene, Camera, Renderer, render loop |
| 2 | [Generative Grid](lessons/02-generative-grid.md) | Geometry, Materials, Meshes, Groups |
| 3 | [Click to Rotate](lessons/03-click-to-rotate.md) | Raycasting, pointer events, userData |
| 4 | [Config Panel](lessons/04-config-panel.md) | DOM-to-Three.js bridge, teardown/rebuild |
| 5 | [Polish](lessons/05-polish.md) | Palettes, resize handling, pause/resume |
| 6 | [Camera Controls](lessons/06-camera-controls.md) | OrbitControls, damping, mouse remapping |
| 7 | [Pre-assigned Axes](lessons/07-predetermined-axis.md) | Front-loading state at generation time |
| 8 | [Camera Toggle](lessons/08-camera-toggle.md) | Dual cameras, OrbitControls factory |

Each step corresponds to a git commit — use `git log` to walk through the progression.

## Controls

| Input | Action |
|-------|--------|
| Left-click block | Toggle spin |
| Right-click + drag | Orbit camera |
| Scroll wheel | Zoom |
| Middle-click + drag | Pan |
| Toggle button (top-left) | Open/close config panel |

## Tech Stack

- [Three.js](https://threejs.org/) v0.170.0 via CDN (unpkg)
- Single HTML file — all CSS and JS inline
- No dependencies, no bundler, no framework
