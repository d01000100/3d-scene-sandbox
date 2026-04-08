# Lesson 4: Configuration Panel — Connecting DOM to Three.js

## Architecture

The config panel is **just HTML/CSS** layered on top of the canvas with `position: fixed`. Three.js doesn't know it exists — we bridge them with JavaScript event listeners.

```
[ HTML Panel ]  ←→  [ CONFIG object ]  ←→  [ Three.js scene ]
   (DOM)           (plain JS object)       (WebGL/GPU)
```

## The Slide-In Pattern

The panel starts off-screen using CSS `transform`:

```css
#config-panel {
  transform: translateX(100%);       /* pushed fully off-screen right */
  transition: transform 0.3s ease;  /* animate when class changes */
}
#config-panel.open {
  transform: translateX(0);          /* slides to natural position */
}
```

Toggle with one line:
```js
configPanel.classList.toggle('open');
```

Using `transform` instead of changing `right` or `width` is important for performance — transforms are GPU-accelerated and don't trigger layout recalculation.

## Wiring Sliders to CONFIG

Each slider follows the same pattern:

```js
slider.addEventListener('input', () => {
  const value = parseFloat(slider.value);
  CONFIG.someProperty = value;      // update the config
  display.textContent = value;      // show the value
});
```

We use the `'input'` event (not `'change'`) because:
- `input` fires **continuously** while dragging the slider
- `change` only fires when you **release** the slider

For live feedback, `input` is what you want.

### The wireSlider helper

To avoid repeating this pattern for every slider, we use a helper:

```js
function wireSlider(sliderId, displayId, setter) {
  const slider = document.getElementById(sliderId);
  const display = document.getElementById(displayId);
  slider.addEventListener('input', () => {
    const value = parseFloat(slider.value);
    display.textContent = value;
    setter(value);
  });
}
```

The `setter` callback lets each slider update a different CONFIG property:

```js
wireSlider('cfg-rows', 'val-rows', (v) => { CONFIG.rows = v; });
wireSlider('cfg-gap',  'val-gap',  (v) => { CONFIG.gap = v; });
```

## Regenerate vs. Live Update

Some settings work **live** (rotation speed takes effect immediately because the animation loop reads `CONFIG.rotationSpeed` every frame).

Others require **regeneration** (rows, cols, gap, block sizes) because the blocks are created once and positioned. Changing these settings means we must:
1. Remove all old blocks (`scene.remove(group)`)
2. Dispose of GPU resources (`.dispose()`)
3. Re-run `generateBlocks()` with the new CONFIG

That's why there's a "Regenerate" button instead of auto-rebuilding on every slider tick.

## Preventing Event Leakthrough

Critical detail: clicks on the panel would also reach the canvas and trigger raycasting. We prevent this with:

```js
configPanel.addEventListener('pointerdown', (e) => e.stopPropagation());
```

`stopPropagation()` prevents the event from bubbling up to the canvas's `pointerdown` handler.

## Why Not dat.gui or lil-gui?

Libraries like [lil-gui](https://lil-gui.georgealways.com/) automate this entire step. They're great for production, but building it manually teaches you:
- How HTML overlays interact with WebGL canvas
- Event propagation and stopPropagation
- The CONFIG → regenerate pattern
- CSS transform animations

Once you understand the manual version, lil-gui is just a shortcut.

## What You Should See

- "Config" button in the top-right corner
- Click it → panel slides in from the right
- Adjust sliders → values update in real time
- Click "Regenerate" → scene rebuilds with new settings
- Click "Close" → panel slides away

## Next Step
→ [Lesson 5: Polish](./05-polish.md) — Add palettes, window resize, and pause/resume.
