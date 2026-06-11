# Darkroom — A Photo Editor in One HTML File

> A Snapseed-style photo editor that runs entirely in your browser.
> No backend, no build step, no dependencies. Built as an experiment with Claude.

**[→ Live demo](https://charanprasanth.github.io/PhotoEditor/)**

---

## What it does

| Tool | What you can do |
|------|-----------------|
| **Looks** | 9 one-tap presets — Pop, Golden, Vintage, Faded Film, Noir, Cool, Drama, Soft Glow |
| **Tune** | Exposure, brightness, contrast, saturation, warmth, tint, highlights, shadows |
| **Selective** | Tap to drop up to 8 control points. Adjust brightness / contrast / saturation locally with smooth radial falloff |
| **Heal** | Brush over a blemish or object. On release it samples 8 candidate patches, scores boundary match, blends the best one |
| **Details** | Structure (unsharp mask) and soft blur |
| **Effects** | Vignette, grain, fade |
| **Crop** | Free or locked aspect (1:1, 4:3, 3:2, 16:9, original), rotate 90°, flip, straighten ±45° |
| **Export** | JPEG / PNG / WebP at full original resolution |
| **History** | Full undo / redo, hold-to-compare with original |

Everything is local. Nothing uploads. No telemetry.

---

## How it's built

> This project was built as an experiment with **[Claude](https://claude.ai)** — testing how far an AI model can take a single-file, dependency-free web app. The entire codebase is one HTML file with vanilla JavaScript and Canvas 2D — no React, no npm, no build pipeline.

### Architecture

**5-pass rendering pipeline** for each preview frame:
1. **GPU filters** — brightness, contrast, saturation, blur via hardware-accelerated `canvas.filter`
2. **Global pixel ops** — warmth, tint, highlights/shadows, fade, grain
3. **Selective control points** — local adjustments with radial Gaussian falloff
4. **Unsharp mask** — 3×3 convolution for sharpening
5. **Vignette** — radial gradient overlay

**Heal brush** stores strokes in **original image coordinates**, so they survive any later crop or rotation and apply at native pixels on export. Patch selection samples 8 candidate offsets around the stroke and scores each by ring-SSD boundary match.

**Selective points** live in normalized frame space and are remapped through every geometric transform (90° rotation, flip, crop).

**Preview** renders downscaled to 1280px for responsiveness; **export** re-runs the full pipeline at native resolution so the result matches exactly.

---

## Try it locally

```bash
git clone https://github.com/charanprasanth/PhotoEditor.git
cd PhotoEditor
open index.html        # macOS
# or just double-click the file
```

That's the whole install. There is no package.json.

---

## Project structure

```
PhotoEditor/
├── index.html        # the entire editor — HTML, CSS, JS, all in one file
└── README.md         # this
```

---

## Keyboard shortcuts

| Key | Action |
|-----|--------|
| `Ctrl/⌘ + Z` | Undo |
| `Ctrl/⌘ + Shift + Z` | Redo |
| `Delete` | Remove selected control point (in Selective mode) |
| `?` | Toggle help panel |
| `Esc` | Close help panel |

---

## Known limitations

These are honest gaps, not roadmap promises:

- **Heal on patterned backgrounds** — patch-clone blending shows seams when surroundings are structured (e.g. a person in front of a fence). A real fix needs PatchMatch-style inpainting, which would be slow in single-threaded JS.
- **No layers, masks, or curves** — global tone curves and non-destructive layers aren't there. The selective tool covers most local edits in practice.
- **sRGB only** — no ICC profile handling, no wide-gamut color.
- **No RAW** — would need a TIFF/DNG decoder (large bundle, defeats the "one HTML file" goal).
- **8 selective points max** — soft UI cap, raisable in code.

---

## Browser support

Tested in Chrome 90+, Firefox 88+, Safari 14+, and mobile Safari / Chrome. Requires `Canvas 2D` and the File API, both of which are essentially universal now.

---

## License

MIT. Use, fork, remix.

---

## Credits

- **Built with [Claude](https://claude.ai)** by Anthropic, as a test of single-file AI-assisted web app development
- Image-processing algorithms (unsharp mask, vignette, patch-based healing) implemented from first principles
- Inspired by Snapseed's interaction model