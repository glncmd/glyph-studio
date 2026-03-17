# Glyph Studio

Real-time ASCII art renderer with layers, effects, and AI-powered object selection.

## Features

- **Layer system** — multiple independent ASCII layers with opacity and blend control
- **Mask editing** — paint masks per layer with feathering and invert; use MobileSAM for AI-assisted object selection
- **Character variants** — swap between ASCII character sets (Standard, Blocks, Braille, Kanji, and more)
- **Contour mode** — Sobel edge detection overlay with spread and sensitivity controls
- **Bloom & glow** — post-processing bloom with multiple styles (soft, sharp, anamorphic) and tunable radius/intensity
- **Video & webcam** — upload video files or use your webcam as a live ASCII source with full playback controls
- **Export** — PNG (with optional transparency) and WebM video recording

## Usage

Open `index.html` in a modern browser. No build step or server required.

Upload an image or video, or start your webcam, and adjust settings in the sidebar.

## Tech Stack

- Vanilla JavaScript (single `index.html`, ~2000 lines)
- [Tailwind CSS](https://tailwindcss.com/) via CDN
- [ONNX Runtime Web](https://onnxruntime.ai/) + [MobileSAM](https://github.com/ChaoningZhang/MobileSAM) for AI object selection
- IBM Plex Mono typeface
