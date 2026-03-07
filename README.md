# PixelPeel

A single-file browser tool for removing solid-color backgrounds from images, with specialized support for LLM-generated pixel art.

**Everything lives in one `index.html` file.** Just open it in any modern browser — no build step, no server, no `npm install`. Works offline once CDN scripts are cached.

## Features

### Background Removal
- **Color picker** — click "Pick Color" then click anywhere on the source image to select the target background color. Auto-samples the top-left pixel on upload.
- **Tolerance slider** — controls how aggressively colors are matched (Euclidean distance in RGB space, mapped to 0–441).
- **Edge feathering** — soft alpha blending at foreground/background boundaries using both spatial distance (BFS from removed pixels) and color distance. Decontaminates RGB channels to remove background color bleed using alpha unmixing: `fg = (pixel - (1-α)·bg) / α`.

### Pixel Art Downscale
Designed for LLM-generated pixel art where each "art pixel" is rendered as an NxN block of real pixels.

- **Auto-detection** — uses autocorrelation of the color-difference signal to find the grid period. Parabolic interpolation around the autocorrelation peak gives sub-pixel precision. Supports fractional, non-square periods (e.g. 5.83 x 6.12). Fourier phase analysis at the detected frequency determines the grid offset.
- **Manual override** — separate Scale X/Y and Shift X/Y inputs accept decimal values for fine-tuning when auto-detect needs help.
- **Preview upscale toggle** — view the downscaled result at true size or scaled back up with nearest-neighbor interpolation.

### Cartoon Outline
Adds a 1px black border around the visible subject edges. Applied after downscale, so the outline is one true art pixel wide.

### Export
Downloads the processed result as a transparent PNG. When pixel art mode is active, always exports at the downscaled resolution — the preview upscale is for display only and does not affect the exported file.

## Tech Stack

All inline in a single HTML file — no build system, no `node_modules`:
- React 18 (UMD via CDN + Babel standalone for JSX)
- Tailwind CSS (CDN play script)
- Canvas API for all image processing

## Processing Pipeline

1. **Hard threshold** — classify each pixel as background (α=0) or foreground (α=255) by Euclidean RGB distance to the target color
2. **BFS edge distance map** — flood-fill from removed pixels to compute each kept pixel's distance to the nearest removed region, capped at the feather radius
3. **Edge feathering + decontamination** — for pixels within the feather band, compute soft alpha as `min(spatialAlpha, colorAlpha)` and recover the true foreground color via alpha unmixing
4. **Pixel art downscale** — sample from the center of each fractional grid cell (supports non-square, sub-pixel periods)
5. **Cartoon outline** — scan for visible pixels with at least one transparent 4-neighbor, paint black

PNG export always uses the downscaled canvas (no upscaling); preview optionally re-upscales with nearest-neighbor interpolation for display only.

## License

Licensed under the [European Union Public Licence v1.2](LICENSE) (EUPL-1.2).
