---
title: LiDAR — Every Setting
layout: default
parent: App Guide
nav_order: 7
---

# LiDAR — Every Setting

This is the deep reference for LiDAR (depth-streaming) mode: every control in the settings sheet, what it actually does to the depth, and how it changes what your NDI receiver sees. Depth from a sensor is noisy, holey and badly scaled before you touch it, so the pipeline gives you range, cleaning, toning and colouring stages — this page documents all of them, in the order the app groups them, with real defaults and ranges.

If you only want addresses and units, see the [OSC Reference]({{ '/osc-reference.html' | relative_url }}). For the live screen and a gentler walkthrough, see [LiDAR Mode]({{ '/lidar-mode.html' | relative_url }}). For the cloud-streaming mode's knobs, see [Point Cloud settings]({{ '/point-cloud-settings.html' | relative_url }}).

> Most users only ever touch **Definition** (rear camera) or **Colormap**. Everything else is there for when a specific shot needs it. If a control is marked **(Pro)** it requires the TDLidar Pro subscription or an unlock code — tapping it while locked opens the paywall instead of changing the value.
{: .tip }

## How the pipeline reads these settings

The order the controls *appear* in is not the order the GPU *applies* them. Roughly, each frame runs:

1. **Capture** the sensor depth (rear LiDAR `sceneDepth` at 256×192, or front TrueDepth).
2. **Clean** it — Apple filtering / Smart Hole Fill, the rear joint-bilateral upsample, temporal smoothing.
3. **Range & clip** — map metres to the 0–1 brightness band, then hard-cut near/far.
4. **Tone** — invert, brightness, contrast, gamma, and (rear) the combined Definition transform.
5. **Colour** — run the 0–1 value through the colourmap LUT, then optional edge **Detail** outline and Raw-mode effects.
6. **Scale & send** — optional HD upscale, depth-mask alpha, and the NDI write.

That order matters for the caveats below — e.g. clipping happens *before* the colourmap, so a clipped pixel is black no matter which palette you pick.

---

## Capture & Mode

These choose which sensor feeds the pipeline and which preset of processing runs. They live at the top of the **Depth Output** section.

### Depth Source

**What it controls.** The NDI source *name* the phone advertises (not the camera). Tapping the row cycles a suffix index 1→9 and back to 1.

**How it changes the look.** Nothing visual — it changes which entry you pick in TouchDesigner's NDI In source dropdown. Index 1 is the bare name `TDLidar`; index 2 is `TDLidar 2`, and so on.

**Default & range.** Index 1 (no suffix). Cycles 1–9.

**When to use it.** Running more than one phone on the same network — bump each phone to a unique index so their NDI sources don't collide.

### Mode

**What it controls.** The depth pipeline preset. Segmented picker: **Environment**, **Face Detail**, **Raw**.

**How it changes the look.**
- **Environment** — rear LiDAR, room-scale depth out to several metres. On rear-LiDAR devices this routes through the Metal joint-bilateral renderer (see Definition / Near Compression below), giving clean RGB-guided edges and filled holes.
- **Face Detail** — front TrueDepth, tuned for a tight depth window around the face so micro-features (nose, cheekbones, lips) get most of the brightness range.
- **Raw** — the least-processed path, plus an entire palette of stylised "glitch / scanner" looks (green phosphor by default). This is the one mode that fundamentally changes the image rather than just tuning it.

**Default & range.** First-launch default is **Face Detail**. The choice persists across launches.

**Interaction/caveat.** Switching mode reconfigures the capture session. The app keeps a *separate depth profile for the front camera*, so face-tuned settings don't disturb room-tuned settings when you flip the camera. The rear Metal controls (Definition, Near Compression, Near Threshold, Smoothing) only appear in **Environment** mode while the **rear** camera is active — they belong to the rear renderer and follow the camera, so they vanish on the front camera.

> **Raw is free; its customisation is Pro.** You can select Raw and see the green effect for free. The Base Color, Glitch, Bloom, Topographic/Posterize Bands, Scanline, Line Thickness and Colour-Map Effect controls all carry a PRO badge — tapping them while locked opens the paywall.
{: .note }

---

## Range & Clipping

Depth values are real metres. The pipeline has to decide which slab of distance maps onto the full brightness range, and where to simply cut everything to black. Get this right first — it does more for the look than any tone control.

