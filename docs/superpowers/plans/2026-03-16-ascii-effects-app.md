# ASCII Effects App Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file browser app that applies real-time ASCII art effects to photos and videos with adjustable rendering parameters and export capabilities.

**Architecture:** Single `index.html` with Tailwind CSS (CDN). Two-canvas pipeline — hidden source canvas with `ctx.filter` for adjustments, visible output canvas for ASCII rendering. `requestAnimationFrame` loop for video/webcam. All JS inline.

**Tech Stack:** HTML, Tailwind CSS (CDN), Canvas 2D API, MediaRecorder API

**Spec:** `docs/superpowers/specs/2026-03-16-ascii-effects-app-design.md`

**Note:** This is a single HTML file with no build tools or test framework. Each task is verified by opening `index.html` in a browser and checking behavior manually. "Test" steps mean: open the file, perform the described action, confirm the expected result.

---

## Chunk 1: Foundation — UI Shell + Basic Rendering

### Task 1: HTML Scaffold and UI Shell

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create the HTML file with Tailwind, dark theme, and top bar**

Create `index.html` with:
- Tailwind CDN script tag
- Dark background (`bg-[#0a0a0a]`), full viewport height, no scroll
- Top bar (`bg-[#111]`, border-bottom `border-[#222]`) with:
  - Left: 3 icon buttons — image upload (SVG image icon), video upload (SVG film icon), webcam (SVG camera icon). Each is a `<button>` with a hidden `<input type="file">` triggered on click.
  - Right: 3 icon buttons — export PNG (SVG download icon), record video (SVG circle icon), settings toggle (SVG sliders icon)
  - All icon buttons: `bg-[#222] border border-[#333] rounded-lg p-2 text-[#aaa] hover:text-white hover:border-[#555]` with `title` attribute for tooltip
- Main area: flex container taking remaining height
- Output `<canvas id="outputCanvas">` filling main area
- Hidden `<canvas id="sourceCanvas">`
- Hidden `<video id="videoSource">`
- Hidden `<input type="file" id="imageInput" accept="image/*">`
- Hidden `<input type="file" id="videoInput" accept="video/*">`
- Empty state: centered `<div id="emptyState">` with text "Drop an image or video here, or use the buttons above" in `text-[#555]`
- Settings drawer `<div id="settingsDrawer">` — 300px wide, `bg-[#111] border-l border-[#222]`, off-screen by default (`translate-x-full`), with CSS transition for slide animation
- SVG icons should be inline (24x24, stroke-based, currentColor)

- [ ] **Step 2: Add settings drawer contents**

Inside `#settingsDrawer`:

**Header row:** "Settings" label (white, font-semibold) + close button (X icon, `cursor-pointer text-[#666] hover:text-white`)

**Variant selector:** Row with left arrow button (`<`), dropdown `<select id="variantSelect">` (centered, `bg-[#222] border border-[#333] rounded-lg text-white`) with options: Standard, Dense, Minimal, Blocks, Braille, Technical, Matrix, Hatching, Contour — and right arrow button (`>`)

**Basic Settings section:**
- Section label: `text-[#888] text-xs uppercase tracking-wider`
- Cell Size: label + value display (`<span id="cellSizeValue">10</span>` in `bg-[#222] border border-[#333] rounded px-2`) + range slider (`<input type="range" id="cellSize" min="4" max="30" value="10" step="1">`)
- Invert: label + toggle switch (`<input type="checkbox" id="invertToggle">` styled as toggle)
- Color Mode: label + toggle switch (`<input type="checkbox" id="colorModeToggle" checked>`)
- Character Rotation: label + toggle switch (`<input type="checkbox" id="charRotationToggle">`)

**Adjustments section:**
- Brightness: label + value display (`<span id="brightnessValue">1</span>`) + range slider (`<input type="range" id="brightness" min="0" max="3" step="0.05" value="1">`)
- Contrast: same pattern, id="contrast"
- Saturation: same pattern, id="saturation"

Style all range sliders with custom CSS: track is `bg-[#333]` rounded, thumb is white circle. Toggle switches: `w-9 h-5 rounded-full` with sliding circle.

- [ ] **Step 3: Wire up drawer toggle and basic interactions**

Add `<script>` at end of body with:

