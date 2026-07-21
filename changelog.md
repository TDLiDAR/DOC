---
title: What's New
layout: default
nav_order: 9
---

# What's New

A per-version log of user-facing changes. For the full control reference see the [App Guide]({{ '/app-guide.html' | relative_url }}) and [Show Control]({{ '/show-control-mode.html' | relative_url }}).

## Latest (July 2026)

- **ARKit Point Cloud mode (beta).** A new Pro, capture-only mode: ARKit person segmentation crossed with LiDAR depth captures every human figure in view as a transient, world-locked point-cloud silhouette, replaced fresh on every scan. Pause freezes the frame so you can walk around it while the camera stays live; capture works like the other modes (tap = photo, hold = video). See [ARKit Point Cloud Mode]({{ '/arkit-point-cloud-mode.html' | relative_url }}).
- **Feedback & Support form.** The docs home page now has a direct feedback form — bug reports, feature requests, questions — see [Feedback & Support]({{ '/#report' | relative_url }}).
- **Hardware buttons.** The volume buttons and the iPhone-16 Camera Control click can each run an action in LiDAR and Monocular Depth — Photo (default), Record, RGB / Depth switch, Camera Flip, or Adjust. Configure under Settings → **Hardware Buttons**.
- **Camera Control in Monocular Depth.** The iPhone-16 slide now scrolls a Mono setting of your choice (Colormap default, Brightness, Contrast, Gamma, Smoothing, Camera) via **Camera Control Scrolls** — LiDAR parity.
- **Saved-to-gallery note.** A brief "photo/video saved to gallery" caption under the fps pill confirms every capture in LiDAR and Monocular Depth.
- **Monocular performance.** Inference and post-processing now overlap, and the color pipeline was vectorized — higher sustained fps on all three models (Small can reach the full 60).
- **Model switching fixed.** Switching depth models now fades to black with a loading spinner (the old model is unloaded first) instead of freezing on a stale frame.
- **Fixes.** Mono's RGB view no longer renders upside down in a stretched box; LiDAR's rear-camera RGB view is upright again; leaving Monocular Depth to the menu no longer leaves the camera light on.

## v9.14

- **Show blackout — screen off, streams on.** Turn the screen off (brightness to zero, touch blocked, preview rendering suspended) while NDI, OSC and recording keep running — a real battery saver for rigged phones and long installs. Trigger it remotely with `/tdlidar/show/screen` (a **Screen Off** toggle on the `tdlidar_show` operator's Output page) in LiDAR / Monocular Depth / Point Cloud, or with the new **screen-off button** in Sensors mode. **Triple-tap** the dark screen to wake.

## v9.13

- **Monocular Depth targets 60 fps.** All three cameras (Front / 1× / 0.5×) now pick 60 fps-capable formats and lock the frame rate so auto-exposure can't sag the sensor to 24 fps.
- **What you see is what records (Monocular Depth).** The preview, the recording and the NDI stream are now fed from one processed frame — the **Sharpen** effect and the chosen **Resolution** finally show up in recordings, exactly as on screen.
- **Faster shutter.** Hold-to-record now arms in **0.37 s** (was ⅔ s) in LiDAR and Monocular Depth; release keeps recording, tap stops.
- **RGB / Depth switch.** The live camera⇄depth button is now labeled by what you'll switch **to** ("RGB" / "Depth") with a morphing person icon, and no longer renders the camera view upside down.

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