### Max Range *(Environment mode)*

**What it controls.** The far edge of the depth-to-brightness mapping in Environment mode. Distances beyond it read as empty/black.

**How it changes the look.** Lower values pack all the brightness range into the near field, so a person at 1–2 m gets strong front-to-back separation while the far wall flattens out. Higher values keep distant scene depth but spend less of the range on anything close — the whole room reads, but small near objects separate less.

**Default & range.** Default **5 m**. Picker: 1, 2, 3, 5, 8 m. With **Extended LiDAR Range (Pro)** on, it also offers **10 m** and **12 m**.

**When to use it.** Drop to 1–2 m for a close subject; raise to 8 m+ for a stage or large room.

**Interaction/caveat.** This is the denominator for the whole Environment mapping, so it interacts with Near/Far Clip and Near Threshold (whose ~1.5 m sweet spot is computed relative to a 5 m range).

### Detail Range *(Face Detail mode)*

**What it controls.** The depth window, in millimetres, kept around the median face distance. The tracker finds the face plane each frame and this is the ± slab around it.

**How it changes the look.** Tighten it and the face is cut cleanly from the background — everything outside the slab goes dark, and the face's own depth fills the full brightness range (lots of facial contrast). Widen it and you keep shoulders, hands and surroundings, at the cost of facial contrast.

**Default & range.** Default **500 mm** (the picker max, so subjects beyond the face plane still register out of the box). Range 20–500 mm.

### Near Clip

**What it controls.** A hard near cutoff in metres. Anything closer than this is forced to black and dropped.

**How it changes the look.** Raise it to erase your own hands or the foreground tripod — they snap to black the instant they cross the line. There's no falloff; it's a guillotine.

**Default & range.** Default **0.00 m** (off). Range 0–1.00 m, 0.05 m steps.

### Far Clip

**What it controls.** A hard far cutoff in metres. Anything farther than this is forced to black and dropped.

**How it changes the look.** Lower it to delete a busy background — the wall behind the subject vanishes to black at the cutoff distance.

**Default & range.** Default **10.0 m** (effectively off, since it sits at the picker top). Range 0.5–10.0 m, 0.5 m steps.

> **Clip vs. Range.** Max Range *rescales* the brightness mapping; Near/Far Clip *delete* pixels. Clipping is the cleanest way to fence the depth to exactly the slab you care about, and because it runs before the colourmap, a clipped pixel is black in every palette.
{: .important }

| Pixel distance | Near Clip = 0, Far Clip = 10 | Near Clip = 0.5, Far Clip = 3 |
|---|---|---|
| 0.3 m | maps to bright | **black** (inside near clip) |
| 1.5 m | mid | bright |
| 5 m | dim | **black** (past far clip) |
| 9 m | very dim | black |

---

## Cleanup & Smoothing

The sensor leaves black holes at glossy/dark/silhouette pixels and shimmers frame-to-frame. These controls (in the **Filtering** section) decide between *smoother-but-filled* and *sharper-but-holey*.

### Smart Hole Fill

**What it controls.** A local 5×5 majority-vote fill in the NDI encoder that patches isolated invalid speckle while leaving large holes invalid. (Internally this is the repurposed `appleFiltering` flag — Apple's own whole-frame filter stays off because it warped silhouettes.)

**How it changes the look.** Turns on: the little black specks scattered around object edges — most visible in the front-camera modes — get filled in with their neighbours' depth, so edges read clean. Big genuine gaps still stay black.

**Default & range.** **Off** by default.

### Disable Apple Depth Filtering

**What it controls.** The master "raw depth" switch, and the opposite of Smart Hole Fill's intent. **Off (default)** lets the gaps fill (front TrueDepth Env/Raw turn on Apple's `isFilteringEnabled`, the rear LiDAR stays confidence-weighted). **On** gives raw unfiltered depth on both cameras — front filtering off and rear confidence weighting off (`sigmaConfidence = 0`).

**How it changes the look.** Left **off** you get smooth, filled depth. Turned **on** you get crisper edges but the black edge-gaps return on the front camera; on the rear, the black "shadow" halos at low-confidence pixels (glossy, dark, silhouette edges) *fill in* because every tap now contributes fully.

**Default & range.** **Off.**