```javascript
// State object
const state = {
  source: null, // 'image' | 'video' | 'webcam' | null
  drawerOpen: false,
  variant: 'Standard',
  cellSize: 10,
  invert: false,
  colorMode: true,
  charRotation: false,
  brightness: 1,
  contrast: 1,
  saturation: 1,
  recording: false,
  webcamStream: null
};

// Drawer toggle
const drawer = document.getElementById('settingsDrawer');
const settingsBtn = document.getElementById('settingsBtn');
settingsBtn.addEventListener('click', () => {
  state.drawerOpen = !state.drawerOpen;
  drawer.classList.toggle('translate-x-full', !state.drawerOpen);
  resizeCanvas();
});
document.getElementById('drawerClose').addEventListener('click', () => {
  state.drawerOpen = false;
  drawer.classList.add('translate-x-full');
  resizeCanvas();
});

// Settings bindings — each input updates state and calls render()
document.getElementById('cellSize').addEventListener('input', e => {
  state.cellSize = parseInt(e.target.value);
  document.getElementById('cellSizeValue').textContent = state.cellSize;
  render();
});
// ... same pattern for brightness, contrast, saturation, toggles
// Variant select + arrow buttons cycle through variants
```

- [ ] **Step 4: Verify — open in browser**

Open `index.html`. Confirm:
- Dark full-screen layout with top bar icons
- Empty state message visible
- Settings button opens/closes drawer with slide animation
- All sliders and toggles work (update displayed values)
- Variant dropdown and arrow buttons cycle variants

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: scaffold HTML shell with top bar, canvas, and settings drawer"
```

### Task 2: Image Upload and Basic Replacement Rendering

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add image loading (file input + drag-and-drop)**

```javascript
const sourceCanvas = document.getElementById('sourceCanvas');
const sourceCtx = sourceCanvas.getContext('2d', { willReadFrequently: true });
const outputCanvas = document.getElementById('outputCanvas');
const outputCtx = outputCanvas.getContext('2d');

// Stub functions — replaced by real implementations in later tasks
function stopWebcam() {}
function hideVideoControls() {}
function loadVideo() {}
let animFrameId = null;

function loadImage(file) {
  if (animFrameId) { cancelAnimationFrame(animFrameId); animFrameId = null; }
  stopWebcam();
  const img = new Image();
  img.onload = () => {
    state.source = 'image';
    state.sourceElement = img;
    hideVideoControls();
    document.getElementById('emptyState').classList.add('hidden');
    resizeCanvas();
    URL.revokeObjectURL(img.src);
  };
  img.src = URL.createObjectURL(file);
}

document.getElementById('imageInput').addEventListener('change', e => {
  if (e.target.files[0]) loadImage(e.target.files[0]);
});

// Drag-and-drop on canvas area
const canvasArea = document.getElementById('canvasArea');
canvasArea.addEventListener('dragover', e => { e.preventDefault(); });
canvasArea.addEventListener('drop', e => {
  e.preventDefault();
  const file = e.dataTransfer.files[0];
  if (!file) return;
  if (file.type.startsWith('image/')) loadImage(file);
  else if (file.type.startsWith('video/')) loadVideo(file);
});
```

- [ ] **Step 2: Implement resizeCanvas() to size canvases correctly**

```javascript
function resizeCanvas() {
  const area = canvasArea.getBoundingClientRect();
  outputCanvas.width = area.width;
  outputCanvas.height = area.height;

  if (!state.sourceElement) return;

  // Source canvas matches source dimensions
  const src = state.sourceElement;
  const srcW = src.videoWidth || src.naturalWidth || src.width;
  const srcH = src.videoHeight || src.naturalHeight || src.height;
  sourceCanvas.width = srcW;
  sourceCanvas.height = srcH;

  // Only re-render for static images (video/webcam has its own render loop)
  if (state.source === 'image') render();
}
window.addEventListener('resize', resizeCanvas);
```

- [ ] **Step 3: Implement the core render() function for replacement mode**

```javascript
// Character variant definitions
const VARIANTS = {
  Standard:  { mode: 'replacement', chars: ' .:-=+*#%@' },
  Dense:     { mode: 'replacement', chars: ' .\'"`:;I!i><~+_-?][}{1)(\\/tfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@$' },
  Minimal:   { mode: 'replacement', chars: ' .:*#' },
  Blocks:    { mode: 'replacement', chars: ' ░▒▓█' },
  Braille:   { mode: 'replacement', chars: generateBrailleChars() },
  Technical: { mode: 'replacement', chars: ' .-+:;=!*#$@' },
  Matrix:    { mode: 'replacement', chars: ' 0123456789ABCDEF' },
  Hatching:  { mode: 'replacement', chars: ' \\/|-+=X#' },
  Contour:   { mode: 'overlay', chars: '.:o0O' }
};

