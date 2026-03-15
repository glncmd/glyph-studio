# ASCII Effects App — Design Spec

## Overview

A local browser app for applying ASCII art effects to photos and videos in real-time. Users can upload images, videos, or use their webcam, then adjust ASCII rendering parameters and export the results.

## Architecture

Single `index.html` file using Tailwind CSS (CDN). No build tools, no server — opens directly in the browser. All logic is inline JavaScript using Canvas 2D APIs.

### Canvas Pipeline

Two canvases work together:

1. **Source canvas** (hidden) — receives the raw image/video frame. Adjustments (brightness, contrast, saturation) are applied via the Canvas 2D `ctx.filter` property (e.g., `ctx.filter = 'brightness(1.2) contrast(1.1) saturate(0.8)'`) before drawing the source, so that `getImageData` reads the filtered result.
2. **Output canvas** (visible) — receives the rendered ASCII characters.

For overlay-mode variants (Contour), a third compositing step draws the original image first, then the ASCII layer on top.

### Render Loop

```
Source (img/video/webcam)
  → Draw to source canvas
  → Apply ctx.filter (brightness, contrast, saturation)
  → Read pixel data (getImageData)
  → Divide into grid (cell size determines grid resolution)
  → Per cell:
      → Replacement mode: compute average luminance → map to character → draw colored/white character on black background
      → Overlay mode: compute edge magnitude (Sobel) → map to character by edge strength → draw on top of original image
  → Output to visible canvas
  → For video/webcam: requestAnimationFrame → repeat
```

## Rendering Modes

### Replacement Mode (Luminosity-Based)

Used by: Standard, Dense, Minimal, Blocks, Braille, Technical, Matrix, Hatching.

Each cell's average luminance (0-255) maps to a character from the variant's character set, ordered by visual density. Dark regions get dense characters, light regions get sparse ones (reversed when Invert is on). The output canvas has a black background with colored or white characters.

**Color mode on:** Each character is drawn in the average color of its source cell.
**Color mode off:** All characters are white.

### Overlay Mode (Edge-Based Hybrid)

Used by: Contour.

The original image is drawn to the output canvas first. Then a Sobel edge detection pass computes gradient magnitude per cell. Every cell gets a baseline dot character. Cells with higher edge magnitude get upgraded to larger characters.

| Edge Strength | Character |
|---|---|
| None (flat) | `.` |
| Weak | `:` |
| Moderate | `o` |
| Strong | `0` |
| Very strong | `O` |

Characters are drawn with alpha 0.7, creating the dot-matrix overlay effect seen in the reference images.

**Color mode on:** Characters use the source cell's average color (with 0.7 alpha).
**Color mode off:** Characters are white (with 0.7 alpha).

## Character Variants

Each variant declares its rendering mode and character set:

| Variant | Mode | Characters |
|---|---|---|
| Standard | replacement | ` .:-=+*#%@` |
| Dense | replacement | ` .'"\`:;I!i><~+_-?][}{1)(\/tfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@$` |
| Minimal | replacement | ` .:*#` |
| Blocks | replacement | ` ░▒▓█` |
| Braille | replacement | `⠀ ⠁ ⠂ ⠃ ⠄ ⠅ ⠆ ⠇ ⠈ ⠉ ⠊ ⠋ ⠌ ⠍ ⠎ ⠏ ⠐ ⠑ ⠒ ⠓ ⠔ ⠕ ⠖ ⠗ ⠘ ⠙ ⠚ ⠛ ⠜ ⠝ ⠞ ⠟ ⠠ ⠡ ⠢ ⠣ ⠤ ⠥ ⠦ ⠧ ⠨ ⠩ ⠪ ⠫ ⠬ ⠭ ⠮ ⠯ ⠰ ⠱ ⠲ ⠳ ⠴ ⠵ ⠶ ⠷ ⠸ ⠹ ⠺ ⠻ ⠼ ⠽ ⠾ ⠿` — sorted by number of raised dots (popcount of code point low byte) |
| Technical | replacement | ` .-+:;=!*#$@` |
| Matrix | replacement | ` 0123456789ABCDEF` — ordered by hex value (0=darkest, F=lightest) |
| Hatching | replacement | ` \ / \| - + = X #` (space, backslash, forward slash, pipe, dash, plus, equals, X, hash) |
| Contour | overlay | `. : o 0 O` |

## Settings

### Basic Settings

