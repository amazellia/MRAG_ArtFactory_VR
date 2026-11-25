# MRAG Art Factory — VR Model Viewer

A minimal, fast static web viewer for the Maitland Regional Art Gallery (MRAG) Art Factory redesign model, built by ARBE 3030 students at the University of Newcastle's PixelSpace Lab.

## What's here

- `index.html` — the entire viewer (Three.js via CDN, no build step)
- `models/render.glb` — the compressed model actually served to browsers (**10.4 MB**)
- `assets/logo.png` — PixelSpace Lab wordmark shown in the top-left corner

## Why it's this size

The original SketchUp export was 117 MB: a ~2.3M-vertex mesh with 51 embedded textures (37 MB). That was reduced to 10.4 MB by:

1. Resizing every embedded texture to a 1024px-longest-side JPEG (37 MB → ~3 MB).
2. Simplifying the mesh with `meshoptimizer` (2.3M → 0.85M vertices, ~63% reduction, visually lossless at web viewing distances).
3. Quantizing attributes and applying `EXT_meshopt_compression`, decoded client-side via a small WASM decoder instead of parsing raw float32 buffers.

Unlike a photogrammetry scan, this is a proper SketchUp export with real materials, normals and real-world scale, so the viewer renders it lit (hemisphere + directional light) rather than unlit, and applies no scale correction or axis rotation.

To reproduce this from a fresh SketchUp `.glb` export:

```bash
npm install @gltf-transform/cli
node node_modules/@gltf-transform/cli/bin/cli.js optimize textures-resized.glb models/render.glb \
  --texture-compress false --compress meshopt --simplify-ratio 0.4 --simplify-error 0.0015
```

(textures were pre-resized with Pillow before this step; see the MRAG × PixelSpace Lab article for the full pipeline.)

## Running locally

```bash
python -m http.server 8080
```

Then open `http://localhost:8080`.

## Hosting

Static only — upload `index.html`, `models/`, and `assets/` as-is to GitHub Pages, Cloudflare Pages, Netlify, or Vercel. No server-side code is required.

## Controls

- Drag to orbit, scroll to zoom, right-drag to pan (touch: drag to orbit, pinch to zoom, two-finger drag to pan)
- Bottom-right buttons: credits, reset view, toggle wireframe, VR (hidden unless the browser reports a connected VR headset), fullscreen

## VR mode

The "VR" button only appears when `navigator.xr.isSessionSupported("immersive-vr")` resolves true — in practice, a WebXR-capable headset browser (e.g. Meta Quest Browser) visiting the page over https.

Point and pinch (bare-hand tracking) or pull the controller trigger to drop a teleport reticle on the floor/ground plane; release to jump there.

## Embedding on another site

```html
<iframe
  src="https://your-static-host.example/mrag-art-factory-vr/"
  style="width: 100%; aspect-ratio: 16 / 9; border: 0;"
  loading="lazy"
  allow="fullscreen"
  title="MRAG Art Factory VR model viewer"
></iframe>
```

- `allow="fullscreen"` is required for the fullscreen button to work inside the iframe.
- The viewer must be served over http/https (not `file://`) for the `.glb` fetch to succeed.