function generateBrailleChars() {
  // Unicode braille U+2800-U+283F sorted by popcount
  const chars = [];
  for (let i = 0; i < 64; i++) chars.push({ cp: 0x2800 + i, bits: popcount(i) });
  chars.sort((a, b) => a.bits - b.bits || a.cp - b.cp);
  return chars.map(c => String.fromCharCode(c.cp)).join('');
}
function popcount(n) { let c = 0; while (n) { c += n & 1; n >>= 1; } return c; }

function render() {
  if (!state.sourceElement) return;

  const variant = VARIANTS[state.variant];
  const src = state.sourceElement;
  const srcW = sourceCanvas.width;
  const srcH = sourceCanvas.height;

  // Draw source with filters
  sourceCtx.filter = `brightness(${state.brightness}) contrast(${state.contrast}) saturate(${state.saturation})`;
  sourceCtx.drawImage(src, 0, 0, srcW, srcH);
  sourceCtx.filter = 'none';

  const imageData = sourceCtx.getImageData(0, 0, srcW, srcH);
  const pixels = imageData.data;

  // Compute "contain" sizing for output
  const outW = outputCanvas.width;
  const outH = outputCanvas.height;
  const scale = Math.min(outW / srcW, outH / srcH);
  const drawW = srcW * scale;
  const drawH = srcH * scale;
  const offsetX = (outW - drawW) / 2;
  const offsetY = (outH - drawH) / 2;

  const cellSize = state.cellSize;
  const cols = Math.floor(srcW / cellSize);
  const rows = Math.floor(srcH / cellSize);

  // Clear output
  outputCtx.fillStyle = '#0a0a0a';
  outputCtx.fillRect(0, 0, outW, outH);

  if (variant.mode === 'replacement') {
    renderReplacement(pixels, srcW, srcH, cols, rows, cellSize, variant.chars, scale, offsetX, offsetY);
  }
  // overlay mode handled in Task 5
}

function renderReplacement(pixels, srcW, srcH, cols, rows, cellSize, chars, scale, offsetX, offsetY) {
  const fontSize = cellSize * scale;
  outputCtx.font = `${fontSize}px monospace`;
  outputCtx.textBaseline = 'middle';
  outputCtx.textAlign = 'center';

  for (let row = 0; row < rows; row++) {
    for (let col = 0; col < cols; col++) {
      // Sample average color of this cell
      let r = 0, g = 0, b = 0, count = 0;
      for (let dy = 0; dy < cellSize; dy++) {
        for (let dx = 0; dx < cellSize; dx++) {
          const px = (col * cellSize + dx);
          const py = (row * cellSize + dy);
          const i = (py * srcW + px) * 4;
          r += pixels[i];
          g += pixels[i + 1];
          b += pixels[i + 2];
          count++;
        }
      }
      r = Math.round(r / count);
      g = Math.round(g / count);
      b = Math.round(b / count);

      // Luminance
      let lum = (0.299 * r + 0.587 * g + 0.114 * b) / 255;
      if (state.invert) lum = 1 - lum;

      // Map to character
      const charIndex = Math.min(Math.floor(lum * chars.length), chars.length - 1);
      const char = chars[charIndex];

      if (char === ' ') continue; // skip spaces for performance

      // Position on output canvas
      const x = offsetX + (col + 0.5) * cellSize * scale;
      const y = offsetY + (row + 0.5) * cellSize * scale;

      outputCtx.fillStyle = state.colorMode ? `rgb(${r},${g},${b})` : '#ffffff';
      outputCtx.fillText(char, x, y);
    }
  }
}
```

- [ ] **Step 4: Verify — upload an image**

Open `index.html`, upload an image. Confirm:
- ASCII art appears on black background
- Characters are colored (color mode is on by default)
- Changing cell size re-renders with different detail
- Brightness/contrast/saturation sliders affect the output
- Invert toggle flips the character mapping
- Variant dropdown changes character set used
- Drag-and-drop also works

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: image upload with replacement-mode ASCII rendering"
```

