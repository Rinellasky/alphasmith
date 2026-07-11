# AlphaSmith ⚒️

**Free procedural alpha brush generator for Blender, Unreal Engine, and Photoshop.**

Forge grayscale alpha masks / height stamps entirely in your browser — no signup, no watermark, no server. Pick a preset, tweak sliders, download as 8-bit or true 16-bit PNG.

## Features

- 13 procedural presets: Clouds, Ridged Rock, Cobblestone, Cracks, Veins, Marble Swirl, Grunge, Scratches, Stipple Dots, Craters, Tree Bark, Chainmail, Damage Decal
- **Tileable / seamless mode** — periodic noise + toroidal stamping so brushes repeat with zero seams (2×2 tile preview built in)
- Seeded generation — the same seed + settings always reproduces the same brush
- Live sculpt-relief preview (see the alpha as if stamped into clay)
- Universal controls: scale, detail, contrast, brightness, edge falloff (for round brush tips), invert
- Export 512 / 1024 / 2048 / 4096 px
- **8-bit PNG** for Photoshop brushes and Blender sculpt alphas
- **True 16-bit grayscale PNG** for Unreal landscape sculpting and Blender displacement (no stair-stepping) — custom in-browser PNG encoder
- 100% client-side: a single `index.html`, no dependencies, no build step. Works offline.

## Running it

Open `index.html` in any modern browser. That's it.

To host it, serve the file anywhere static files go — **GitHub Pages** is the natural free option:

1. Push this repo to GitHub
2. Repo → Settings → Pages → Deploy from branch → `main` / root
3. Your site is live at `https://<username>.github.io/alphasmith/`

## Configuration

All personal links live in one `CONFIG` block near the top of the `<script>` section in `index.html`:

```js
const CONFIG = {
  siteName:  'AlphaSmith',
  patreonUrl:'https://www.patreon.com/rinellasky',
  discordUrl:'https://discord.gg/a8sjdRFjz',
};
```

## Architecture (for contributors)

Everything is in `index.html`:

- **Engine** — lives in a `<script type="text/plain" id="engineSrc">` block and runs in a Web Worker (built from a Blob), so 4096×4096 renders never freeze the UI. Contains seeded simplex noise, fBm, ridged multifractal, Worley/cellular noise, and stamp-based generators (scratches, dots, craters), plus the post pipeline (normalize → contrast/brightness → radial falloff → invert).
- **App** — preset gallery with live thumbnails, debounced preview rendering, relief shading, and the exporters. 8-bit goes through canvas `toBlob`; 16-bit is a hand-rolled PNG encoder (grayscale, bit depth 16) using `CompressionStream` with a stored-block zlib fallback.

## Roadmap ideas

- Batch "brush pack" ZIP export
- Photoshop `.abr` packaging
- Shareable settings URLs
- Community preset gallery

## License

MIT — free for any use. If AlphaSmith saves you time, consider [supporting on Patreon](https://www.patreon.com/rinellasky) or joining the [guild Discord](https://discord.gg/a8sjdRFjz).
