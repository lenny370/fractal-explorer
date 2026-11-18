# fractal-explorer

A deep-zoom fractal explorer in one self-contained HTML file. No build, no server, no dependencies — open it in a browser and dive to zoom 10²⁹⁰ on an ordinary GPU.

Everything runs locally: WebGL2 shaders for rendering, a web worker for arbitrary-precision math, WebCodecs for video export.

Features
Seven fractals — Mandelbrot, Julia, Burning Ship, Tricorn, Sierpinski Carpet, Sierpinski Triangle, Koch Snowflake
Deep zoom — 10²⁹⁰ for the Mandelbrot set (10²⁵ for Burning Ship / Tricorn, 10¹³ for Julia), up to 100,000 iterations
Video rendering — exponential zoom dives as MP4 (H.264 / HEVC), WebM (VP9 / AV1) or a lossless PNG sequence with ready-made ffmpeg commands; large renders stream straight to disk
Music-reactive color — load an audio track and the palette pulses and cycles to its energy, bass and transients, with a depth-weighted focus control that lets only the dense structure react; a live preview in the render dialog shows the effect before you commit
Photo export up to custom resolutions, 2–4× supersampled anti-aliasing
Shareable states — copy/paste a small JSON blob or save it as a file; coordinates are stored as decimal strings, so states are exact and device-independent
Six palettes plus custom gradients, color density/shift/animation controls, interior color
Quick start
Download mandelbrot-explorer.html
Open it in Chrome or Edge (WebGL2 required; video export needs WebCodecs)
Scroll to zoom, drag to pan, double-click to center, H for help

Hardware floats die early: past zoom ~10¹³, neighbouring pixels round to the same float64 coordinate and the image degenerates. The explorer stacks five techniques, each born from the failure of the previous one:

#	Technique	What it solves
1	Arbitrary-precision reference	One orbit per view is computed in BigInt fixed-point (hundreds of bits) in a worker. Only one.
2	Perturbation (K. I. Martin)	Every pixel is a tiny offset δ from the reference. The huge coordinates cancel algebraically — δ' = 2Zδ + δ² + δc — so per-pixel math fits in float32 on the GPU.
3	Rebasing (Zhuoran)	When a pixel's value dips below its own δ, it restarts against iteration 0 of the same reference. Exact for the Mandelbrot set (Z₀ = 0); replaces classical glitch detection entirely.
4	floatexp	float32 underflows at 10⁻⁴⁵, but deep deltas go far below. A float32 mantissa paired with a separate exponent extends the range to ~10^±10⁸; a dedicated deep shader takes over past zoom ~10²⁵.
5	BLA (bivariate linear approximation, Zhuoran / Fraktaler-3)	While δ is small the iteration is effectively linear, and linear steps compose. A precomputed binary tree lets each pixel skip 2…8192 iterations in a single multiply-add — falling back to exact steps precisely where rebasing has to run anyway.

The numerics were developed test-first: every algorithm was validated against exact BigInt per-pixel iteration (and, where possible, independent constructions) in Node harnesses before touching the shaders. The HUD shows which engine is active — GPU·pert, GPU·pert-fx (floatexp), +bla when skip tables are live, GPU·geo for the geometric fractals, or the CPU fallback.

A companion page, engine-notes.html, explains all five techniques with interactive instruments that compute the real formulas live.

Performance
Interactive frames are paced to what the GPU sustains; the frames in between are cheap affine transforms of the last render, so motion stays at display rate even when the GPU manages far fewer true frames
Adaptive iteration budget and resolution during motion (FAST), or full-resolution interactive frames (CRISP) — selectable
Heavy final renders draw progressively in bands and cancel instantly on input
A probe pass measures the actual iteration requirement of each view, cutting the depth heuristic down where deep regions are genuinely simple
Background tabs keep rendering videos at full speed (worker-based timing instead of throttled requestAnimationFrame)
Video rendering notes

The encoder is calibrated at render start: a short burst of incompressible noise is pushed through candidate configurations (hardware/software, latency and bitrate modes) and the first one that measurably delivers the requested bitrate wins — some platform encoders silently ignore the bitrate parameter otherwise. If no configuration reaches the target, the completion message says so honestly. The PNG sequence export sidesteps hardware encoding entirely and bundles the ffmpeg commands to assemble a master-quality file.

Browser support

Chrome / Edge / other Chromium browsers recommended. Requires WebGL2; video export requires WebCodecs; streaming renders to disk use the File System Access API. HEVC and AV1 encoding depend on your GPU and OS exposing them.

Credits

Standing on the shoulders of the deep-zoom community: perturbation theory by K. I. Martin (SuperFractalThing), rebasing and bivariate linear approximation by Zhuoran, floatexp lineage from Kalles Fraktaler and Claude Heiland-Allen's Fraktaler-3. Built in collaboration with Claude (Anthropic).