---

## Chunk 2: Overlay Mode + Character Rotation

### Task 3: Contour Mode (Sobel Edge Detection Overlay)

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Implement Sobel edge detection helper**

```javascript
function sobelAt(pixels, srcW, srcH, x, y) {
  // 3x3 Sobel kernels
  // Returns { magnitude, angle }
  function lum(px, py) {
    px = Math.max(0, Math.min(srcW - 1, px));
    py = Math.max(0, Math.min(srcH - 1, py));
    const i = (py * srcW + px) * 4;
    return 0.299 * pixels[i] + 0.587 * pixels[i + 1] + 0.114 * pixels[i + 2];
  }

  const tl = lum(x - 1, y - 1), tc = lum(x, y - 1), tr = lum(x + 1, y - 1);
  const ml = lum(x - 1, y),                           mr = lum(x + 1, y);
  const bl = lum(x - 1, y + 1), bc = lum(x, y + 1), br = lum(x + 1, y + 1);

  const gx = -tl + tr - 2 * ml + 2 * mr - bl + br;
  const gy = -tl - 2 * tc - tr + bl + 2 * bc + br;

  return {
    magnitude: Math.sqrt(gx * gx + gy * gy),
    angle: Math.atan2(gy, gx)
  };
}
```

- [ ] **Step 2: Implement renderOverlay() for Contour mode**

```javascript
function renderOverlay(pixels, srcW, srcH, cols, rows, cellSize, chars, scale, offsetX, offsetY) {
  // Draw original image to output canvas first
  const outW = outputCanvas.width;
  const outH = outputCanvas.height;
  const drawW = srcW * scale;
  const drawH = srcH * scale;
  outputCtx.drawImage(sourceCanvas, offsetX, offsetY, drawW, drawH);

  const fontSize = cellSize * scale;
  outputCtx.font = `${fontSize}px monospace`;
  outputCtx.textBaseline = 'middle';
  outputCtx.textAlign = 'center';

  // Edge strength thresholds for character tiers
  const thresholds = [0, 30, 80, 150, 220]; // maps to . : o 0 O

  for (let row = 0; row < rows; row++) {
    for (let col = 0; col < cols; col++) {
      const cx = col * cellSize + Math.floor(cellSize / 2);
      const cy = row * cellSize + Math.floor(cellSize / 2);

      const { magnitude, angle } = sobelAt(pixels, srcW, srcH, cx, cy);

      // Map magnitude to character tier
      let charIndex = 0; // baseline dot
      for (let t = thresholds.length - 1; t >= 0; t--) {
        if (magnitude >= thresholds[t]) { charIndex = t; break; }
      }
      const char = chars[charIndex];

      const x = offsetX + (col + 0.5) * cellSize * scale;
      const y = offsetY + (row + 0.5) * cellSize * scale;

      // Color
      const pi = (cy * srcW + cx) * 4;
      const r = pixels[pi], g = pixels[pi + 1], b = pixels[pi + 2];

      outputCtx.save();
      if (state.charRotation) {
        outputCtx.translate(x, y);
        outputCtx.rotate(angle);
        outputCtx.globalAlpha = 0.7;
        outputCtx.fillStyle = state.colorMode ? `rgb(${r},${g},${b})` : '#ffffff';
        outputCtx.fillText(char, 0, 0);
      } else {
        outputCtx.globalAlpha = 0.7;
        outputCtx.fillStyle = state.colorMode ? `rgb(${r},${g},${b})` : '#ffffff';
        outputCtx.fillText(char, x, y);
      }
      outputCtx.restore();
    }
  }
}
```

- [ ] **Step 3: Wire overlay mode into render()**

In the `render()` function, after the replacement mode branch, add:

```javascript
if (variant.mode === 'overlay') {
  renderOverlay(pixels, srcW, srcH, cols, rows, cellSize, variant.chars, scale, offsetX, offsetY);
}
```

- [ ] **Step 4: Verify — test Contour mode**

