# Glyph Studio

Real-time ASCII art renderer with layers, masks, and compositing effects.

## Features

- **Layer system** — multiple independent ASCII layers with opacity and blend mode control
- **Mask editing** — lasso tool with composition modes (add, subtract, intersect via Shift/Alt modifiers), feathering, and invert
- **Character variants** — swap between ASCII character sets (Standard, Blocks, Braille, Kanji, and more)
- **Contour mode** — Sobel edge detection overlay with spread and sensitivity controls
- **Bloom & glow** — post-processing bloom with multiple styles (soft, sharp, anamorphic) and tunable radius/intensity
- **Post-processing** — per-layer paint-spread, organic edges, splatter, and softness effects
- **Video & webcam** — upload video files or use your webcam as a live ASCII source with full playback controls
- **Export** — PNG (with optional transparency) and WebM video recording
- **Projects** — save and restore layer configurations to local storage

## Mask Composition

Hold modifier keys when starting a lasso stroke to combine with the existing mask:

| Modifier | Mode |
|----------|------|
| None | New (replace) |
| Shift | Add (union) |
| Alt/Option | Subtract (cut hole) |
| Shift+Alt | Intersect |

The toolbar shows the active mode and a GUIDES toggle to show/hide selection outlines.

## Usage

Open `index.html` in a modern browser. No build step or server required.

Upload an image or video, or start your webcam, and adjust settings in the sidebar.

## Tech Stack

- Vanilla JavaScript (single `index.html`)
- [Tailwind CSS](https://tailwindcss.com/) via CDN
- IBM Plex Mono typeface