**When to use it.** Turn on when you want maximum sharpness and will clean up downstream, or when rear-LiDAR confidence halos are bothering you more than edge gaps.

### Temporal Smoothing

**What it controls.** A frame-to-frame exponential average of the depth (temporal EMA). Higher blends more of the previous frame into the current one.

**How it changes the look.** Calms the per-frame shimmer the sensor produces, so flat surfaces stop crawling. More smoothing is steadier but adds a little motion lag — fast movement smears.

**Default & range.** Default **Off (0%)**. Range 0–80%, 5% steps.

**Interaction/caveat.** The rear Metal renderer has its own dedicated **Smoothing** slider (under Environment mode) that overrides this for the rear path, because the rear LiDAR's flash needs to be damped independently from how snappy you want the front camera.

### HD Smoothing Strength

**What it controls.** The sigma of the post-Lanczos Gaussian smoother in the HD upscale path (`MPSImageGaussianBlur`). Only meaningful when an HD resolution is selected.

**How it changes the look.** 0 = just the sharp Lanczos upscale (you may see faint stair-stepping on band/silhouette edges). Raise it to soften those upscale artifacts; ~3.0 is a heavy blur.

**Default & range.** Default **Off (0.0)**. Range 0–3.0, 0.1 steps. (See also the separate **Edge-preserving smoothing** toggle in the Pro section, which is a different, edge-aware pass.)

---

## Tone Curve

These shape the colour-mapped image — they remap brightness, not the underlying distances. They live in **Output Adjustments**.

### Invert Depth

**What it controls.** Swaps which end of the range is bright.

**How it changes the look.** By default **near is bright, far is dark**. Invert and far becomes bright, near goes dark.

**Default & range.** **Off.**

### Brightness

**What it controls.** A flat additive offset on the mapped value.

**How it changes the look.** Shifts the whole image lighter or darker without changing contrast — the gap between near and far stays the same, it just floats up or down.

**Default & range.** Default **0%**. Range −50% to +50%, 5% steps.

### Contrast

**What it controls.** A multiplicative spread of the midtones around the centre.

**How it changes the look.** Above 1× pushes near and far apart — more punch, but you can crush the extremes to pure white/black. Below 1× compresses everything toward grey and flattens the depth.

**Default & range.** Default **1.0×**. Range 0.5×–2.0×, 0.1 steps.

### Gamma

**What it controls.** A power curve on the response.

**How it changes the look.** Above 1 bends the curve to lift shadows (more visible detail in the dark/far end). Below 1 crushes shadows and favours highlights (the near end).

**Default & range.** Default **1.0**. Range 0.3–3.0, 0.1 steps.

### Detail (depth-edge outline)

**What it controls.** A post-pass in the NDI encoder that traces crisp lines along depth discontinuities (where distance jumps). Works in Environment, Face and Raw.

**How it changes the look.** 0 = off. Raise it and clean outlines appear along every silhouette and depth step — a line-art look layered over the colour. Higher values draw the outlines more strongly.

**Default & range.** Default **0.00 (off)**. Range 0.00–1.00, 0.01 steps.

**When to use it.** Great for a graphic, contour-traced look over any colourmap, or to give flat-coloured depth structure.

---

## Definition & Near-Range *(rear LiDAR, Environment mode)*

These knobs only appear in **Environment** mode while the **rear** camera is active on a rear-LiDAR device — they drive the Metal renderer that produces the per-pixel depth value before the colourmap. They are not Pro-gated, but the Metal renderer itself is a rear-LiDAR feature.

### Definition

**What it controls.** A single combined transform: contrast lift + near-range expansion + edge contrast, all in one knob.

**How it changes the look.** This is the one most people use on the rear camera. Higher separates small objects from each other and from the background without you tuning three controls by hand — a cluttered desk reads as distinct objects instead of a grey lump. At 0 you get the flatter legacy look.

**Default & range.** Default **100%** (1.0). Range 0–100%, 5% steps. (0% restores the pre-Definition behaviour exactly.)

### Near Compression *(advanced)*

**What it controls.** How hard the close band gets expanded across the LUT. 0 = linear (close objects blow out to the bright end). At 1 the near band gets a sqrt expansion so objects under ~1 m spread across more LUT entries instead of clipping to white.