Upload an image, switch variant to "Contour". Confirm:
- Original image visible underneath
- Dot grid covers entire image
- Edges/contours show larger characters (o, 0, O)
- Flat areas show only dots
- Color mode toggle works (colored vs white characters)
- Adjustments affect the source image visible underneath

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add Contour overlay mode with Sobel edge detection"
```

### Task 4: Character Rotation

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add rotation to replacement mode rendering**

In `renderReplacement()`, wrap the character drawing with rotation when `state.charRotation` is true:

```javascript
// Inside the cell loop, replace the fillText call:
if (state.charRotation) {
  // Compute Sobel at cell center for gradient angle
  const cx = col * cellSize + Math.floor(cellSize / 2);
  const cy = row * cellSize + Math.floor(cellSize / 2);
  const { angle } = sobelAt(pixels, srcW, srcH, cx, cy);
  // Need to pass pixels, srcW, srcH into renderReplacement
  outputCtx.save();
  outputCtx.translate(x, y);
  outputCtx.rotate(angle);
  outputCtx.fillText(char, 0, 0);
  outputCtx.restore();
} else {
  outputCtx.fillText(char, x, y);
}
```

The `renderReplacement` signature already includes `pixels, srcW, srcH` (added in Task 2). The `sobelAt` function from Task 3 is already available.

- [ ] **Step 2: Verify — test character rotation**

Upload an image, enable Character Rotation toggle. Confirm:
- Characters rotate to follow edge directions
- Creates an organic, sketch-like feel
- Works with all replacement variants
- Also works with Contour mode (already implemented in Task 3)
- Toggling off returns to uniform orientation

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add character rotation based on gradient direction"
```

---

## Chunk 3: Video, Webcam, and Export

### Task 5: Video Upload and Playback

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add video loading and playback controls**

Add video controls HTML (initially hidden) between the canvas area and the settings drawer:

```html
<div id="videoControls" class="hidden px-4 py-2 bg-[#111] border-t border-[#222] flex items-center gap-3">
  <button id="playPauseBtn" class="text-white text-lg">&#9654;</button>
  <input type="range" id="seekBar" class="flex-1" min="0" max="100" step="0.1" value="0">
  <span id="timeDisplay" class="text-[#666] text-xs font-mono">0:00 / 0:00</span>
</div>
```

Replace the `loadVideo` stub from Task 2 with the real implementation:

```javascript
function loadVideo(file) {
  if (animFrameId) { cancelAnimationFrame(animFrameId); animFrameId = null; }
  stopWebcam();
  const video = document.getElementById('videoSource');
  video.src = URL.createObjectURL(file);
  video.onloadedmetadata = () => {
    state.source = 'video';
    state.sourceElement = video;
    stopWebcam();
    document.getElementById('emptyState').classList.add('hidden');
    showVideoControls();
    resizeCanvas();
    video.play();
    animFrameId = requestAnimationFrame(renderLoop);
  };
}

document.getElementById('videoInput').addEventListener('change', e => {
  if (e.target.files[0]) loadVideo(e.target.files[0]);
});
```

- [ ] **Step 2: Implement render loop for video**

