# GLYPH_STUDIO

`GLYPH_STUDIO` is a browser-based ASCII art playground for turning images, video, and live camera input into layered, stylized output. The app ships as a tiny static project: one HTML app file plus a generated dense-glyph data file used for high-density character rendering.

## What It Does

- Renders uploaded images, video files, and live webcam input as ASCII.
- Stacks multiple ASCII layers with per-layer visibility, opacity, and blend modes.
- Switches between variants including Standard, Dense, Minimal, Blocks, Braille, Technical, Matrix, Hatching, Contour, Outline, and Highlight.
- Adds lasso masks with replace, add, subtract, and intersect behavior plus invert and feather controls.
- Applies layer effects including bloom, grain, ink bleed, chromatic aberration, pixelate, and multiple blur types.
- Embeds word artifacts with adjustable distribution, density, and coherence.
- Saves projects to local storage and exports PNG, transparent PNG, and WebM recordings.

## Project Structure

- `index.html` - main application UI and rendering pipeline
- `dense-glyph-paths.js` - generated outline data for dense glyph rendering

## Running Locally

Open `index.html` in a modern desktop browser.

For image and video uploads, opening the file directly is enough. Webcam access may require serving the folder from `localhost` because browsers often restrict camera APIs to secure contexts.

## Mask Shortcuts

When starting a lasso stroke:

- No modifier: replace the current mask
- `Shift`: add to the current mask
- `Alt` / `Option`: subtract from the current mask
- `Shift` + `Alt` / `Option`: intersect with the current mask

## Notes For GitHub

- `dense-glyph-paths.js` is a required runtime asset and should be committed.
- `.dense-glyph-paths.json` and `.ibmplexmono-glyphs.ttx` are local generation artifacts and are ignored.
- There is no build step or package manager setup in this repo right now.

## Stack

- Vanilla JavaScript
- HTML canvas
- [Tailwind CSS](https://tailwindcss.com/) via CDN
- Space Grotesk, JetBrains Mono, Inter via Google Fonts
