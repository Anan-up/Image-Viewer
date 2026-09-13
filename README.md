[English](README.md) | [简体中文](README_Simplified_Chinese.md) | [繁體中文](README_Classical_Chinese.md)

## I. Overview

A **purely local, zero-dependency, single-file** image viewer (web app). All processing happens inside the browser — no files are uploaded, no third-party libraries, no build step. Just double-click the HTML to run.

---

## II. Core Features

### 1. Loading images (three ways)

| Method | Implementation |
|---|---|
| Select files | `<input type="file" multiple>` |
| Select folder | `<input webkitdirectory>` (Chrome/Edge) |
| Drag into window | Global dragenter/drop events + overlay hint |

- Uses `URL.createObjectURL()` to generate a local URL, sorted by `webkitRelativePath` using the **Chinese locale**.
- Supported formats: JPG/PNG/GIF/WebP/SVG/BMP/ICO/AVIF/TIF, etc. (MIME detection + extension fallback).

### 2. View transformation (core state machine)

```js
state = { scale, tx, ty, rot, index, fitScale }
```

- **Pan**: Pointer-event dragging (`setPointerCapture`, touch supported)
- **Zoom**: Mouse-wheel zoom anchored at the **pointer position** (formula `T' = m - k(m - T)`), range 0.02 ~ 60×
- **Fit to window**: Computed from viewport ratio; swaps width/height at 90°/270° rotation
- **Actual size**: Back to 1:1
- **Rotate**: +90° each time
- **Double-click**: Toggle between "fit" and "100%"

### 3. Navigation

- Previous / next image (looping, modulo)
- Thumbnail strip (shown when `items.length > 1`; auto-scrolls to current; single item removable)
- Keyboard shortcuts: `←/→/Space/PageUp/PageDown` to page, `+/-/0/1/r/Delete/Esc` and more

### 4. Immersive mode (highlight design)

- When `scale > fitScale` (zoomed past the viewport), automatically enters `immersive`:
  - Top/bottom toolbars fade out, viewport fills the screen
  - Moving the mouse within **44px of the top/bottom edge** temporarily reveals the toolbar (`reveal`)
  - Auto-exits immersive mode when scaled back to fit

### 5. Misc

- Fullscreen (Fullscreen API, icon switches with state)
- Delete current image (`URL.revokeObjectURL` to free memory)
- `Esc` priority: exit fullscreen first; if not in fullscreen, clear all images back to the empty state

---

## III. Code Structure

```
IIFE closure (strict mode)
├── constants: MIN_SCALE / MAX_SCALE
├── state object
├── transform functions: apply / zoomAt / zoomCenter / fit / actual / rotate
├── render: renderCurrent / updateInfo / renderThumbs / updateThumbsActive / select
├── file: addFiles / removeCurrent / clearAll / showViewer / showEmpty
├── interaction: drag-pan / wheel-zoom / double-click / toolbar / thumbnails / drag-in
└── global: mousemove (edge reveal) + keydown shortcuts
```

**DOM structure**: empty state → toolbar → viewport (stage+img) → thumbnail strip → drag overlay → two hidden inputs.

---

## IV. Design Strengths

1. **No memory leaks**: Correct `revokeObjectURL` on delete/clear.
2. **Clean transform composition**: `translate + scale + rotate` unified on the stage; the image itself uses `translate(-50%,-50%)` to anchor at center.
3. **Performance**: `will-change:transform`, `pointer-events:none` to stop the image interfering with dragging, `touch-action:none` to suppress mobile default gestures.
4. **Minimal UI**: White theme, thin dividers, tabular-nums for aligned digits, generous whitespace.
5. **Complete interaction**: Mouse, touch (unified via Pointer Events), and keyboard all covered.

---

## Screenshots

---

## License

[MIT](LICENSE)