Remove the `let animFrameId = null;` stub from Task 2 Step 1 (it's now defined here for real):

```javascript
// animFrameId was declared as stub in Task 2 — keep declaration, just reassign

let lastFrameTime = 0;
const targetFrameInterval = 1000 / 30; // target 30fps

function renderLoop(timestamp) {
  if (state.source !== 'video' && state.source !== 'webcam') {
    animFrameId = null;
    return;
  }

  // Skip frame if video is paused (avoid wasting CPU)
  const video = state.sourceElement;
  if (state.source === 'video' && video.paused && lastFrameTime > 0) {
    animFrameId = requestAnimationFrame(renderLoop);
    return;
  }

  // Frame skipping: only render if enough time has passed
  if (timestamp - lastFrameTime >= targetFrameInterval) {
    render();
    updateTimeDisplay();
    lastFrameTime = timestamp;
  }

  animFrameId = requestAnimationFrame(renderLoop);
}

function updateTimeDisplay() {
  if (state.source !== 'video') return;
  const video = state.sourceElement;
  const cur = formatTime(video.currentTime);
  const dur = formatTime(video.duration);
  document.getElementById('timeDisplay').textContent = `${cur} / ${dur}`;
  document.getElementById('seekBar').value = (video.currentTime / video.duration) * 100;
}

function formatTime(s) {
  const m = Math.floor(s / 60);
  const sec = Math.floor(s % 60).toString().padStart(2, '0');
  return `${m}:${sec}`;
}
```

- [ ] **Step 3: Wire up play/pause and seek controls**

```javascript
document.getElementById('playPauseBtn').addEventListener('click', () => {
  const video = state.sourceElement;
  // The render loop is already running (it keeps spinning even when paused),
  // so just play/pause the video — no need to restart the loop.
  if (video.paused) video.play();
  else video.pause();
});

document.getElementById('seekBar').addEventListener('input', e => {
  const video = state.sourceElement;
  video.currentTime = (e.target.value / 100) * video.duration;
  // Re-render and update display when seeking while paused
  if (video.paused) {
    video.onseeked = () => { render(); updateTimeDisplay(); video.onseeked = null; };
  }
});

// Replace the hideVideoControls stub from Task 2 with real implementations:
function showVideoControls() {
  document.getElementById('videoControls').classList.remove('hidden');
}
function hideVideoControls() {
  document.getElementById('videoControls').classList.add('hidden');
}
```

- [ ] **Step 4: Verify — upload a video**

Upload a video file. Confirm:
- Video plays with ASCII rendering in real-time
- Play/pause button works
- Seek bar scrubs through the video
- Time display updates
- All settings (variant, cell size, adjustments) affect video rendering
- Drag-and-drop also works for video files

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: video upload with real-time ASCII rendering and playback controls"
```

### Task 6: Webcam Support

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Implement webcam start/stop**

```javascript
async function startWebcam() {
  if (animFrameId) { cancelAnimationFrame(animFrameId); animFrameId = null; }
  stopWebcam();
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    const video = document.getElementById('videoSource');
    video.srcObject = stream;
    video.play();
    state.source = 'webcam';
    state.sourceElement = video;
    state.webcamStream = stream;
    document.getElementById('emptyState').classList.add('hidden');
    hideVideoControls();
    resizeCanvas();
    animFrameId = requestAnimationFrame(renderLoop);
  } catch (err) {
    console.error('Webcam access denied:', err);
  }
}

function stopWebcam() {
  if (state.webcamStream) {
    state.webcamStream.getTracks().forEach(t => t.stop());
    state.webcamStream = null;
  }
}

document.getElementById('webcamBtn').addEventListener('click', startWebcam);
```

- [ ] **Step 2: Ensure switching sources stops webcam**

Replace the `stopWebcam` stub from Task 2 with the real implementation above. The `loadImage()` and `loadVideo()` already call `stopWebcam()` and `cancelAnimationFrame()` (added in Task 2 and Task 5 respectively), so those calls now invoke the real function.

- [ ] **Step 3: Verify — test webcam**

Click webcam button. Confirm:
- Browser prompts for camera permission
- Live webcam feed renders as ASCII art
- All settings work in real-time
- Uploading an image/video stops the webcam
- No playback controls shown for webcam

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: webcam support with live ASCII rendering"
```

### Task 7: PNG Export

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Implement PNG export**

```javascript
function exportPNG() {
  if (!state.sourceElement) return;

  // For overlay mode, the output canvas already has the composite
  // For replacement mode, the output canvas has the ASCII on black
  outputCanvas.toBlob(blob => {
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'ascii-export.png';
    a.click();
    URL.revokeObjectURL(url);
  }, 'image/png');
}

document.getElementById('exportPngBtn').addEventListener('click', exportPNG);
```

- [ ] **Step 2: Verify — export an image**

Upload an image, click export PNG. Confirm:
- PNG file downloads
- Image matches what's on screen
- Works for both replacement and overlay modes

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: PNG export of ASCII canvas"
```

### Task 8: WebM Video Recording

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Implement video recording with MediaRecorder**

```javascript
let mediaRecorder = null;
let recordedChunks = [];

