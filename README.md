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

- **Mushroom grove** — Kaleidoscope (default), Zebra, Pointillist, Jitter, Melt, Pulse, Calm. Jitter turns on size and SF variance with a slow redraw; it is in every tab.
- **Gabor of Gabors** — Assemble (default), Nested, Jitter, Twist, Shatter, Zoom, Swarm, Classic. Nested runs the multi-scale cascade described under Nesting. Assemble starts with the small Gabors pulled apart, draws them together into the one big Gabor, holds, and repeats.
- **Portrait** — Assemble (default), Trip, Breathe, Jitter, Engraved, Kaleidoscope, Pointillist, Shatter, Melt, In color, Faithful. All but Pointillist, In color and Faithful are grayscale. Assemble starts as a cloud of separate Gabor patches and draws them together into the face; Trip sweeps pull, point spread and spatial frequency at once with angle-driven colour; Breathe sweeps the point spread; Faithful fades the carriers out, so it is the plain scan in colour.

## Make your own portrait

In the Portrait tab, **+ Make your own** opens these steps and a file picker. The scan is unpacked and converted in the browser; nothing is uploaded and no server or AI service is involved.

1. Get **Scaniverse** (free, iPhone and Android). It builds the scan on the phone.
2. Tap **New Scan** and choose **Splat**, not Mesh.
3. Have the person sit very still in soft, even light. Walk one slow, full circle around them at about arm's length, keeping head and shoulders in frame, then go around once more from a little higher.
4. Let it process, then **Edit → Crop** and pull the box in until only head and shoulders are left. An uncropped room swamps the face.
5. **Share → Export Model → SPZ**, and save it to Files.
6. In Gabor Grove, **+ Make your own → Choose your scan file**, and pick that file.

The dialog shows a progress bar and names each stage (reading, unpacking, building), and shows any failure in a red box under the button. The build runs in slices so the bar can move, and the scan already on screen stays until the new one is ready. The file is copied into memory before the picker is cleared, and gzip is unpacked by code in the page, so loading does not depend on the browser's stream decompressor.

Once it loads, drag to orbit until you find the face. If it arrives upside down, tap **Flip** in the preset row. Every preset and control works on it.

Two file formats are read: Scaniverse's `.spz` (versions 2 and 3) and the standard Gaussian-splat `.ply` (binary, with `f_dc_*`, `opacity`, `scale_*` and `rot_*` fields), which is what Polycam ("Splat PLY", on a paid plan) and KIRI Engine export. Only Scaniverse `.spz` files have been tried from a real app; the `.ply` reader was checked against a conversion of the same scan. Compressed `.ply` files and point clouds are rejected with a message. Up to 70,000 of the most opaque splats are kept and far-off background splats are dropped.

## Controls

The controls sit in a shelf that slides up from the bottom edge and stops below the header. Closed, it is fully off screen and only its **Controls** tab shows; the tab floats clear of the bottom edge so that reaching for it does not start the phone's own bottom-edge swipe, and its touch target is wider than it looks. Tap or drag the tab to open the shelf, and use the tab or ✕ to close it. When the open shelf is tall enough to hide the scene, holding a slider makes everything but that slider step aside. **WTF?** (top right) opens a plain-words explainer of splats, Gabors and what each tab is made of.

- **Carrier angle / Speed** — rotates every carrier's orientation over time.
- **Rotate: Together / As a wave** — one angle scene-wide, or an angle that lags with position.
- **About: Each splat's own axis / One world axis** — in-surface rotation, or a shared 3D rotation.
- **Point spread** — scales every splat's Gaussian envelope (0.33× to 3×).
- **Spread sweep** — swells and shrinks every envelope together, up to 2.4× either side of the Point spread value, on a log axis.
- **Spatial freq** — scales every carrier's frequency (0.25× to 4×).
- **SF sweep** — oscillates spatial frequency one octave either side of the slider value.
- **Size variance** — gives every splat its own envelope size, drawn at random from a log-normal spread centred on the Point spread value. The slider is the spread's SD, from 0 to 1.5 octaves (draws are clipped at 2 SD). A splat that swells also thins out, so the few largest do not bury the scene.
- **SF variance** — the same for spatial frequency: every splat draws its own carrier frequency around the Spatial freq value, SD up to 1.5 octaves. Each splat keeps its phase at its own centre, so a phase-locked whole (the big Gabor, the portrait) stays roughly together while assembled; the differences show most once it is pulled apart.
- **Variance sweep** — opens both spreads from zero (every splat alike) to the slider values and closes them again.
- **Redraw rate** — how often each splat draws afresh, 0.05 to 4 times a second, or frozen. Each splat turns smoothly from one draw into the next, out of step with its neighbours.
- **Contrast** — carrier contrast, 1% to 100% on a log axis. In the grove this is texture contrast; object colour and shading stay.
- **Contrast sweep** — log-sinusoid between the slider value and a 1% floor.
- **Color by angle** — tints the two carrier phases with opponent colours whose hue follows the carrier angle (180° of orientation spans the hue circle).
- **Nesting** (Gabor of Gabors) — how many scales deep the recursion goes, from classic (one big Gabor made of 1,290 small ones) up to 6 scales. At 2 or more, the big Gabor divides into eight half-size Gabors, each of those into eight more, and so on (8, 64, 512, 4,096, 32,768 at the bottom). Pull apart opens one scale at a time, and the camera follows one branch down so the pieces stay about the same size on screen. The pieces share one phase-locked carrier whose frequency rises as each scale opens, they are thin along the view axis (a fat many-cycle Gabor vanishes a few degrees off axis), and pieces behind the focused one fade toward gray.
- **Pull apart** (Gabor of Gabors, Portrait) — spreads the pieces out. In Gabor of Gabors, only one small Gabor in eight stays visible once apart, so each reads as a separate patch. In Portrait, one scan splat in 48 stays and grows into a patch wide enough to carry a few stripes of its own, with a longer wavelength so the stripes survive the camera backing off. The rest fade back in as it closes up.
- **Pull sweep** (Gabor of Gabors, Portrait) — cycles between pulled apart and assembled.
- **Scan color** (Portrait) — how much of the scan's own colour is kept. Portrait presets open in grayscale; Color by angle still tints on top.
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

- **Inputs:** `speed`, `pointSpread`, `spatialFreq`, `sfSweep`, `colorByAngle`, `contrast`, `contrastSweep`, `pullApart`, `pullSweep`, `nesting`, `depth3d`, `lens`, `angle` (the carrier angle itself, one turn per 0..1), `scene` (below 0.5 grove, above Gabor of Gabors), `spreadSweep`, `scanColor`, `sizeVariance`, `sfVariance`, `varianceSweep`, `redrawRate`.
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