- **Cell Size** (slider, 4–30, default 10): Size in pixels of each sampling cell. Smaller = more detail, slower rendering.
- **Invert** (toggle, default off): Flip the luminance-to-character mapping. Only applies to replacement mode variants.
- **Color Mode** (toggle, default on): Use source colors vs. monochrome white. In replacement mode, characters use source cell color or white. In overlay mode, characters use source cell color (alpha 0.7) or white (alpha 0.7).
- **Character Rotation** (toggle, default off): Rotate each character based on the local gradient direction (Sobel angle). Applies to both modes. In overlay mode, reuses the same Sobel pass that computes edge magnitude (magnitude → character, angle → rotation).

### Adjustments

- **Brightness** (slider, 0–3, step 0.05, default 1): `ctx.filter` brightness on source canvas.
- **Contrast** (slider, 0–3, step 0.05, default 1): `ctx.filter` contrast on source canvas.
- **Saturation** (slider, 0–3, step 0.05, default 1): `ctx.filter` saturate on source canvas.

## UI Layout

### Top Bar

Left side — input source icons:
- Upload image (image icon)
- Upload video (film icon)
- Webcam (camera icon)

Right side — actions:
- Export PNG (download icon)
- Record video (circle/stop icon, red pulse while recording)
- Settings toggle (sliders/gear icon)

All buttons are icon-only with tooltips on hover, except the record button which shows a red dot indicator while active.

### Main Canvas

Full-width, full-height (minus top bar). Dark background (#0a0a0a). The ASCII output canvas fills this area using "contain" sizing — maintains source aspect ratio, letterboxed (centered) if needed. When the settings drawer opens, the canvas area shrinks horizontally and re-renders at the new dimensions.

**Empty state:** Before any source is loaded, show centered text: "Drop an image or video here, or use the buttons above" in muted gray (#555).

### Video Controls

Shown only when a video file is loaded (not for images or webcam):
- Play/pause button
- Seek bar (progress slider)
- Current time / duration display

### Settings Drawer

Slides in from the right edge, ~300px wide. Dark background (#111) with subtle left border. Contains:

1. **Variant selector** — dropdown with left/right arrow buttons for quick cycling
2. **Basic Settings** section — cell size slider, invert/color mode/char rotation toggles
3. **Adjustments** section — brightness/contrast/saturation sliders with numeric input

Close button (X) in the top-right of the drawer. Clicking the settings icon in the top bar toggles the drawer. The main canvas area resizes to accommodate the drawer.

## Export

### Image Export (PNG)

- Captures current canvas state via `canvas.toBlob('image/png')`
- For overlay mode: composites source image + ASCII layer into a temporary canvas, exports that
- Triggers browser download as `ascii-export.png`

### Video Recording (WebM)

- Uses `canvas.captureStream()` + `MediaRecorder` API
- Record button toggles between record (circle icon) and stop (square icon)
- Red pulse animation on the record button while active
- On stop: creates `.webm` blob, triggers download as `ascii-recording.webm`
- For overlay mode: captures from the composite canvas

## Input Sources

### Image Upload

- File input accepting `image/*`
- Drag-and-drop onto the canvas area
- Draws to source canvas, triggers single render pass
- Re-renders when any setting changes

### Video Upload

- File input accepting `video/*`
- Drag-and-drop onto the canvas area (same as images)
- Loads into a hidden `<video>` element
- Playback controls appear (play/pause, seek, timestamp)
- `requestAnimationFrame` loop reads each frame and renders

### Webcam

- `navigator.mediaDevices.getUserMedia({ video: true })`
- Streams to hidden `<video>` element
- Same `requestAnimationFrame` render loop as video
- No playback controls shown (live feed)
- Clicking any other input source (image/video upload) stops the webcam stream (`stream.getTracks().forEach(t => t.stop())`)

## Performance Considerations

- Cell size directly controls performance: larger cells = fewer characters = faster rendering
- For video, skip frames if rendering takes longer than the frame interval (target 30fps)
- Use `willReadFrequently: true` on source canvas context for optimized `getImageData`
- Sobel edge detection (for Contour/character rotation) adds ~30% overhead per frame

## Tech Stack

- HTML/CSS/JS — single file, no dependencies beyond Tailwind CDN
- Tailwind CSS via `<script src="https://cdn.tailwindcss.com"></script>`
- Canvas 2D API for rendering
- MediaRecorder API for video export
- No server, no build step — open `index.html` in browser
