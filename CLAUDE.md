# PixelPeel - Project Guide

## What This Is
Single-file React app (`index.html`) that removes solid-color backgrounds from images. Everything is inline — no build system, no node_modules.

## Architecture
One `index.html` file containing:
- CDN dependencies: React 18 (UMD + Babel standalone), Tailwind CSS play script
- Inline SVG icons (no Lucide CDN)
- All logic in a single `<script type="text/babel">` block

## Key Code Sections (in order within the file)
1. **SVG icon components** — PeelIcon, PipetteIcon, etc.
2. **`detectPixelGrid()`** — autocorrelation-based grid detection for pixel art (fractional, non-square periods)
3. **`App` component** — all state, effects, handlers, and JSX
4. **Processing pipeline** (in the main `useEffect`):
   - Pass 1: hard threshold (color distance → alpha 0/255)
   - Pass 2: BFS edge distance map
   - Pass 3: edge feathering + RGB decontamination
   - Pass 4: pixel art downscale (fractional grid sampling)
   - Pass 5: cartoon outline (1px black border)
   - Pass 6: autocrop (trim transparent edges to bounding box)
   - Final: write to export canvas + preview canvas

## State Model
- `targetColor`, `tolerance`, `feather` — background removal params
- `pixelArt`, `gridInfo`, `manualScaleX/Y`, `manualOx/Oy`, `previewUpscale` — pixel art downscale
- `outline` — cartoon outline toggle
- `autocrop` — trim transparent edges toggle
- Three canvas refs: `hiddenCanvas` (source read), `exportCanvas` (true-size output), `previewCanvas` (display)

## When Making Changes
- All code is in one file — search within `index.html`
- The processing pipeline is order-dependent (passes 1-6 must stay sequential)
- `effectiveGrid` merges auto-detected `gridInfo` with manual overrides — check both paths
- The `useEffect` dependency array must include all params that affect output
- Export uses `exportCanvasRef` (downscaled when pixel art is active), preview uses `previewCanvasRef` (optionally upscaled for display only)