**How it changes the look.** Tames the over-bright, blown-out foreground the rear LiDAR produces when something is very close — recovers shape in the near object instead of a flat white blob.

**Default & range.** Default **0%** (0.0). Range 0–100%, 5% steps.

### Near Threshold *(advanced)*

**What it controls.** The normalised-depth point above which the near-compression curve fades back to identity. Below it, compression applies; above it, depth passes through unchanged.

**How it changes the look.** Sets *how deep* the near-compression effect reaches. The default ≈ 1.5 m at a 5 m Max Range, so the 1–2 m sweet spot stays untouched while only the very-near band is reshaped.

**Default & range.** Default **0.10**. Range 0.10–0.60, 0.05 steps.

### Smoothing *(rear Metal)*

**What it controls.** Temporal smoothing dedicated to the rear Metal renderer, independent of the global Temporal Smoothing. 0 = no smoothing; toward 1 freezes more of the previous frame.

**How it changes the look.** Damps the rear LiDAR's occasional bright flash between sensor updates without forcing the front modes to feel laggy.

**Default & range.** Default **0%** (0.0). Range 0–95%, 5% steps.

> Near Compression and Near Threshold are genuinely advanced — most users get everything they need from **Definition** alone. Reach for them only when a very close object is blowing out the foreground.
{: .tip }

---

## Colour & Effects

These pick the palette that turns distance into an image, and the optional stylised looks layered on top. The colourmap picker lives in **Filtering**; the Raw-mode effects live inline under the **Raw** mode tab.

### Colormap

**What it controls.** The 256-entry colour lookup table applied to the 8-bit depth value just before the NDI write. Distance → colour.

**How it changes the look.** Entirely — it's the palette. The cheat-sheet below describes each. Several are industry-standard names matched to TouchDesigner / LOTA so composites line up with existing projects.

**Default & range.** Default **Black & White**. Six maps are **(Pro)**; they preview correctly but can only be *selected* with Pro (otherwise the picker reverts and opens the paywall).

| Colormap | Look (near → far) | Tier |
|---|---|---|
| **Black & White** | white → black, linear greyscale | Free |
| **Black Aqua White** | black → aqua → white | Free |
| **Blue Red** | blue → red linear | Free |
| **Deep Sea** | black → deep blue → teal → seafoam → white | Free |
| **Color Spectrum** | full HSV hue sweep (rainbow) | Free |
| **Incandescent** | black → deep red → orange → yellow → white | Free |
| **Heated Metal** | black → purple → red → orange → yellow → white | Free |
| **Sunrise** | indigo → magenta → orange → yellow | Free |
| **Visible Spectrum** | violet → blue → green → yellow → orange → red | Free |
| **TouchDesigner Heat** | TD's `heat.lut` — black → blue → teal → orange → red → white | **Pro** |
| **LiDAR Scan** | phosphor-green-on-black scanner aesthetic | **Pro** |
| **Cyberpunk** | magenta → cyan → white neon | **Pro** |
| **Topographic** | 8 stepped contour bands, survey-map elevation tints | **Pro** |
| **Depth Slice** | only the middle ~20% of the range is visible (high-contrast grey), the rest black | **Pro** |
| **Dual-tone** | teal → orange (cool → warm) | **Pro** |

**Interaction/caveat.** The colourmap runs *after* clipping and toning, so clipped pixels stay black and the tone curve decides which colours a given distance lands on. **Depth Slice** is effectively a soft far/near clip baked into the palette — pair it with brightness/contrast/gamma to move the visible slice.

### Colour-Map Effect *(Raw mode, Pro)*

**What it controls.** A master toggle in the Raw mode tab that layers a topographic ridge-line look — the chosen colourmap painted as bright contour lines whose thick/thin character is curated per map.

**How it changes the look.** Off = the classic green Raw effect. On = the depth field is drawn as topographic ridges in the selected palette (e.g. bold neon maps get thick sparse bands; scan/sea maps get fine dense lines).

**Default & range.** **Off** (classic green). When on, a **Colour Map** picker appears beneath it (default Color Spectrum).

### Topographic Bands *(Raw mode, Pro)*

**What it controls.** A stepper for the number of contour rings drawn on close objects.

**How it changes the look.** Higher = more, finer rings, like a denser survey map; lower = a few bold contours.

**Default & range.** Default **8**. Range 1–20.

