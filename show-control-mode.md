---
title: Show Control
layout: default
nav_order: 8
---

# Show Control

Show Control is the reverse of everything else in this app: instead of the phone sending data out, it **listens** for commands and reacts. Turn it on and any system that already speaks OSC or MIDI in a show rig — a lighting desk, QLab, MIDI Show Control, Bitfocus Companion / Stream Deck, or a DAW's MIDI clock — can switch the phone's mode, recall a saved look, start/stop recording, drive a live parameter, or trigger a capture step, all without anyone touching the device.

It is not a mode you pick in the mode switcher. It runs independently of whichever mode is active, so a remote mode-switch command works even from the main menu.

## Turning it on

Open the **gear icon** on the main menu (the same "Network" sheet that sets the TouchDesigner/OSC destinations) and find the **Show Control** section:

- **Enabled** — starts/stops the listener.
- **Listen Port** — UDP port Show Control listens on. Default **9200**, separate from the sensor OSC bus (9000) so the two never collide.
- **Status Heartbeat** — when on, the phone sends its own state back out once a second (see [Status heartbeat](#status-heartbeat-out)).
- **MIDI** — shows the assembled MIDI timecode (if any is arriving) and a **Pair Bluetooth MIDI device…** button that opens Apple's own Bluetooth MIDI picker. Network MIDI (Wi-Fi) needs no pairing — any Network MIDI session on the same network connects automatically once Show Control is enabled.

## OSC in — `/tdlidar/show/*`

All UDP, to whatever port Show Control is listening on (default 9200).

| address | args | does |
|---|---|---|
| `/tdlidar/show/recall` | int | Recalls a saved [Look Preset](#look-presets) by its index in the list. |
| `/tdlidar/show/mode` | string or int | Switches the app's mode. **Restricted to the three live-output modes** — string matches `ndi` (LiDAR), `monocularDepth`, or `pointCloud`; int indexes that list in that order (`0` LiDAR, `1` Monocular Depth, `2` Point Cloud). The utility modes (Sensors, Scene Build, Mesh Cloud, Cue Deck, Align) are deliberately **not** remote-switchable and are ignored. |
| `/tdlidar/show/record` | int/float/bool (0 or 1) | Starts or stops recording in LiDAR or Monocular Depth mode — whichever is currently active. No-op in other modes. |
| `/tdlidar/show/ndi` | int/float/bool (0 or 1) | Toggles the **persistent-NDI master switch** (same one as "Keep NDI running across modes" in the Network sheet). `1` starts the shared NDI stream and arms whichever of LiDAR / Monocular Depth / Point Cloud is active, so the feed comes up on command; `0` stops it. |
| `/tdlidar/show/resolution` | int, 0–3 | Sets the NDI output resolution for **all three modes at once**: `0` Low, `1` Med, `2` High, `3` Max. LiDAR + Monocular Depth share one upscale ladder (≈800 → 1920 wide); Point Cloud has its own (540 → 1440 wide). |
| `/tdlidar/show/alpha` | int/float/bool (0 or 1) | Toggles the alpha mask in **all three modes at once** — LiDAR depth-mask alpha, Monocular Depth alpha mask, and Point Cloud background removal. `1` on, `0` off. |
| `/tdlidar/show/reset` | — (pulse) | **Factory-reset every setting** to its default — the same as the in-app "Reset All Settings". Restores every editable parameter across all modes; your Pro unlock and saved captures are untouched. Any argument (or none) triggers it. |
| `/tdlidar/show/screen` | int/float/bool (1 = off, 0 = wake) | **Show blackout** — turns the screen off (brightness to zero, all touch blocked, preview rendering suspended) while **NDI, OSC and recording keep running**. A real battery saver for long installs. Wake remotely with `0`, or **triple-tap** the dark screen on the device. Works in LiDAR, Monocular Depth and Point Cloud modes (and anywhere else — it's app-wide). |
| `/tdlidar/show/capture/finish` | — | Mesh Cloud: ends the current scan (same as tapping Finish). |
| `/tdlidar/show/capture/rescan` | — | Mesh Cloud: discards the finished scan and starts over. |
| `/tdlidar/show/capture/start` | — | Point Cloud: starts TCP streaming. |
| `/tdlidar/show/capture/stop` | — | Point Cloud: stops TCP streaming. |
| `/tdlidar/show/param/<key>` | float 0–1 (or ≥0.5 for toggles) | **Any** LiDAR / Monocular Depth / Point Cloud setting — see the full per-mode reference in [Every setting over OSC & MIDI](#every-setting-over-osc--midi) below. |

## Every setting over OSC & MIDI

**v9.11 — every user-facing setting in the three live modes is a remote parameter.** Each `/tdlidar/show/param/<key>` takes a **normalized `0–1`** value (matching a MIDI CC's `0–127`) that the phone maps to the control's real range; on/off toggles use a `0.5` threshold; enums/indexes map `0` → first option … `1` → last. The `<key>` is the setting's own name, grouped by mode below, and it's the same vocabulary a MIDI CC can target. They apply live while that mode is on screen (Point Cloud values also persist for the next time you enter it).

> **Changed in v9.11:** the historical shared faders (`gamma`, `contrast`, `brightness`, `threshold`, `colorMapIndex`) are now **LiDAR-only and normalized `0–1`** — they no longer also write the Monocular values, and no longer take raw units. Monocular Depth has its own `mono…` keys (e.g. `monoGamma`). If you drove these from a raw-value OSC source, send `0–1` now (the `tdlidar_show` operator already does). This also corrects MIDI, whose CCs always sent `0–1`.
{: .note }
#### LiDAR — 45 controls

| address (`/tdlidar/show/param/…`) | control | type |
|---|---|---|
| `gamma` | Gamma | 0–1 |
| `contrast` | Contrast | 0–1 |
| `brightness` | Brightness | 0–1 |
| `colorMapIndex` | Colormap | 0–1 |
| `invertDepth` | Invert Depth | on/off (≥0.5) |
| `depthColorSource` | Colour Source | 0–1 |
| `confidenceSmooth` | Confidence Smooth | on/off (≥0.5) |
| `depthLineSpacing` | Depth Line Spacing | 0–1 |
| `depthMode` | Depth Look | 0–1 |
| `threshold` | Far Clip (m) | 0–1 |
| `nearClipMeters` | Near Clip (m) | 0–1 |
| `maxDepthMeters` | Max Range (m) | 0–1 |
| `envNoiseFloor` | Noise Reduction | 0–1 |
| `extendedRange` | Extended Range | on/off (≥0.5) |
| `detailLevel` | Edge Outline | 0–1 |
| `smoothing` | Temporal Smoothing | 0–1 |
| `holeFill` | Smart Hole Fill | on/off (≥0.5) |
| `rawDepth` | Disable Depth Filtering | on/off (≥0.5) |
| `hdSmoothingEnabled` | HD Edge Smoothing | on/off (≥0.5) |
| `hdSmoothingSigma` | HD Smoothing Strength | 0–1 |
| `faceDetailRange` | Face Detail Range | 0–1 |
| `facialDetailGain` | Facial Detail Gain | 0–1 |
| `faceFocusPoint` | Face Focus Point | 0–1 |
| `medianTrackingSpeed` | Tracking Speed | 0–1 |
| `crosshairEnabled` | Measure Crosshair | on/off (≥0.5) |
| `crosshairInNDI` | Crosshair in NDI | on/off (≥0.5) |
| `crosshairOpacity` | Crosshair Opacity | 0–1 |
| `rawColorMapEnabled` | Raw Colour-Map FX | on/off (≥0.5) |
| `rawColorMap` | Raw Colour Map | 0–1 |
| `rawBaseColorR` | Raw Base R | 0–1 |
| `rawBaseColorG` | Raw Base G | 0–1 |
| `rawBaseColorB` | Raw Base B | 0–1 |
| `rawTopographicDensity` | Raw Topo Bands | 0–1 |
| `rawPosterizeBands` | Raw Posterize Bands | 0–1 |
| `rawScanlineStyle` | Raw Scanline Style | 0–1 |
| `rawLineThickness` | Raw Line Width | 0–1 |
| `rawGlitchIntensity` | Raw Glitch Intensity | 0–1 |
| `rawGlitchSpeed` | Raw Glitch Speed | 0–1 |
| `rawReflectionSensitivity` | Raw Reflection | 0–1 |
| `rawBloom` | Raw Bloom | 0–1 |
| `rawCameraControlTarget` | Raw Cam-Control Target | 0–1 |
| `bcDefinition` | Back-LiDAR Definition | 0–1 |
| `bcNearThreshold` | Back-LiDAR Near Threshold | 0–1 |
| `bcNearCompression` | Back-LiDAR Near Compression | 0–1 |
| `bcSmoothing` | Back-LiDAR Smoothing | 0–1 |

#### Monocular Depth — 13 controls

| address (`/tdlidar/show/param/…`) | control | type |
|---|---|---|
| `monoModel` | Depth Model | 0–1 |
| `monoCamera` | Camera | 0–1 |
| `monoColorMap` | Colormap | 0–1 |
| `monoInvert` | Invert Depth | on/off (≥0.5) |
| `monoGamma` | Gamma | 0–1 |
| `monoContrast` | Contrast | 0–1 |
| `monoBrightness` | Brightness | 0–1 |
| `monoSmoothing` | Smoothing | 0–1 |
| `monoSharpen` | Sharpen | on/off (≥0.5) |
| `monoAutoAdjust` | Auto-Adjust | on/off (≥0.5) |
| `monoAutoSensitivity` | Auto Sensitivity | 0–1 |
| `monoAlpha` | Alpha Mask | on/off (≥0.5) |
| `monoAlphaThreshold` | Alpha Threshold | 0–1 |

#### Point Cloud — 83 controls

| address (`/tdlidar/show/param/…`) | control | type |
|---|---|---|
| `pcZoom` | Zoom | 0–1 |
| `pcX` | Offset X | 0–1 |
| `pcY` | Offset Y | 0–1 |
| `pcZ` | Offset Z | 0–1 |
| `pcPivotX` | Pivot X | 0–1 |
| `pcPivotY` | Pivot Y | 0–1 |
| `pcPivotZ` | Pivot Z | 0–1 |
| `pcPoints` | Points | 0–1 |
| `pcSize` | Point Size | 0–1 |
| `pcFov` | Field of View | 0–1 |
| `pcSpin` | Orbit Speed | 0–1 |
| `pcFlat` | Flat Projection | on/off (≥0.5) |
| `pcFreeze` | Freeze FX | on/off (≥0.5) |
| `pcOrbit` | Orbit Mode | 0–1 |
| `pcRotX` | Tilt X | 0–1 |
| `pcRotY` | Tilt Y | 0–1 |
| `pcRotZ` | Tilt Z | 0–1 |
| `pcAppleFilter` | Apple Filter | on/off (≥0.5) |
| `pcSurfaceSmooth` | Surface Smooth | 0–1 |
| `pcAccumulate` | Accumulate | 0–1 |
| `pcGrazingCull` | Grazing Cull | 0–1 |
| `pcFillHoles` | Fill Holes | 0–1 |
| `pcDespeckle` | Despeckle | 0–1 |
| `pcVoxelSize` | Voxel Size | 0–1 |
| `pcDetailUpsample` | Detail Upsample | 0–1 |
| `pcStabilization` | EMA Smoothing | 0–1 |
| `pcEdgeStable` | Edge Stable | 0–1 |
| `pcDepthCurve` | Depth Curve | 0–1 |
| `pcEdgeRelax` | Edge Cleanup | 0–1 |
| `pcParallax` | Parallax | 0–1 |
| `pcViewerColorEnabled` | Show Colour | on/off (≥0.5) |
| `pcShowAxes` | Show Axes | on/off (≥0.5) |
| `pcAxesOpacity` | Axes Opacity | 0–1 |
| `pcShowOrbitCube` | Show Orbit Cube | on/off (≥0.5) |
| `pcOrbitCubeOpacity` | Orbit Cube Opacity | 0–1 |
| `pcMoveSpeed` | Move Speed | 0–1 |
| `pcViewerFlipX` | Flip X | on/off (≥0.5) |
| `pcViewerFlipY` | Flip Y | on/off (≥0.5) |
| `pcViewerFlipZ` | Flip Z | on/off (≥0.5) |
| `pcCameraControlTarget` | Camera Control Target | 0–1 |
| `pcKeepScreenOn` | Keep Screen On | on/off (≥0.5) |
| `pcTrailFrames` | Trails Frames | 0–1 |
| `pcTrailDecay` | Trails Decay | 0–1 |
| `pcTrailRecede` | Trails Stream | 0–1 |
| `pcTrailSpread` | Trails Spread | 0–1 |
| `pcTrailBiasX` | Trails Bias X | 0–1 |
| `pcTrailBiasY` | Trails Bias Y | 0–1 |
| `pcTrailMode` | Trails Curve | 0–1 |
| `pcEchoCount` | Echo Count | 0–1 |
| `pcEchoStride` | Echo Spacing | 0–1 |
| `pcEchoDecay` | Echo Decay | 0–1 |
| `pcEchoZPush` | Echo Z Push | 0–1 |
| `pcFeedbackPersist` | Light Paint | 0–1 |
| `pcLightpaintBand` | LightPaint Band | 0–1 |
| `pcLightpaintThreshold` | LightPaint Threshold | 0–1 |
| `pcLightpaintHardness` | LightPaint Hardness | 0–1 |
| `pcLightpaintMotionGate` | LightPaint Motion Gate | 0–1 |
| `pcVelStretch` | Velocity | 0–1 |
| `pcVelSpectrum` | Velocity Spectrum | on/off (≥0.5) |
| `pcVelColorR` | Velocity Tint R | 0–1 |
| `pcVelColorG` | Velocity Tint G | 0–1 |
| `pcVelColorB` | Velocity Tint B | 0–1 |
| `pcAgeFade` | Age Fade | 0–1 |
| `pcAgeFloor` | Age Edge Dim | 0–1 |
| `pcSculptFrames` | Sculpt Frames | 0–1 |
| `pcSculptLock` | Sculpt Lock | on/off (≥0.5) |
| `pcShearAmount` | Riptide | 0–1 |
| `pcShearPush` | Riptide Push | 0–1 |
| `pcShearResponse` | Riptide Response | 0–1 |
| `pcShearReach` | Riptide Reach | 0–1 |
| `pcEraseAmount` | Vanish | 0–1 |
| `pcEraseThresh` | Vanish Threshold | 0–1 |
| `pcEraseHold` | Vanish Linger | 0–1 |
| `pcEraseInvert` | Vanish Reveal | on/off (≥0.5) |
| `pcGravityRate` | Gravity | 0–1 |
| `pcGravityForce` | Gravity Force | 0–1 |
| `pcGravityLifetime` | Gravity Lifetime | 0–1 |
| `pcGravitySpread` | Gravity Spread | 0–1 |
| `pcGravityZ` | Gravity Z | 0–1 |
| `pcNDIEnabled` | NDI Stream | on/off (≥0.5) |
| `pcNDIResolution` | NDI Resolution | 0–1 |
| `pcNDIRemoveAlpha` | NDI Transparent BG | on/off (≥0.5) |
| `pcTCPEnabled` | TD POP (TCP) | on/off (≥0.5) |

> `pcX/Y/Z` moves the **whole cloud**; `pcPivotX/Y/Z` moves the **orbit centre** the turntable rotates around; `pcRotX/Y/Z` **tilts** the whole cloud (±180°). All are independent so you can compose them. `pcNDIEnabled` / `pcTCPEnabled` start/stop the Point Cloud viewport's own NDI / TCP streams.
{: .note }

> **On the `tdlidar_show` operator**, the handful of discrete controls are **dropdown menus** rather than 0–1 faders — `pcNDIResolution` (Low / Med / High / Max, like the global NDI Resolution), plus `pcOrbit` (Off / Spin / Sway), `pcDetailUpsample` (Off / 2× / 4×), `monoModel` (Small / Medium / High), `monoCamera` (Front / 1× / 0.5×) and `depthMode` (Environment / Face / Raw). They still send the same normalized `0–1` on the wire (menu position ÷ last index), so raw OSC senders can drive them too.
{: .note }

> Mesh Cloud's Send (stream to TD) and Export (save PLY) aren't remote-triggerable yet — they live as private state inside the mode's own 3D viewer, not reachable from outside it. Finish and Rescan are.
{: .note }

## From TouchDesigner — the `tdlidar_show` operator

You don't have to hand-build an OSC Out CHOP for every address above — **`tdlidar_show`** in the operator family is a ready-made control panel for exactly this. Drop it, set **Address** to the phone's IP and **Port** to its Show Control port, and every address in the table above is a parameter:

| parameter page | pars | sends |
|---|---|---|
| Network | Address, Port | (where everything below is sent) |
| Cues | Recall Index + **Recall** pulse | `/tdlidar/show/recall` |
| Cues | Mode + **Switch Mode** pulse | `/tdlidar/show/mode` — menu limited to LiDAR / Monocular Depth / Point Cloud |
| Cues | Record toggle | `/tdlidar/show/record`, on change |
| Cues | Capture Verb + **Send Capture** pulse | `/tdlidar/show/capture/<verb>` |
| Cues | **Reset Settings** pulse | `/tdlidar/show/reset` — factory-reset every setting |
| Output | **NDI Enable** toggle | `/tdlidar/show/ndi`, on change |
| Output | **NDI Resolution** menu (Low / Med / High / Max) | `/tdlidar/show/resolution`, on change |
| Output | **Alpha Mask** toggle | `/tdlidar/show/alpha`, on change |
| Output | **Screen Off** toggle | `/tdlidar/show/screen`, on change — blackout on/off while streams keep running |
| **LiDAR** | 45 controls (tone, clip, detail, face, raw look, back-LiDAR) | the matching `/tdlidar/show/param/*`, normalized on change |
| **Monocular Depth** | 13 controls (model, camera, tone, smoothing, auto-adjust, alpha) | the matching `/tdlidar/show/param/*`, normalized on change |
| **Point Cloud** | 83 controls (view, tilt, cleanup, motion FX, network) | the matching `/tdlidar/show/param/*`, normalized on change |

Each mode tab exposes every setting from that mode as a `0–1` fader (or a toggle), sending the matching `/tdlidar/show/param/<key>` live on change — the same keys listed in [Every setting over OSC & MIDI](#every-setting-over-osc--midi).

`out1` mirrors the current parameter values as a CHOP, in case you want to chain or display what was last sent. It's a thin wrapper — internally just an OSC Out DAT and a Parameter Execute DAT — so the DMX/Art-Net/QLab/Companion patterns below can drive `tdlidar_show`'s own parameters (export a CHOP into **Recall Index**, pulse **Recall**) instead of building a raw OSC Out CHOP by hand, if you'd rather stay in parameter-land.

## Look Presets

A Look Preset is a saved snapshot of LiDAR or Monocular Depth's colormap, tone curve (gamma/contrast/brightness/invert) and mode-specific extras (LiDAR: depth mode + near/far clip; Monocular Depth: model + camera). Build the list in the same Show Control section: **Save LiDAR look** / **Save Monocular look** snapshots whatever that mode is currently set to; tap a saved preset to rename it or re-save it from the mode's current live settings.

Recall one from outside the app with `/tdlidar/show/recall <index>` (index = its position in the list, 0-based) or a MIDI Note On (see below) — the same recall path either way.

## MIDI in

**Network MIDI** (Wi-Fi, `MIDINetworkSession`) is always available once Show Control is enabled — connect to it the same way you'd connect to any Network MIDI session (macOS Audio MIDI Setup → MIDI Studio → Network, or any rtpMIDI-compatible app), no pairing step on the phone.

**Bluetooth MIDI** needs one-time pairing: tap **Pair Bluetooth MIDI device…** in the Show Control section, which opens Apple's standard Bluetooth MIDI picker. Once paired, the device behaves exactly like a Network MIDI source.

Either transport decodes the same two message types:

- **Note On** (velocity > 0) — recalls a Look Preset by note number, same as `/tdlidar/show/recall`.
- **Control Change** — drives a live param, mapped through an editable CC list. Ships with a default mapping so a generic MIDI controller works immediately:

| CC | drives |
|---|---|
| 1 | gamma |
| 2 | contrast |
| 3 | brightness |
| 4 | threshold |

The CC target is any key from the same parameter vocabulary as OSC — so **every** setting listed in [Every setting over OSC & MIDI](#every-setting-over-osc--midi) (all 141 LiDAR / Monocular / Point Cloud controls) is equally MIDI-mappable: a CC's `0–127` maps straight onto the same normalized `0–1` range.

**MIDI Time Code** (MTC) quarter-frames are assembled into a running `HH:MM:SS:FF` timecode, shown live in the Show Control section — useful for confirming the phone is receiving a show's clock. Show Control does not yet fire commands at specific timecodes (that's a real, separate cue-scheduler feature — a timecode-indexed cue list with its own editor — not built yet); today, MTC reception is for monitoring, and cueing is done via `/tdlidar/show/recall` or MIDI Note On.

## Status heartbeat out

With **Status Heartbeat** on, the phone sends its own state back out once a second to the **Point Cloud TCP host** (the TouchDesigner IP you set in Point Cloud mode), on **port 9000**:

| address | args | meaning |
|---|---|---|
| `/tdlidar/show/status/battery` | float | battery level, 0–1 |
| `/tdlidar/show/status/thermal` | int | 0 nominal, 1 fair, 2 serious, 3 critical |
| `/tdlidar/show/status/depthMode` | string | LiDAR mode's current depth mode |

Useful for a Companion or QLab dashboard that wants to show the phone is alive and not overheating, without polling it.

## Bridging from a lighting desk, QLab, MIDI Show Control or Companion

Show Control speaks OSC and MIDI directly — none of the following need anything built into the phone, they're patterns for the *other* end.

**DMX / Art-Net from a lighting desk.** In TouchDesigner: a **DMX In CHOP** or **Art-Net In CHOP** reads the universe, a **Select CHOP** isolates the channel you want to use as a trigger, and an **OSC Out CHOP** sends `/tdlidar/show/recall` (or `/tdlidar/show/param/threshold`, for a fader) to the phone's IP and Show Control port. This is a TD patch pattern, not a `.tox` — build it in your own network next to whatever else reads that universe.

**QLab / MIDI Show Control cues.** QLab's OSC cues (or an MSC cue relayed through TD) can send `/tdlidar/show/recall <n>` directly to the phone — no TD in the path at all if QLab and the phone are on the same network. Point a QLab Network cue at the phone's IP and Show Control port.

**Bitfocus Companion / Stream Deck.** Add the phone as a **Generic OSC** device in Companion (Connections → Generic OSC): host = the phone's IP, port = the Show Control listen port. Then any button can send `/tdlidar/show/recall`, `/tdlidar/show/mode`, `/tdlidar/show/record` or `/tdlidar/show/capture/*` as its action, exactly like the addresses above. There is no dedicated TDLidar Companion module — the Generic OSC device covers everything Show Control exposes without one.

## Needs

Nothing beyond the app's normal network setup for OSC and Network MIDI. Bluetooth MIDI needs a one-time pairing via the button above. Works on any iPhone.
