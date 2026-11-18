# Gabor Grove

A WebGL2 demo of 3D Gabor splatting: Gaussian splatting where each primitive is a Gaussian envelope multiplied by an oriented 3D sinusoidal carrier.

Open `index.html` in a browser. It is a single self-contained file with no build step.

VR head tracking needs the page served over HTTPS as a top-level page (for example GitHub Pages); embedded previews that block motion sensors fall back to drag-to-look.

## Tabs

- **Mushroom grove** — a procedural twilight grove of about 35,000 Gabor splats. Gills, cap striations, stem fibres, bark, growth rings and fern leaflets all come from carriers, not from extra splats.
- **Gabor of Gabors** — one large 3D Gabor on a mean-gray field, built from 1,290 small phase-locked Gabor splats on a jittered lattice.
- **Portrait** — a head-and-shoulders Gaussian-splat scan from a phone (Scaniverse), loaded on demand from `portrait.json` and rendered as Gabor splats. Each Gaussian keeps its position, shape, colour and opacity, and all of them share one phase-locked carrier, so the stripes cut through the head as parallel sheets and show on the surface like engraving lines.

## Presets

Each tab opens in its first preset; the pills under the tabs switch between them. A preset just sets the controls below, and changing any control by hand clears the highlight.

- **Mushroom grove** — Kaleidoscope (default), Zebra, Pointillist, Melt, Pulse, Calm.
- **Gabor of Gabors** — Assemble (default), Nested, Twist, Shatter, Zoom, Swarm, Classic. Nested runs the multi-scale cascade described under Nesting. Assemble starts with the small Gabors pulled apart, draws them together into the one big Gabor, holds, and repeats.
- **Portrait** — Engraved (default), Faithful (carriers faded out, so the plain scan), Kaleidoscope, Pointillist, Shatter, Melt.

## Controls

- **Carrier angle / Speed** — rotates every carrier's orientation over time.
- **Rotate: Together / As a wave** — one angle scene-wide, or an angle that lags with position.
- **About: Each splat's own axis / One world axis** — in-surface rotation, or a shared 3D rotation.
- **Point spread** — scales every splat's Gaussian envelope (0.33× to 3×).
- **Spatial freq** — scales every carrier's frequency (0.25× to 4×).
- **SF sweep** — oscillates spatial frequency one octave either side of the slider value.
- **Contrast** — carrier contrast, 1% to 100% on a log axis. In the grove this is texture contrast; object colour and shading stay.
- **Contrast sweep** — log-sinusoid between the slider value and a 1% floor.
- **Color by angle** — tints the two carrier phases with opponent colours whose hue follows the carrier angle (180° of orientation spans the hue circle).
- **Nesting** (Gabor of Gabors) — how many scales deep the recursion goes, from classic (one big Gabor made of 1,290 small ones) up to 6 scales. At 2 or more, the big Gabor divides into eight half-size Gabors, each of those into eight more, and so on (8, 64, 512, 4,096, 32,768 at the bottom). Pull apart opens one scale at a time, and the camera follows one branch down so the pieces stay about the same size on screen. The pieces share one phase-locked carrier whose frequency rises as each scale opens, they are thin along the view axis (a fat many-cycle Gabor vanishes a few degrees off axis), and pieces behind the focused one fade toward gray.
- **Pull apart** (Gabor of Gabors) — spreads the small Gabors out. Once apart, only one in eight stays visible so each reads as a separate patch; the rest fade back in as they close up.
- **Pull sweep** (Gabor of Gabors) — cycles between pulled apart and assembled.
- **3D** — anaglyph stereo for blue/green or red/cyan glasses, with a depth slider and an eye swap. Each eye's view is reduced to luminance and written to that eye's colour channels; the two views use off-axis frusta with zero disparity at the orbit target.
- **VR** — split-screen stereo for phone headsets (Cardboard-style, e.g. Destek V5). Head tracking uses the device orientation sensor; tap, Space or Enter toggles walking (grove) or flying (Gabor) in the gaze direction. **Lens correction** sets the barrel pre-distortion; **Drift forward in VR** moves continuously for headsets with no button.
- **Carriers on** — switch off to see the same splats as plain Gaussians.
- **Rotate envelopes too** — rotates the Gaussian envelopes along with the carriers.

## CV port

Other pages, or MIDI gear, can play Gabor Grove's sliders and read its motion back. `cv-test.html` is a small stand-in for a mod matrix that does both, and is the reference for the messages below.

**From another page** (any origin). Open or frame Gabor Grove, then post messages to that window:

```js
const gg = window.open('https://samuelling.github.io/GaborGrove/?cv', 'gabor-grove');   // or iframe.contentWindow, with ?embed to hide the interface

gg.postMessage({ type: 'gg:set', param: 'spatialFreq', value: 0.73 }, '*');   // 0..1, absolute slider position
gg.postMessage({ type: 'gg:mod', param: 'contrast',    value: lfo  }, '*');   // -1..1, added to where the slider sits

addEventListener('message', e => {
  if (e.data?.type === 'gg:ready')  gg.postMessage({ type: 'gg:describe' }, '*');
  if (e.data?.type === 'gg:params') console.log(e.data.inputs, e.data.outputs);  // what can be played, what can be read
  if (e.data?.type === 'gg:state')  console.log(e.data.values);                  // about 30 times a second, each 0..1
});
```

- **Inputs:** `speed`, `pointSpread`, `spatialFreq`, `sfSweep`, `colorByAngle`, `contrast`, `contrastSweep`, `pullApart`, `pullSweep`, `nesting`, `depth3d`, `lens`, `angle` (the carrier angle itself, one turn per 0..1), `scene` (below 0.5 grove, above Gabor of Gabors).
- **Outputs:** `angle`, `angleSin`, `spatialFreq`, `contrast`, `pullApart`.
- A slider moved by hand, or set with `gg:set`, becomes the centre that `gg:mod` swings around.
- Pages on the same origin can use `new BroadcastChannel('gabor-grove')` with the same messages.

**MIDI.** "Connect MIDI" in the drawer listens on every MIDI input for CC 20 upward (one per input, in the order above) and sends the outputs as CC 40 upward. This needs a browser with Web MIDI.

## How the projection works

Each splat's covariance is expressed in a frame whose third axis is the viewing ray. Integrating `G(x) cos(k·x + φ)` along the ray gives a 2D Gaussian times a 2D carrier:

- in-plane frequency `k_eff = k_u + Σ_uu⁻¹ Σ_uz k_z`
- contrast attenuation `exp(-½ k_z² σ²_{z|u})`, where `σ²_{z|u} = Σ_zz − Σ_zu Σ_uu⁻¹ Σ_uz`

So stripes fade as the carrier tips toward the line of sight. Carriers above the pixel Nyquist rate are faded out as well. Splats are depth-sorted on the CPU each frame and alpha-composited back to front.

The Gabor of Gabors tab accumulates into a 16-bit float buffer as linear luminance around a mean of 0.5, then resolves once to sRGB with dither. This keeps low contrasts free of 8-bit rounding mottle. It falls back to direct 8-bit compositing if float render targets are unavailable.