### Posterize Bands *(Raw mode, Pro)*

**What it controls.** Quantises the depth into N discrete levels.

**How it changes the look.** Fewer bands = chunky, stepped "scanner" terracing; more bands approaches smooth.

**Default & range.** Default **8**. Range 2–16.

### Line Thickness *(Raw mode, Pro)*

**What it controls.** A global multiplier on each colourmap's curated contour-line weight.

**How it changes the look.** Thicker or thinner topographic/contour lines across the whole effect.

**Default & range.** Default **1.0×**. Range 0.2×–3.0×, 0.2 steps.

> The Raw tab carries more stylistic knobs — **Base Color**, **Glitch Speed / Intensity**, **Reflection Sensitivity**, **Bloom**, **Scanline Style** (None / Horizontal / Vertical / Diagonal). These are all Pro and only change the Raw look; they're listed here for completeness. On iPhone 16+ hardware, a **Camera Control Scrolls** picker lets the hardware Camera Control button drive any one of these live without touching the screen.
{: .note }

---

## Resolution & Frame Rate

How big and how often the NDI frame goes out. Resolution lives in **Depth Output**; Frame Rate lives in **Performance**.

### Resolution

**What it controls.** The output size of the NDI/recorded frame. One sorted list mixes the sensor-native hardware formats with the HD upscale presets.

**How it changes the look.** Native keeps the sensor-rotated dimensions and looks exactly like the sensor. HD entries run a Lanczos upscale (see HD below) — a larger, smoother frame. Everything above 640×480 — whether hardware-native or HD — is **(Pro)**.

**Default & range.** Native (device-dependent). HD presets: 800×600, 1024×768, 1280×960, 1440×1080, 1920×1440 (shown in sensor-landscape; the portrait NDI output swaps the axes).

**Interaction/caveat.** HD resolution does **not** persist across launches (crash-safety — a too-high tier could OOM on cold start); every launch resets to Native so you have a known-good baseline to dial up from. The on-device preview keeps showing the *native* frame for responsiveness while NDI/recordings go out at the HD size.

### Frame Rate (Target FPS)

**What it controls.** The capture/advertise rate.

**How it changes the look.** Higher is smoother motion on the receiver. The depth sensor itself caps at 30 fps (TrueDepth front *and* the LiDAR depth stream), so:
- 10/15/24/30 fps capture and send at that rate.
- 45 fps captures at 30 (capped) and sends at 45.
- **60 fps** is a deliberate trick: depth captures at 30 fps but the NDI stream is *advertised* at 60 fps with each frame doubled (timecodes spaced 1/60 s apart), so receivers play it back at a clean 60 Hz with no added judder — and it costs nothing in capture load.

**Default & range.** Default **30 fps**. Options: 10, 15, 24, 30, 45, 60.

**When to use it.** 60 for the smoothest downstream playback on rear LiDAR. If the **frame-rate pill** on the live screen sits well below your target, the phone is throttling (usually heat) — drop resolution or turn off Supercharge.

---

## Pro & HD

These live in the **TDLidar Pro** section. All are **(Pro)** and no-op while a subscription is lapsed (your tuning is kept and comes back when you re-subscribe).

### Extended LiDAR Range

**What it controls.** Unlocks 10 m and 12 m on the Environment **Max Range** picker and pushes pixels out to those distances instead of clipping them.

**How it changes the look.** Distant scene depth in big rooms/stages still makes it to the output instead of falling off a cliff at 8 m.

**Default & range.** **Off.**

### Edge-preserving smoothing *(HD)*

**What it controls.** A gentle 3×3 tent convolution over the upscaled HD frame. Appears only when an HD resolution is selected.

**How it changes the look.** Softens the stair-step edges the Lanczos upscale introduces — smoother diagonals and silhouettes — while preserving more edge than a plain blur.

**Default & range.** **On** by default (most HD users want it).

### Depth-mask alpha NDI

**What it controls.** Writes alpha = 0 for depth-invalid pixels in the NDI BGRA frame instead of alpha = 255.

**How it changes the look.** The background (anything with no valid depth) becomes transparent in TouchDesigner / OBS, so you can composite the subject straight over other layers without keying.

**Default & range.** **Off.**

**Interaction/caveat.** Pair it with Near/Far Clip to define exactly what counts as "background" — clipped pixels are invalid, so they key out.

