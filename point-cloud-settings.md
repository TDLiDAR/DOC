---
title: Point Cloud — Effects & Settings
layout: default
parent: App Guide
nav_order: 7
---

# Point Cloud — Effects & Settings

This is the complete reference for every control that ships in **Point Cloud mode** — the third top-level mode in TDLiDAR. Point Cloud mode unprojects the phone's LiDAR depth into real 3D points (each carrying an exact 32-bit position and an optional colour), renders them in an on-device viewer you can orbit, and streams them to TouchDesigner over NDI (rendered video) or TCP (bit-exact geometry). It also exports a single frozen cloud as a PLY file.

Every control below exists in the shipping app. There are two places to reach them:

- The **live Settings Bar** — a slide-up bar over the capture buttons (tap **Settings** to open, swipe ↓ to close). It's a horizontal scroll of boxes; tap a box and the stage below morphs into that control. Edits apply to the live cloud immediately — you watch the cloud change as you drag.
- The **full settings sheet** — reached from the ⓘ on the bar (or the gear). Same controls, grouped into sections with the descriptive footer text.

Both surfaces bind the same `AppSettings.pc*` values, so a change in one is reflected in the other and persists across launches.

> Point Cloud mode is a **Pro feature**. The entire mode — viewer, cleanup pipeline, NDI/TCP streaming and PLY export — is gated behind the Pro unlock. Everything documented on this page therefore requires Pro.
{: .important }

---

## Viewer & framing

These controls change how the cloud looks **on the phone**. With one exception (Points), they do not change the geometry you stream over TCP — they're camera and presentation settings for the on-device viewer. Drag one finger to orbit, pinch to zoom, two fingers to pan.

### Points (density)

The master control for how many points the cloud carries each frame. It drives the live render, PLY/MP4 captures, **and** the TCP stream to TouchDesigner — one budget for everything. Internally it picks a grid stride across the depth map (`stride = √(depthW·depthH / density)`), so lower values group points into coarser, quantised clusters and higher values separate them into finer LiDAR detail.

- **Default / range:** 20,000 · 500 then 1,000 → 50,000 in 1,000-point steps. (The live ruler snaps to the nearest 1,000, with a 500 floor.)
- **Effect:** Low = coarse, blocky, fast. High = dense, detailed, heavier.
- **Perf cost:** High. The point count is the single biggest driver of GPU/CPU and (for TCP) network load. Higher counts lower frame rate and raise bandwidth proportionally.
- **When to use:** Push it as high as your scene and machine comfortably carry. Drop it to keep the frame rate up on busy scenes or constrained networks.

