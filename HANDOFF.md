# AlphaSmith — Project Handoff

*Written 2026-07-11 by the previous Claude session. Read this fully before doing anything.*

## What this project is

**AlphaSmith** — a free, client-side alpha-brush / height-mask generator website for game artists (Blender sculpting, Unreal landscapes, Photoshop/Krita brushes). Built for Rin Ella (rinellasky@braindeadguild.com). Free forever, donation-supported.

- **Patreon:** https://www.patreon.com/rinellasky (handle: @rinellasky)
- **Discord (game guild):** https://discord.gg/a8sjdRFjz
- Both links live in ONE `CONFIG` object near the top of the `<script>` section in `index.html`. Never hardcode them elsewhere.

## Current state — v0.1, working and verified

The entire product is a **single self-contained `index.html`** (~35 KB, no dependencies, no build step, works offline from `file://`). Repo also contains README.md, LICENSE (MIT), .gitignore, this file. Git history: 2 commits on `main` (scaffold+v0.1, then 3 new presets).

**Verified by headless Chromium (Playwright) — all of this was actually run and observed, not assumed:**

- 13 presets all render: Clouds, Ridged Rock, Cobblestone, Cracks, Veins, Marble Swirl, Grunge, Scratches, Stipple Dots, Craters, Tree Bark, Chainmail, Damage Decal — each in 4–235 ms at 512px preview
- 8-bit PNG export via canvas works (verified with PIL: valid RGBA PNG)
- **16-bit grayscale PNG export works** — hand-rolled PNG encoder (IHDR bitdepth 16 colortype 0, CRC32, CompressionStream deflate with stored-block zlib fallback). PIL verified: mode I;16, 58k+ unique gray levels at 1024px (true 16-bit, not upscaled 8-bit)
- Export sizes 512/1024/2048/4096 (4096 16-bit ≈ tens of MB, takes seconds — that's expected)
- Patreon/Discord hrefs wired correctly; zero console errors

## Architecture (know before editing)

- **Generation engine** lives in `<script type="text/plain" id="engineSrc">` and runs in a **Web Worker** built from a Blob of that block's textContent. All heavy pixel work happens there; main thread never blocks.
- Engine contents: `mulberry32` seeded RNG, `hash2`, seeded 2D simplex noise, `makeFBM`, Worley (F1/F2), `fillField` per-pixel presets, `stampMax`/`stampRing` stamp-based presets (scratches/dots/craters/chainmail/damage), then `post()` = normalize → contrast/brightness → radial edge falloff → invert → clamp.
- Fields are Float32Array 0..1, transferred back via postMessage transferables.
- Preview = 512px debounced (90 ms); thumbnails = 96px with fixed seed 99; two preview modes: raw Alpha and "Sculpt preview" (relief shading computed on main thread from the last field).
- Params per preset: seed, scale, detail, pA, pB (0..1, labeled per preset in `PRESET_META`), contrast, brightness, falloff, invert. Amplitude never matters pre-normalize — `post()` min/max normalizes, so presets can be sloppy about output range.
- **Worker code block must never contain the literal string `</script>` or backtick issues — it's plain JS, no template literals in there.**

### Preset tuning history (don't regress these)
- Chainmail took 3 iterations: current geometry is colW=cell, rowH=cell*0.72, R=cell*0.54, rings squashed vertically (asp=1.25 in stampRing), alternate rows at 0.84 intensity. More overlap than this reads as "lattice", less reads as "separate rings".
- Damage Decal: core = one soft dab R*0.5 + ~20 overlapping dabs within R*0.34 so they merge into one ragged splat (distinct visible circles = wrong). Cracks evenly distributed by angle with jitter, 35% are stubs, taper to points.
- Bark: anisotropic ridged noise (y scaled ×0.22) with meandering warp — considered done, looks good.

## Where things live

- **User's PC:** project should be at `c:\appdev\alphasmith\` — **status at handoff: NOT yet confirmed placed there.** User was extracting the zip manually (or may ask you to do it).
- **Desktop bridge:** repeatedly probed via ToolSearch in the previous session and NEVER connected (session was started from mobile). If the new session starts from the desktop app, `mcp__remote-devices__*` tools may now exist — probe once before assuming either way. If connected, prefer writing files directly to `c:\appdev\alphasmith`.
- **GitHub:** NOT yet created/pushed. Plan: user creates **private** repo `alphasmith` under their account, then from `c:\appdev\alphasmith`:
  `git init -b main && git add -A && git commit -m "AlphaSmith v0.1" && git remote add origin https://github.com/USERNAME/alphasmith.git && git push -u origin main`
  Later: GitHub Pages (Settings → Pages → deploy from `main`/root) when they're ready to go public.
- **This session's container is gone** — the new session cannot read the old `/root/alphasmith`. The source of truth is the `alphasmith.zip` the user has (delivered twice in the old chat; latest version includes all 13 presets + this handoff). **If you need to edit code, ask the user to attach `alphasmith.zip` (or `index.html`) to the chat first.**

## Testing recipe (reuse this)

Playwright with the pre-installed Chromium (`executablePath: '/opt/pw-browsers/chromium'`), load `file://.../index.html`, then: wait for `#status` to contain "ms" → click each `.preset[data-id=…]` and confirm status updates → check preview canvas pixel min/max spread → `waitForEvent('download')` on `#dl8`/`#dl16` clicks → verify PNGs with PIL (`np.unique` count > 256 proves 16-bit). Zero tolerance for console errors.

## Agreed roadmap (user has NOT prioritized these yet)

Seamless/tileable mode (periodic noise), batch "brush pack" ZIP export, Photoshop `.abr` packaging, shareable settings URLs, community preset gallery.

## User preferences observed

- Wants things easy, fast, reliable; non-expert with git — give exact copy-paste commands, expect to debug their error messages patiently.
- Site must stay free, no signup, no watermark; donation nudge exists (toast after 2nd download) — keep it gentle.
- Name "AlphaSmith" was chosen after checking for conflicts; folder name is `alphasmith`.

## Immediate next steps (in order)

1. Probe for the desktop bridge (`remote-devices` tools). If present, verify/place `c:\appdev\alphasmith`.
2. Confirm the site runs from the user's disk (double-click `index.html`).
3. GitHub: private repo created → pushed. Handle auth errors (they'll likely use the browser sign-in flow).
4. Then ask which roadmap item they want next.