---

## RGB (the colour camera)

### Send RGB Camera *(Pro)*

**What it controls.** Broadcasts the ordinary colour camera as its **own second NDI output** alongside the depth stream.

**How it changes the look.** Adds a real-photo feed next to the depth feed. In TouchDesigner you can composite the camera against the depth, key one with the other, or just use the phone as a wireless camera.

**Default & range.** **Off.**

**Interaction/caveat.** When on, a **Combine Side-by-Side** toggle appears:
- **Off** (default) — two independent NDI sources (drop each into its own NDI In TOP).
- **On** — one combined stream, camera on the left and depth on the right, for receivers that can't take two sources.

The RGB source advertises its own NDI name (a "Camera Source" / "Side-by-side" name shown in the row), so a second NDI source appears in the TouchDesigner dropdown.

---

## Receiving it in TouchDesigner

On the TouchDesigner side, an **NDI In TOP** picks the phone from its source dropdown — no IP or port. If you turned on **Send RGB Camera**, a second NDI source appears for the colour feed. The operator family includes ready-made **NDI** and **Depth** operators that wrap this up. For wiring and addresses see the [OSC Reference]({{ '/osc-reference.html' | relative_url }}) and the [LiDAR Mode]({{ '/lidar-mode.html' | relative_url }}) overview.

---

## Quick reference — every LiDAR setting

| Setting | Section | Default | Range / options | Pro |
|---|---|---|---|---|
| Depth Source | Depth Output | index 1 | 1–9 | – |
| Mode | Depth Output | Face Detail | Environment / Face Detail / Raw | – |
| Max Range | Depth Output (Env) | 5 m | 1,2,3,5,8 (+10,12 Pro) | range ext. is Pro |
| Detail Range | Depth Output (Face) | 500 mm | 20–500 mm | – |
| Near Clip | Depth Clipping | 0.00 m | 0–1.00 m | – |
| Far Clip | Depth Clipping | 10.0 m | 0.5–10.0 m | – |
| Smart Hole Fill | Filtering | Off | on/off | – |
| Disable Apple Depth Filtering | Filtering | Off | on/off | – |
| Temporal Smoothing | Filtering | Off (0%) | 0–80% | – |
| HD Smoothing Strength | Filtering | 0.0 | 0–3.0 | HD only |
| Colormap | Filtering | Black & White | 15 maps | 6 are Pro |
| Invert Depth | Output Adjustments | Off | on/off | – |
| Brightness | Output Adjustments | 0% | −50…+50% | – |
| Contrast | Output Adjustments | 1.0× | 0.5–2.0× | – |
| Gamma | Output Adjustments | 1.0 | 0.3–3.0 | – |
| Detail (edge outline) | Output Adjustments | 0.00 | 0–1.00 | – |
| Definition | Depth Output (rear Env) | 100% | 0–100% | – |
| Near Compression | Depth Output (rear Env) | 0% | 0–100% | – |
| Near Threshold | Depth Output (rear Env) | 0.10 | 0.10–0.60 | – |
| Smoothing (rear Metal) | Depth Output (rear Env) | 0% | 0–95% | – |
| Colour-Map Effect | Raw tab | Off | on/off (+ map) | Pro |
| Topographic Bands | Raw tab | 8 | 1–20 | Pro |
| Posterize Bands | Raw tab | 8 | 2–16 | Pro |
| Line Thickness | Raw tab | 1.0× | 0.2–3.0× | Pro |
| Resolution | Depth Output | Native | native + 5 HD tiers | >640×480 Pro |
| Frame Rate | Performance | 30 fps | 10,15,24,30,45,60 | – |
| Extended LiDAR Range | TDLidar Pro | Off | on/off | Pro |
| Edge-preserving smoothing | TDLidar Pro | On | on/off | Pro / HD |
| Depth-mask alpha NDI | TDLidar Pro | Off | on/off | Pro |
| Send RGB Camera | TDLidar Pro | Off | on/off (+ side-by-side) | Pro |

> The **Performance** section also carries Keep Screen On, Keep Alive in Background, Supercharge and NDI Wired Mode. Those affect stability and battery rather than the depth *look* — see [Performance & Pro]({{ '/performance-pro.html' | relative_url }}).
{: .note }
