---
title: What's New
layout: default
nav_order: 9
---

# What's New

A per-version log of user-facing changes. For the full control reference see the [App Guide]({{ '/app-guide.html' | relative_url }}) and [Show Control]({{ '/show-control-mode.html' | relative_url }}).

## v9.12

- **Remote factory reset.** Show Control adds `/tdlidar/show/reset` (a **Reset Settings** pulse on the `tdlidar_show` operator) — restores every setting to its default, exactly like the in-app *Reset All*. See [Show Control]({{ '/show-control-mode.html#every-setting-over-osc--midi' | relative_url }}).
- **Menu controls on the show operator.** The discrete Show-Control parameters are now dropdown menus instead of 0–1 faders — `pcNDIResolution` (Low / Med / High / Max), `pcOrbit`, `pcDetailUpsample`, `monoModel`, `monoCamera` and `depthMode`. They still send the same normalized value on the wire.

## v9.11

- **Every setting over OSC & MIDI.** All **141** LiDAR / Monocular Depth / Point Cloud controls are now remote-controllable — three parameter tabs on the `tdlidar_show` operator, each a normalized `0–1` fader (or toggle). Full key list in [Show Control → Every setting over OSC & MIDI]({{ '/show-control-mode.html#every-setting-over-osc--midi' | relative_url }}). *(Breaking: the shared `gamma`/`contrast`/`brightness`/`threshold`/`colorMapIndex` keys are now LiDAR-only and normalized; Monocular has its own `mono…` keys.)*
- **Point Cloud movement lock.** A padlock button (top-right of Point Cloud mode) freezes orbit / pan / pinch / reset and any auto-orbit, so a framed shot can't be nudged during a show. See [Point Cloud Mode]({{ '/point-cloud-mode.html' | relative_url }}).
- **Cloud Tilt.** Rotate the whole point cloud on X/Y/Z (±180°), alongside Cloud Position. See [Point Cloud settings]({{ '/point-cloud-settings.html' | relative_url }}).

## v9.10

- **Shutter: tap = photo, hold = record.** In LiDAR and Monocular Depth modes a quick tap of the shutter saves a photo (the on-screen frame **plus** a plain RGB still) to Photos; tap-and-hold (~⅔ s) starts a video recording, and a tap stops it.
- **Live camera ⇄ depth switch.** A button (Pro, right of the NDI button) flips the whole output — preview, NDI stream and any recording — between the colour-mapped depth and the plain RGB camera, instantly, even mid-stream.
- **Point Cloud OSC/MIDI + Off/Spin/Sway orbit.** The first Point Cloud remote controls, plus an auto-orbit picker: Off, continuous Spin, or a ±90° Sway pendulum.

## Earlier

- **Monocular Depth** — camera-only neural depth on any iPhone, three model tiers (Small / Medium / High), streamed over NDI.
- **Point Cloud** — lossless 3D point cloud over TCP (POP) and an NDI viewport.
- **Mesh Cloud, Scene Build, Sensors, Cue Deck, Align** — see the [App Guide]({{ '/app-guide.html' | relative_url }}).
