# ⚡ FPS Forge — Cognix Benchmark

**Live demo:** [abhiraj1121.github.io/fps](https://abhiraj1121.github.io/fps)

A single-file, zero-dependency (CDN-only) browser benchmark suite for stress-testing GPU and CPU performance directly in the browser. No installs, no build step — open `index.html` and forge.

![status](https://img.shields.io/badge/build-single--file-00f0ff) ![deps](https://img.shields.io/badge/deps-three.js%20r128-ff2ee6) ![license](https://img.shields.io/badge/license-MIT-ffb020)

---

## Overview

FPS Forge is a cyberpunk-themed hardware benchmarking dashboard that pushes a machine's GPU and CPU through four distinct workloads while tracking frame timing in real time. It's built for developers, gamers, and hardware reviewers who want a fast, shareable, no-install way to sanity-check a device's rendering performance — straight from a GitHub Pages URL.

Everything — rendering engine, shaders, worker threads, UI, and export logic — lives in one `index.html` file. The only external dependency is Three.js, pulled from cdnjs.

## Key Features

- **Real-time HUD** — giant color-coded FPS counter (green ≥60, yellow 30–59, red <30), live frame-time readout, and a glowing rolling sparkline graph of FPS/frame-time history.
- **Five stress presets:**
  | Preset | Engine | What it does |
  |---|---|---|
  | 🟢 Light | Canvas 2D | Thousands of neon particles with velocity-based collision physics |
  | 🔵 Medium | Three.js `InstancedMesh` | 50,000+ (up to 300k) low-poly icosahedra, per-instance rotation, 3 dynamic point lights |
  | 🔴 Ultra / GPU Meltdown | Raw GLSL fragment shader | Real-time raymarched SDF scene (torus fractal + sphere), additive glow, particle storm overlay |
  | 🟠 CPU Burn | Web Workers | Spawns one worker per logical core running prime-sieve + matrix-multiplication loops to saturate multithreaded CPU throughput |
  | ⚠️ System Overload | Shader + Workers combined | Runs a second raymarched fractal scene *and* the full CPU worker pool simultaneously to simulate worst-case concurrent load |
- **Live tunables** — object/particle count multiplier (0.1×–4×), render resolution scale (0.5×–2.0×), and a frame-rate cap (uncapped / 30 / 60 / 144fps) — all hot-swappable mid-run.
- **Composite Benchmark Score** — a normalized 0–999 score with an S/A/B/C/D grade, weighted toward average FPS, 1% low, and 0.1% low, adjusted for consistency; shown as an animated glowing progress ring.
- **Stability detection** — automatic stutter-event counting (frame spikes >2.5× the rolling baseline) with a live STABLE / MODERATE / UNSTABLE tag.
- **Run history log** — the last 5 completed benchmark runs are kept in-session with preset, multiplier, resolution, and average FPS for quick comparison.
- **One-click summary copy** — formats the current run's stats as shareable plain text on the clipboard.
- **GPU/system detection** — reads `WEBGL_debug_renderer_info` for unmasked renderer/vendor strings, plus logical core count and display metrics.
- **Timed benchmark runs** — 15s / 30s / 60s / unlimited, with automatic stop and stat rollup.
- **Export** — full JSON report (raw samples + computed stats) or a rendered PNG scorecard, both downloadable client-side.
- **Keyboard shortcuts** — `Space` start/stop, `R` reset, `F` fullscreen, `1`–`5` to switch presets.
- **Fullscreen mode**, glassmorphic cyberpunk UI, noise + scanline overlay, fully responsive down to mobile.

## Stress Test Methodology

FPS Forge measures frame delta time on every `requestAnimationFrame` callback and derives:

- **Current FPS** — `1000 / frameDeltaMs`, sampled every rendered frame.
- **Avg FPS** — arithmetic mean across the full sample window.
- **Min / Max FPS** — extremes of the sample set for the run.
- **1% Low** — the 99th-percentile-worst frame rate (average of the bottom 1% of sorted samples), the industry-standard metric for perceived stutter, distinct from a raw average.
- **0.1% Low** — bottom 0.1% of sorted samples, exposing rare but severe frame-time spikes that 1% lows can mask.
- **Frame Time (ms)** — derived as `1000 / avgFPS`, shown alongside the live per-frame delta.

**Workload design:**
- *GPU stress* (Medium/Ultra presets) intentionally maximizes draw calls, matrix updates, and per-pixel shader cost (raymarch step count scales with the multiplier slider) to surface thermal throttling and driver-level frame pacing issues.
- *CPU stress* (CPU Burn) offloads trial-division primality testing and small dense matrix multiplications onto one `Worker` per logical core (`navigator.hardwareConcurrency`), each running a self-yielding 16ms work loop so the main thread — and therefore the FPS measurement itself — stays responsive as an independent signal of scheduler contention.
- The **resolution scale** slider directly multiplies the WebGL renderer's effective pixel ratio, isolating fill-rate-bound bottlenecks from geometry/CPU-bound ones.

Because everything runs in a single browser tab against the compositor's actual paint cycle, the FPS numbers reflect real-world rendering pressure rather than a synthetic offscreen benchmark.

## Usage

1. Open the [live demo](https://abhiraj1121.github.io/fps) or `index.html` locally.
2. Pick a stress preset from the left panel.
3. Adjust the object/particle multiplier and resolution scale as needed.
4. Choose a run duration and hit **Start Benchmark**.
5. Watch live metrics and the sparkline; export a JSON or PNG report when done.

## Tech Stack

- Vanilla HTML/CSS/JS — no build tools, no bundler, no npm install.
- [Three.js r128](https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js) via CDN for WebGL rendering (instanced meshes + raw `ShaderMaterial` raymarching).
- Native Canvas 2D API for the particle-physics preset and sparkline chart.
- Native Web Workers for multithreaded CPU load generation.
- `WEBGL_debug_renderer_info` extension for GPU identification.

## Deployment

This is a static single file — deploy anywhere. For GitHub Pages:

```bash
git add index.html
git commit -m "Deploy FPS Forge"
git push origin main
```

Enable Pages on the repo (Settings → Pages → deploy from `main` branch, root), and the app is live at `https://abhiraj1121.github.io/fps`.

## Benchmark Score Methodology

The composite score blends `avg×0.6 + 1%low×0.3 + 0.1%low×0.1`, then scales the result by a consistency factor (`0.7 + 0.3×consistency`, where consistency shrinks as the gap between average and 1% low widens). This rewards both raw throughput and frame-pacing stability, capped at 999 for display. Grades: **S** ≥180, **A** ≥120, **B** ≥70, **C** ≥35, **D** below.

## Credits

Developed in **[Cognix Studio](https://cognixstudio.github.io/)** — building experimental, performance-obsessed web tools.

## License

MIT
