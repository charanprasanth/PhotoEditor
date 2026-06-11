# Darkroom — Photo Editor

A single-file, fully client-side photo editor built as an experiment with Claude. Everything stays on your device.

## Features

### 🎨 **Looks**
Nine curated presets: Original, Pop, Golden, Vintage, Faded Film, Noir, Cool, Drama, Soft Glow. Live thumbnails render from your actual photo.

### 🎚️ **Tune**
Global adjustments with independent sliders:
- Exposure, Brightness, Contrast, Saturation
- Warmth, Tint, Highlights, Shadows

### 🎯 **Selective**
Snapseed-style control points for local edits. Tap the photo to drop a point (up to 8), then adjust:
- **Size** — radius of the adjustment zone (smooth radial falloff)
- **Brightness** — lighten or darken locally
- **Contrast** — tighten or soften local contrast
- **Saturation** — color punch or desaturate specific areas

Drag points to reposition. Points auto-remap when you rotate, flip, or crop.

### 🩹 **Heal**
Clone-based healing brush. Paint over blemishes or unwanted objects; on release, the tool:
1. Scans 8 candidate patches around your stroke
2. Scores each by boundary match (ring SSD)
3. Blends the best one in with a feathered edge

Works best on skin, sky, and uniform texture. Adjustable brush size, undo last heal, clear all heals.

### ✨ **Details**
- **Structure** — unsharp mask (sharpening via 3×3 convolution)
- **Soft Blur** — gaussian-ish blur for glow effects

### 🌀 **Effects**
- **Vignette** — darkens edges
- **Grain** — adds film grain
- **Fade** — bleaches towards white (fades contrast)

### 🔲 **Crop**
- Free or locked aspect (1:1, 4:3, 3:2, 16:9, original)
- Rotate 90° CW/CCW, flip horizontal/vertical
- Straighten slider with auto-fill scaling
- Rule-of-thirds grid overlay

### 💾 **Export**
- Format: JPEG, PNG, WebP
- Quality slider (40–100%)
- Full-resolution output (native image size)
- Renders at actual pixels; preview downscaled to 1280px for performance

### 🔄 **Undo/Redo**
Full history stack. Keyboard: `Ctrl+Z` / `Ctrl+Shift+Z` (or `Cmd` on Mac).

### 👁️ **Compare**
Hold the compare button to see the original; release to return to edits.

---

## How to Use

### Opening a Photo
- Click **Choose a photo** or drag & drop anywhere on the page
- Supported: JPG, PNG, GIF, WebP, and other browser-native formats

### Editing Workflow
1. **Pick a Look** — start with a preset or start from Original
2. **Tune** — adjust exposure, contrast, warmth globally
3. **Selective** — tap to add local control points, adjust individually
4. **Heal** — paint over blemishes
5. **Crop** — frame and rotate
6. **Details** — sharpen or soften
7. **Effects** — add grain, vignette, or fade
8. **Export** — download at full resolution

### Keyboard Shortcuts
| Key | Action |
|-----|--------|
| `Ctrl+Z` / `Cmd+Z` | Undo |
| `Ctrl+Shift+Z` / `Cmd+Shift+Z` | Redo |
| `Delete` (in Selective mode) | Remove selected control point |

---

## Technical Architecture

### Rendering Pipeline
The editor runs a 5-pass rendering pipeline per frame:

1. **GPU Filters** — brightness, contrast, saturation, blur via `canvas.filter` (hardware-accelerated)
2. **Global Pixel Ops** — warmth, tint, highlights/shadows, fade, grain (JavaScript pixel loop)
3. **Selective Control Points** — local brightness/contrast/saturation with radial falloff
4. **Sharpen** — 3×3 convolution unsharp mask
5. **Vignette** — radial gradient overlay

### Canvas Architecture
- **Preview Mode** — renders at up to 1280px on the longest edge for responsiveness
- **Export Mode** — full-resolution pipeline using the original image
- **Working Canvas** — holds heals baked into the original, fed to subsequent passes