function startRecording() {
  if (!state.sourceElement) return;

  const stream = outputCanvas.captureStream(30);
  mediaRecorder = new MediaRecorder(stream, { mimeType: 'video/webm' });
  recordedChunks = [];

  mediaRecorder.ondataavailable = e => {
    if (e.data.size > 0) recordedChunks.push(e.data);
  };

  mediaRecorder.onstop = () => {
    const blob = new Blob(recordedChunks, { type: 'video/webm' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'ascii-recording.webm';
    a.click();
    URL.revokeObjectURL(url);
    state.recording = false;
    updateRecordButton();
  };

  mediaRecorder.start();
  state.recording = true;
  updateRecordButton();
}

function stopRecording() {
  if (mediaRecorder && mediaRecorder.state !== 'inactive') {
    mediaRecorder.stop();
  }
}

function toggleRecording() {
  if (state.recording) stopRecording();
  else startRecording();
}

function updateRecordButton() {
  const btn = document.getElementById('recordBtn');
  if (state.recording) {
    btn.classList.add('recording');
    btn.title = 'Stop Recording';
    // Swap SVG to stop icon (square)
    btn.innerHTML = '<svg width="24" height="24" viewBox="0 0 24 24" fill="currentColor"><rect x="6" y="6" width="12" height="12" rx="2"/></svg>';
  } else {
    btn.classList.remove('recording');
    btn.title = 'Record Video';
    // Swap SVG back to record icon (circle)
    btn.innerHTML = '<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="6"/></svg>';
  }
}

document.getElementById('recordBtn').addEventListener('click', toggleRecording);
```

- [ ] **Step 2: Add recording indicator CSS**

```css
@keyframes pulse-red {
  0%, 100% { box-shadow: 0 0 0 0 rgba(239, 68, 68, 0.7); }
  50% { box-shadow: 0 0 0 8px rgba(239, 68, 68, 0); }
}
.recording {
  background: #ef4444 !important;
  animation: pulse-red 1.5s infinite;
}
```

- [ ] **Step 3: Verify — record a video**

Upload a video or start webcam, click record. Let it run a few seconds, then click stop. Confirm:
- Record button pulses red while recording
- WebM file downloads on stop
- Recorded video contains the ASCII effect
- Button returns to normal state after stop

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: WebM video recording with MediaRecorder"
```

---

## Chunk 4: Polish

### Task 9: Final Polish and Edge Cases

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Handle canvas resize on window resize and drawer toggle**

Ensure `resizeCanvas()` is called on window resize and drawer toggle, and that it re-renders. For video/webcam, the render loop handles this. For images, call `render()` after resize.

- [ ] **Step 2: Update play/pause button icon**

```javascript
// In the video source's event listeners:
const video = document.getElementById('videoSource');
video.addEventListener('play', () => {
  document.getElementById('playPauseBtn').innerHTML = '&#9646;&#9646;'; // pause icon
});
video.addEventListener('pause', () => {
  document.getElementById('playPauseBtn').innerHTML = '&#9654;'; // play icon
});
video.addEventListener('ended', () => {
  document.getElementById('playPauseBtn').innerHTML = '&#9654;';
  // Stop recording if active before killing the render loop
  if (state.recording) stopRecording();
  if (animFrameId) { cancelAnimationFrame(animFrameId); animFrameId = null; }
});
```

- [ ] **Step 3: Performance — sample every Nth pixel for large cells**

For large cell sizes, sampling every pixel is wasteful. Sample a fixed number of points per cell:

```javascript
// In renderReplacement, replace the full-pixel sampling with:
const sampleStep = Math.max(1, Math.floor(cellSize / 4));
let r = 0, g = 0, b = 0, count = 0;
for (let dy = 0; dy < cellSize; dy += sampleStep) {
  for (let dx = 0; dx < cellSize; dx += sampleStep) {
    // ... same pixel sampling
  }
}
```

- [ ] **Step 4: Verify — full integration test**

Test the complete flow:
1. Open app — see empty state
2. Drag-drop an image — ASCII rendering appears
3. Change all settings — each affects output correctly
4. Switch to Contour — overlay mode works
5. Enable character rotation — characters rotate
6. Upload a video — real-time rendering with playback controls
7. Start webcam — live feed renders
8. Export PNG — file downloads correctly
9. Record WebM — file downloads correctly
10. Toggle settings drawer — canvas resizes appropriately

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: polish, performance optimization, and edge case handling"
```
