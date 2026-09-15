[English](README.md) | [简体中文](README_Simplified_Chinese.md) | [繁體中文](README_Classical_Chinese.md)

## 1. Overview

A **zero-dependency, pure vanilla JS, single-file** local image viewer. All processing happens locally in the browser — no files are ever uploaded. The UI is in Simplified Chinese and follows a minimalist white-background style (a lightweight take on macOS Preview / Google Photos).

---

## 2. Feature List

| Category | Capability |
|---|---|
| Open | Drag to window / folder (recursive expansion), file picker, folder picker (`webkitdirectory`) |
| Browse | Previous / next, thumbnail strip (shown only when >1 image), wrap-around cycling |
| Zoom | Wheel (pointer as anchor), two-finger pinch, double-click to toggle fit/1:1, buttons / shortcuts |
| View | Fit to window, actual size (100%), rotate 90°, fullscreen |
| Manage | Delete current item, Esc to clear, dedupe (path + size + lastModified) |
| Immersive | Auto-hide top/bottom toolbars when zoomed beyond fit; hover within 44px of top/bottom edge to temporarily reveal |
| Formats | JPG / PNG / GIF / WebP / SVG / BMP / ICO / AVIF / TIF, etc. |

---

## 3. Architecture & Data Flow

```
state {scale, tx, ty, rot, index}    data items[{file,url,name,path,key}]
        ↓                                        ↓
    apply() writes stage.transform          render img / thumbs / toolbar text
```

- **Single reused `<img>`**: switching images only swaps `src`, never rebuilds the DOM.
- **Stage wrapper**: `transform-origin:0 0`; the image centers itself on the stage origin via `translate(-50%,-50%)`. Thus the semantics of `(tx,ty)` stay very clean — **the image center's coordinates within the viewport**.

---

## 4. Core Implementation Notes

### 1. Transform model (the math is clean)
```js
stage.transform = `translate(tx,ty) scale(s) rotate(rot)`
```
All zoom/pan operations only mutate `tx/ty/scale`; the image itself is never reflowed.

### 2. Pointer-anchored zoom
```js
T' = m - k·(m - T)   // k = newScale/oldScale
```
Both `zoomAt` and `updatePinch` use the same formula; pinch additionally layers in the two-finger overall displacement. This keeps the "pixel under the finger" stationary while zooming, for a natural feel.

### 3. ⭐ P0 Key Design: decoupling fitScale from immersive
This is the most praiseworthy fix in the entire file. The problem was:

- In immersive mode the viewport goes from `h-54` to `h`;
- If fitScale were computed using the **actual viewport height**, zooming triggers immersive → viewport grows taller → fitScale grows → relative scale shrinks → exits immersive → viewport shrinks → …**infinite jitter loop**.

The solution: `computeFitScale()` always uses `win.h - BAR`, independent of the actual viewport height; `fit()`'s centering still uses the real `vp.h/2` (visually correct immediately). The criterion is unified, eliminating jitter.

```js
// Base height fixed at win.h - BAR, decoupled from immersive
return Math.min(win.w / w, Math.max(0, win.h - BAR) / h);
```

### 4. Geometry cache, avoiding forced synchronous layout
```js
let vp  = {left,top,w,h}   // refreshed by ResizeObserver
let win = {w,h}            // refreshed by resize
let imgDim = {w,h}         // refreshed once on onload
```
`pointermove`, pinch, and `checkEdgeReveal` read the cache throughout — never `getBoundingClientRect()` / `innerHeight`. This is key to smooth, finger-following drag.

### 5. Multi-touch state machine
Managed via `Map<pointerId, {x,y}>`:

| Event | Handling |
|---|---|
| 1st finger down | Enter drag |
| 2nd finger down | End drag, establish pinch baseline |
| 3rd+ finger down | Ignore, `pinch = null` debounce |
| Release from 2 → 1 finger | Continue drag with remaining finger as new start |
| Release from 3 → 2 fingers | **Unconditionally rebuild pinch baseline** (P0, otherwise "neither drag nor zoom" freeze) |

Combined with `setPointerCapture`, pointer events aren't lost even when moving outside the viewport.

### 6. Transition control
During drag / pinch / image-switch positioning, `stage.style.transition='none'`; restore via `requestAnimationFrame` on release — avoiding "rubber-band lag" and "image flying in".

---

## 5. File Handling

- **Dedupe key**: `(webkitRelativePath||name) + size + lastModified` — prevents different directories' same-named files from being misjudged.
- **Sorting**: sort only the **current batch** by path `localeCompare('zh')` (global order may be suboptimal across multiple drag-ins).
- **Dragging directories**: `webkitGetAsEntry()` → recursive `createReader().readEntries()`, using a `do...while` loop because `readEntries` **returns at most 100 entries per call**, so it must be repeatedly called until an empty array.

---

## 6. Accessibility (A11y) — done more thoroughly than typical demos

- Thumbnails: `role="button"` + `tabIndex=0` + `aria-label` + `:focus-visible` outline.
- Inner `<img alt="">` marked decorative, avoiding screen-reader double announcements.
- Delete buttons `aria-hidden="true"` (deletion goes through the Delete key uniformly, avoiding semantic ambiguity in the a11y tree).
- `pct` writes to the DOM only when the percentage **actually changes**, with an `aria-label` — preventing the screen reader from announcing every frame during drag.
- After delete/clear, actively move focus to a sensible element (new current thumbnail / "Select Image" button), never dropping focus onto `body`.
- The spacebar is **not hijacked** when focus is on `button/[role=button]/a/textarea/select`, deferring to native activation.
- Icon buttons auto-fill `aria-label` from `title`.

---

## 7. Memory & Lifecycle

- `URL.createObjectURL` + `revokeObjectURL` on delete / clear.
- **`pagehide` replaces `beforeunload`**, and checks `e.persisted`: if the page enters bfcache, don't reclaim — otherwise images would all break after back/forward restoration. This is a very typical correct practice.

---

## 8. P0–P3 Markers in Code Comments

This set of markers shows the code went through a round of **systematic review/iteration**, with comments explaining "why" rather than "what" — far more readable than the average demo:

- **P0**: causes functional bugs (fit/immersive infinite loop, pinch baseline freeze)
- **P1**: affects usability (spacebar hijacking buttons, bfcache reclamation)
- **P2**: experience/performance details (DOM-write guards, cached geometry reads, immersive-state residue)
- **P3**: polish items (focus management, size reset)

---

## Screenshots
---
## License

[MIT](LICENSE)