### Heal Implementation
Heals are stored in **original image coordinates**:
- Screen taps are inverse-mapped through the full transform matrix (rotation, flip, straighten, crop)
- Strokes are baked into a full-resolution working canvas
- On export, heals apply at native pixels
- Undo/redo deterministically replays strokes

Patch selection algorithm:
- Samples candidate patches in 8 directions around the stroke
- Scores each by boundary color matching (sum of squared differences on a ring)
- Picks the best match and blends with a feathered edge (quadratic falloff)

### Selective Points
Control points live in **normalized frame space** (0–1 in both axes):
- Adjusted automatically when you rotate/flip/crop the image
- Apply via a radial Gaussian-ish envelope (smooth falloff to 0 at the boundary)
- Each point stores independent brightness, contrast, saturation, and size

### Transform State
Tracked separately from adjustments:
- **Rotation** — 90° increments (stored as `rot` in degrees)
- **Flips** — boolean flags (horizontal, vertical)
- **Straighten** — continuous angle (–45° to +45°)
- **Crop** — normalized rectangle {x, y, w, h} applied after all geometric transforms

---

## Performance Notes

- **Preview** downscaled to 1280px max for fast frame rates
- **Selective points & heal** apply only during preview and export (no intermediate compositing)
- **Undo/redo** stores full state snapshots (up to 60 history entries kept; oldest discarded)
- **Heal strokes** processed on release, not live-painting (async would be better at scale)

---

## Limitations

### Heal Brush
Works best on:
- ✅ Skin, sky, water, uniform textures
- ❌ Large objects crossing structured backgrounds (e.g., person in front of a fence)

Reason: patch-clone blending can show seams when the surrounding area is patterned. A real solution would need **PatchMatch-style inpainting**, which is computationally expensive in single-threaded JavaScript.

### Selective Points
- Max 8 points per image (reasonable UI limit; can be raised)
- Points don't "follow" objects if you crop (they stay anchored to the frame)
- No mask visualization beyond the dashed ring

### Geometric Transforms
- Straighten is applied **before** crop, so very large angles may introduce letterboxing
- No perspective correction (lens distortion, tilt-shift)

### Color Space
- All ops are in sRGB; no ICC profile support
- Highlights/shadows use simple luminance-based masks, not perceptual color spaces

---

## Built With

**Claude** — an AI assistant by Anthropic. This editor was created as an experiment in how far a single-file, client-side photo app can go without server dependencies or specialized libraries. No frameworks, no build step, no npm packages — just HTML, CSS, and vanilla JavaScript using the Canvas 2D API.

Reasoning behind this approach:
- **Portability** — save one .html file, open in any browser
- **Privacy** — all processing happens locally; nothing leaves your device
- **Learning** — demonstrates rendering pipelines, geometric transforms, and healing algorithms from first principles
- **Simplicity** — no deployment, no Node.js, no dependencies to manage

---

## Browser Support

Works in all modern browsers with Canvas 2D support:
- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Mobile browsers (iOS Safari, Chrome Android)

File upload and drag-drop require File API support (broadly available).

---

## What's Inside

| File | Size | Role |
|------|------|------|
| `photo-editor.html` | ~37 KB | Single-file editor: HTML, CSS, JavaScript, Canvas rendering |

---

## Try It

1. Open `photo-editor.html` in a browser
2. Drag a photo onto the page or click **Choose a photo**
3. Experiment: pick a look, add selective points, try the healing brush
4. Export when happy

No installation, no account, no telemetry.

---

## Future Ideas

Not implemented, but could be added:

- **Mask visualization** — see the falloff zone for each control point
- **Curves** — parametric tone curve adjustment (currently just highlights/shadows)
- **Liquify** — local warping/smudging
- **Inpainting** — PatchMatch-based content-aware fill (slow, but much better heals)
- **RAW support** — would need a TIFF/DNG decoder (large)
- **Presets as files** — save/load custom adjustment stacks
- **Layers** — blend multiple edits non-destructively (adds complexity)
- **AI upscaling** — would need a model (ONNX.js, TensorFlow.js)

---

## License

Public domain. Use, modify, remix as you like.

---

**Questions?** This is a learning project. The code is intentionally readable — feel free to fork, experiment, or build on it.