> Enabling TCP streaming auto-raises the budget to a safe 20,000 if it was lower (it never lowers an already-higher choice). Raise it further to 50,000 right here. See [Streaming](#streaming).
{: .note }

### Point Size

A multiplier on each point's base dot size in the viewer. 100% is the per-camera default (the front and back cameras start from different base sizes); drag to expand or shrink from there.

- **Default / range:** 10% (`0.10`) · 0.1–1.5 (i.e. 10%–150% of base).
- **Effect:** Larger dots fill gaps and read as a solid surface; smaller dots look sparse and pointillist.
- **Perf cost:** Negligible (a sprite-size change).
- **When to use:** Grow it when a sparse cloud looks gappy; shrink it for a fine, airy look.

### Zoom

A **dolly** — it moves the viewer camera nearer or farther at a fixed 60° internal field, so the view scales cleanly with no perspective warp. Right = closer/bigger. The slider is shown as a 0–100% remap of an underlying camera distance.

- **Default / range:** ~70% (camera distance `1.11` m, remapped from the ~0.3–3.0 m range).
- **Effect:** Frames the cloud bigger or smaller without distorting it.
- **Perf cost:** None.
- **When to use:** Quick framing. Pinch-to-zoom on the viewer does the same thing live.

### Field of View

The viewer camera's lens angle, separate from Zoom. Lower = telephoto (flatter, less perspective); higher = wide-angle (stronger depth fan).

- **Default / range:** 120° · 20–120°.
- **Effect:** Changes how strongly near points loom over far points. Pairs with **Flat Projection** below.
- **Perf cost:** None.
- **When to use:** Drop toward 20° for an orthographic-looking, undistorted read; raise for a dramatic, immersive perspective.

### Flat Projection

Lays the points on a flat grid so the cloud doesn't fan out into a wedge.

- **Default:** Off (true camera perspective).
- **Effect:** On = points sit on a flat plane (good for a clean, map-like read); Off = real perspective depth.
- **When to use:** Turn on when the perspective fan is distracting and you want a flatter, more even layout.

### Axis flips (Flip X / Y / Z)

Three toggles that mirror the **viewer** along each axis by scaling the render node by −1 on that axis. Crucially, they mirror only what you see on screen — the underlying point positions, and therefore the TCP/NDI/PLY data, are **unchanged**.

- **Default:** All toggles read "off." Note: an X-flip is baked on by default as the orientation fix, so the X toggle's "off" state is already the corrected orientation; switching X **on** un-flips it.
- **Effect:** Corrects a cloud that reads mirrored relative to how you expect to see it.
- **Perf cost:** None.
- **When to use:** Flip the matching axis when left/right, up/down or near/far come in reversed in the preview.

> Because the viewer flips are presentation-only, they will **not** fix a mirrored cloud on the TouchDesigner side. If TD shows the cloud mirrored, correct it in TD (or with a POP transform), not with these toggles. See [per-axis guidance](#per-axis--mirroring-guidance).
{: .warning }

### View-warp sliders: Depth Curve, Edge Cleanup, Parallax

Three subtle render-space warps surfaced here so you don't need to enter Edit mode to reach them. Each is a 0–1 slider that warps how the existing points are drawn — they reshape the view, they don't add or remove points.

- **Depth Curve** — bends the depth response. **Default `0.50`** (neutral). Lower flattens the depth spread; higher exaggerates it.
- **Edge Cleanup** (Edge Relax) — relaxes harsh silhouette edges in the render. **Default `0.0`** (off).
- **Parallax** — adds a parallax shift to the view. **Default `0.0`** (off).
- **Perf cost:** Negligible (view-space only).
- **When to use:** Fine aesthetic tuning of the preview/NDI look.

### Freeze Effects

Holds the current view-warp / effect state instead of letting it track live.

- **Default:** Off.
- **When to use:** Lock a look you like so it doesn't drift while you keep moving.

### Show Colour

Switches the viewer between colour and monochrome points. Colour samples the RGB camera pixel at each point.

- **Default:** Off (monochrome).
- **Note:** **Detail Upsample requires Show Colour on** — it uses the RGB image as its guide (see [Detail Upsample](#a3--despeckle-voxel--detail-upsample)).
- **When to use:** Turn on for natural-colour clouds; leave off for a clean wireframe/geometry read.

### Keep Screen On

Prevents the display from auto-locking while you're in Point Cloud mode.

- **Default:** Off (streaming over NDI or TCP also keeps the screen awake on its own).
- **When to use:** Long installs/performances where you're not touching the phone but need the cloud visible.

### On-screen chrome: axes, cubes, opacity

A set of optional reference helpers drawn in the viewer:

- **Show XYZ Axes** — master switch for a coloured X/Y/Z axis gizmo at the world origin. **Default: on.**
- **Centre Axes (Axes Opacity)** — a faint X/Y/Z ruler that appears at the centre while you move the view. **Default `0.0` (off)**; drag up to fade it in (range 0–1). Drag back to 0 to hide it.
- **Show Phone Cube** — draws a small cube marking the phone's position in the scene. **Default: off.**
- **Show Orbit Cube** — a red cube marking the pivot the view orbits around. **Default: off.** When on, you also get **Cube X/Y/Z** position sliders (−1…+1 m, reaching the ends of the 1 m axes) and **Cube Transparency** (`0.1`–`1.0`, default `0.5`).
- **When to use:** Turn axes/cubes on while framing and aligning, off for a clean output. None of these are streamed as geometry — they're viewer overlays.

### Cloud Position (X / Y / Z)

Moves the whole cloud relative to the 0,0,0 axis centre.

- **Range:** −1…+1 m per axis. **Defaults:** X `+0.14`, Y `+0.14`, Z `+0.04`.
- **Effect:** Re-centres the cloud on the origin gizmo (useful before recording or for lining up with the orbit cube).
- **When to use:** Nudge the cloud so it sits on the axes centre / orbit pivot you want.

### Cloud Tilt (X / Y / Z)

Rotates the whole cloud about its centre, tilting it on each axis. The world-origin XYZ ruler stays put — only the points tilt (like Cloud Position, but rotating instead of shifting).

- **Range:** −1…+1 per axis (mapped to ±180°). **Default:** `0` on all axes.
- **When to use:** Level a cloud captured off-axis, or deliberately cant it for a look; combine with Cloud Position and the orbit pivot to compose the framing. Also remote-drivable (`pcRotX/Y/Z`).

### Orbit Mode

Automatic camera motion for the viewer. **Default: Off.**

- **Off** — manual only: drag to orbit, pinch to zoom, two-finger pan.
- **Spin** — the camera rotates continuously around the pivot at **Orbit Speed**.
- **Sway** — the camera swings back and forth like a pendulum, ±90° to each side of front-on (a 180° arc), reversing at each end, at **Orbit Speed**.
- Touching the screen pauses the motion so a manual drag always wins; it resumes on release. Also settable remotely (`/tdlidar/show/param/pcOrbit`).

### Orbit Speed & Movement Speed

Gesture sensitivities for the viewer, and the rate of the automatic **Spin / Sway** motion above. The minimum is 0.25× (never 0) so movement is always possible, just slower; 1× is the original feel.

- **Range:** 0.25×–3.0× each. **Defaults:** both `0.5×`.
- **When to use:** Slow them down for precise framing on a tripod; speed up for fast hand-held orbiting or a quicker spin/sway.

### Camera Control Scrolls (iPhone 16+ only)

On hardware with the hardware Camera Control, picks which parameter the Camera Control slider scrolls. Options: **Zoom (distance)**, **Point Size**, **Field of View**, **Point Count**, **Smoothing**, **Axes Opacity**.

- **Default:** Zoom (distance).
- **Availability:** Only shown on devices that have the Camera Control (the box/row is hidden otherwise).
- **When to use:** Drive the most relevant live parameter from the hardware button while tripod-mounted, without touching the screen.

---

## Depth cleanup

Cleanup refines the **live LiDAR depth before it becomes points**, so it improves the render, the captures and both streams at once. It runs as a chain: a set of GPU raster passes operate on the depth map (between the stabilization pass and the unprojection), then a CPU point-pass operates on the final point array.

**Everything bypasses at 0 / off.** Turn it all down for raw, unprocessed sensor data. Most sliders default to 0 (a true no-op until you move them); the two exceptions are noted below.

> Defaults ship at 0/off deliberately — each stage is a no-op until you move its slider. The one tuned default is **Grazing Cull `0.25`** (a gentle always-on silhouette cleanup). Apple Depth Filtering defaults **off**.
{: .note }

### A0 — Apple Depth Filtering

A single toggle that turns on Apple's own built-in temporal depth smoothing + hole-fill (the system depth filter).

- **What it does:** Near-free temporal smooth and gap-fill applied by the OS to the depth map.
- **Visual result:** Steadier, more filled-in depth with very little cost. Apple fills most dropouts itself, which is why **Fill Holes** (below) mainly matters when this is **off**.
- **Default:** Off. · **Pro-only:** Yes. · **Perf cost:** Near-zero (it's the system filter).
- **When to use:** Turn on as a cheap baseline for cleaner depth; turn off if you want the rawest sensor data or are doing your own hole-filling.

### A1 — Surface Smooth + Grazing Cull

Two depth-domain passes that denoise surfaces and strip flying-pixel edges.

- **Surface Smooth** (`pc.surfaceSmooth`, 0–1, default `0`) — a bilateral smooth that denoises grainy surfaces **without rounding edges** (it's edge-aware). Removes the speckly noise on flat walls/objects while keeping silhouettes crisp.
- **Grazing Cull** (`pc.grazingCull`, 0–1, **default `0.25`**) — drops the "flying pixels" that smear off silhouettes at grazing angles (depth samples where the surface is nearly edge-on to the sensor and the reading is unreliable). Cleans up the haze of stray points around object outlines.
- **Visual result:** Cleaner surfaces and tighter, less-smeared edges.
- **Pro-only:** Yes. · **Perf cost:** Low–moderate (one GPU compute pass each, runs at depth resolution).
- **When to use:** Surface Smooth on grainy/low-light scans; Grazing Cull whenever object outlines look fuzzy with stray points (it's gently on by default).

### A2 — Accumulate + Fill Holes

Two more depth passes: one averages static scenes over time, one in-paints dropouts.

- **Accumulate** (`pc.accumulate`, 0–1, default `0`) — temporally averages the depth across frames toward zero noise for **static** scenes. It self-resets on motion (a ~10 cm depth jump resets the history), so a moving subject stays snappy rather than smearing. Moving the slider also resets the accumulation so it never blends against state tuned for a different strength.
- **Fill Holes** (`pc.fillHoles`, 0–1, default `0`) — in-paints invalid/dropped depth pixels, closing gaps in the cloud.
- **Visual result:** Accumulate makes a still cloud go glassy-clean; Fill Holes plugs the black gaps where depth was missing.
- **Pro-only:** Yes. · **Perf cost:** Moderate (Accumulate keeps ping-pong history textures; Fill Holes is one pass).
- **When to use:** Accumulate for tripod/static captures of a still scene. Fill Holes **mainly helps with Apple Depth Filtering off** — Apple already fills most gaps when its filter is on.

### A3 — Despeckle, Voxel & Detail Upsample

The CPU point-pass and the RGB-guided detail sharpener. These act on the final point array rather than the depth map.

- **Despeckle** (`pc.despeckle`, 0–1, default `0`) — radius-outlier removal: drops lone floating points that have too few neighbours nearby. Stronger = tighter radius and a higher neighbour threshold = more aggressive culling. **Live, it's capped to ~10k points** (so it stays cheap); it runs at full strength on frozen captures.
- **Voxel Size** (`pc.voxelSize`, slider 0–5 cm, step 0.01, default `0` = **Off** below 0.005 cm) — voxel downsample: collapses all points inside each cubic cell to one averaged centroid point. Larger cells = fewer, more evenly-spaced points.
- **Detail Upsample** (`pc.detailUpsample`, **Off / 2× / 4×**, default Off) — a joint-bilateral upsample that **sharpens depth edges using the RGB image as a guide**. It holds the point count (it doesn't increase it) and snaps edges to the colour image. **Requires Show Colour on** (it needs the RGB guide).
- **Visual result:** Despeckle removes the lone floaters/fireflies; Voxel evens out density and thins dense clouds; Detail Upsample crisps up the silhouettes against the colour image.
- **Pro-only:** Yes. · **Perf cost:** Despeckle and Voxel are O(n) grid-hash CPU passes (Despeckle is the heavier of the two — hence the live 10k cap); Detail Upsample is a GPU pass at 2×/4× the depth resolution (4× is appreciably heavier than 2×).
- **When to use:** Despeckle to clean fireflies; Voxel to thin or regularise a dense cloud; Detail Upsample (with colour on) when you want the sharpest edges and can spare the GPU.

### Cleanup stages at a glance

| Stage | Control(s) | Domain | Removes / adds | Visual result | Default | Relative cost |
|-------|-----------|--------|----------------|---------------|---------|---------------|
| A0 | Apple Depth Filtering | Depth (OS) | Smooths + fills | Steadier, filled depth | Off | Near-zero |
| A1 | Surface Smooth | Depth (GPU) | Removes surface noise | Clean surfaces, crisp edges | 0 | Low |
| A1 | Grazing Cull | Depth (GPU) | Removes flying-pixel edges | Tight, un-smeared silhouettes | **0.25** | Low |
| A2 | Accumulate | Depth (GPU) | Averages static noise | Glassy-still cloud | 0 | Moderate |
| A2 | Fill Holes | Depth (GPU) | Adds in-painted depth | Closed gaps | 0 | Moderate |
| A3 | Despeckle | Points (CPU) | Removes lone floaters | No fireflies | 0 | Moderate (10k live cap) |
| A3 | Voxel Size | Points (CPU) | Merges to one/cell | Even density, thinner | Off | Low–moderate |
| A3 | Detail Upsample | Depth (GPU, needs colour) | Sharpens edges (count held) | Crisp colour-snapped edges | Off | Moderate–high (2×/4×) |

### Stabilization (Smoothing / EMA)

A single slider that calms the live point jitter (an exponential temporal smooth). Separate from the cleanup passes above.

- **Range:** 0–100% (`pc.stabilization` 0–1, step 0.05). **Default: 0%** (raw, no smoothing).
- **Effect:** 0% = raw; 100% = maximum hold. Static surfaces go still while a moving subject stays snappy. Balanced ≈ 70%.
- **Perf cost:** Negligible.
- **When to use:** Raise it to kill the shimmer on a static scene; keep it low when you want the cloud to track fast motion with no lag.

---

## Streaming

Point Cloud mode can send the cloud out two completely different ways at once. They're independent toggles with independent purposes:

- **NDI** sends the **rendered viewport** as a 30 fps video — what you see on screen, including viewer framing and overlays — discoverable by any NDI receiver.
- **TCP** sends the **bit-exact geometry** (raw XYZ + RGB floats) to TouchDesigner to be rebuilt as a POP.

### NDI viewport stream

Broadcasts the point-cloud viewport at an exact 30 fps as its own NDI source. TouchDesigner, OBS and other NDI tools discover it automatically — no IP needed. The view you orbit on screen is exactly what streams.

- **Stream over NDI** — the on switch (`pc.ndiEnabled`, default off). The resolution / transparent-background controls appear only while it's on.
- **Resolution** (`pc.ndi.resolution`) — portrait 9:16 output size. Options: **540×960**, **720×1280** (default), **1080×1920**, **1440×2560**.
- **Transparent Background** (`pc.ndi.removeAlpha`, default off) — keys out the black so only the dots are sent, **with alpha**, so TouchDesigner can composite the cloud cleanly over other layers.
- **NDI Source name** — `TDLidar PC` (or `TDLidar PC N` when you've set a source-name index, for multiple phones on one network). **Receivers** shows the live connection count.
- **Pro-only:** Yes. · **Network cost:** A compressed video stream — bandwidth scales with resolution. NDI is a lossy 8-bit 4:2:2 codec, so it is **not** geometry; it's a picture of the cloud.
- **When to use:** When you want a ready-to-composite *image* of the cloud (with the viewer's look baked in), or to feed OBS/another NDI tool. Use Transparent Background to drop the black and composite only the dots.

> NDI is the right choice for a finished *look* (it carries your viewer framing, point size, colour and overlays as pixels). It is the **wrong** choice for accurate geometry — the 8-bit video codec is lossy. For real points, use TCP.
{: .warning }

### TCP point stream to TouchDesigner (TD POP)

Streams the **full point cloud** — up to 50,000 points, bit-exact — to TouchDesigner over TCP, where it's rebuilt as POP geometry. This is the high-resolution path that beats the old 2,000-point OSC cap, with no codec loss.

- **Stream over TCP** — the on switch (`pc.tcpEnabled`, default off). The IP/port fields appear only while it's on. **Enabling it raises the point budget to a safe 20,000** if it was lower (raise it to 50k in [Points](#points-density)).
- **TD IP** (`pc.tcpHost`) — your computer's address. Use the network-discovery dropdown to pick the machine by name, or type the IP. If left blank, the app reuses the Point Cloud OSC target IP.
- **Port** (`pc.tcpPort`, default **9002**) — match it to the TCP/IP DAT (server mode) in TouchDesigner.
- **Wire format:** length-prefixed binary frames — a `"TDP1"` magic, a frame id, the point count, then 16 bytes per point (float32 `x,y,z` + uint8 `r,g,b,a`), little-endian. The app is the TCP **client**; TD's TCP/IP DAT listens. TD parses it with `numpy.frombuffer`. Full details in the [TCP streaming wire format]({{ '/tdlidar_pointcloud_tcp.html' | relative_url }}).
- **Pro-only:** Yes. · **Network cost:** ~16 bytes per point per frame, so **bandwidth scales directly with the point count**. At 50,000 points that's ~800 KB/frame before TCP overhead — high counts need a solid LAN. The app drops frames while a send is outstanding (TCP backpressure), so a slow link lowers the effective frame rate rather than building lag.
- **When to use:** Whenever you want real, renderable, lossless geometry in TouchDesigner — to instance geometry onto every point, run a POP chain, or render the points directly.

> **Density vs bandwidth (TCP):** point count is the one dial that trades quality against the wire. Start at the auto-set 20,000; raise toward 50,000 only if the frame rate holds. If TD shows dropped/stuttering frames, the link is the bottleneck — lower the Points budget.
{: .tip }

### NDI vs TCP — which to use

| | NDI viewport | TCP point stream |
|---|---|---|
| Carries | Rendered video (pixels) | Bit-exact geometry (floats) |
| In TouchDesigner | A TOP image | A POP you can render/instance |
| Fidelity | Lossy 8-bit 4:2:2 | Bit-exact float32 XYZ+RGB |
| Viewer look baked in | Yes (framing, size, overlays) | No (raw points only) |
| Cost scales with | Resolution | Point count |
| Default port | (auto-discovered) | 9002 |
| Best for | Compositing a finished *look* | Real 3D geometry work |

---

## PLY capture

Besides streaming, you can capture a single **frozen** cloud to a standard PLY file you can open in Blender, MeshLab, CloudCompare or any 3D tool, or bring back into TouchDesigner later.

- **PLY Density** — independent of the live Points budget. Choose **500 / 1,000 / 2,000 / 5,000** points (default 1,000). The app walks the depth grid at a stride that yields roughly that many samples, so the count is approximate.
- **Save Point Cloud as PLY** — captures the current frame and writes the file. A spinner shows while it captures.
- **What you get:** a vertex-only binary PLY (little-endian) — `xyz` float + `rgb` uchar per vertex (15 bytes/point), **no faces**. It lands in the app's `Documents/PLY-Captures/` folder, timestamped (`TDLidar-<timestamp>.ply`).
- **Pro-only:** Yes. · **Cost:** A one-shot capture; no streaming cost.
- **When to use:** Lower densities (500/1,000) make small, shareable files; higher densities (2,000/5,000) preserve more of the scan for offline work.

> PLY density tops out at 5,000 — it's deliberately separate from the live/TCP budget (which reaches 50,000) because a still file rarely needs that many points and smaller files are easier to share. For a dense full-resolution cloud, use the TCP stream instead.
{: .note }

---

## Per-axis / mirroring guidance

A few rules that keep orientation straightforward:

- The viewer **Flip X / Y / Z** toggles are **presentation-only** — they mirror the on-screen render, not the data. They will **not** change what TCP, NDI or PLY send. An X-flip is already baked on as the orientation fix, so switching X *on* un-flips it.
- Because the flips don't touch the data, the cleanest workflow is: get the orientation right **on the receiving side** (a POP/transform in TouchDesigner, or your 3D tool for PLY), and use the phone flips only to make the **preview** read the way you expect while framing.
- **Cloud Position (X/Y/Z)** moves the cloud relative to the origin gizmo for framing/alignment; it does not mirror anything.
- If the colour looks mirrored relative to the geometry, that's a separate colour-mapping concern, not the viewer flips.

---

## Related pages

- [TCP streaming wire format]({{ '/tdlidar_pointcloud_tcp.html' | relative_url }}) — the exact byte layout the TCP point stream sends, and how to receive it in TouchDesigner.
- [LiDAR settings]({{ '/lidar-settings.html' | relative_url }}) — the depth, tone, colour and NDI controls for the depth-video mode.
- [Point Cloud Mode]({{ '/point-cloud-mode.html' | relative_url }}) — the short overview of the mode and its live screen.
