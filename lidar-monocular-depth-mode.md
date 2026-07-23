---
title: LiDAR + Monocular Depth Mode
layout: default
parent: App Guide
nav_order: 5
---

# LiDAR + Monocular Depth Mode

This mode combines the two things the other depth modes each only have half of. The rear LiDAR knows *real distances* but only samples them on a coarse 256×192 grid, so its edges are blobby and it drops out on dark, glossy and distant surfaces. Monocular Depth resolves fine structure beautifully but has no idea how far away anything actually is. Here, the LiDAR's own depth is fed into a neural depth model on every frame as a **prompt**: the sensor supplies true distance, the network supplies the sharp, image-aligned edges the sensor cannot resolve on its own.

The result is a dense depth image in **real metres**, with edges that follow the picture.

**Requires the rear LiDAR scanner** (iPhone Pro models) to enter the mode at all — that is what supplies the metric prompt, so the mode is greyed out on phones without it. Once you're in, a camera button lets you switch to the **front TrueDepth camera** for close, face-scale depth, using the same prompted-network approach. There is no ultra-wide option: no depth sensor is bonded to that lens, so there is nothing to prompt with. It is a Pro feature.

## Why metric depth matters

Every other depth mode in TDLidar normalises each frame against whatever is currently in view. That is the right choice for colour-mapping and keying, but it means a given grey is a *different* distance from frame to frame and room to room — the image "breathes" as things enter and leave the shot.

This mode does not. You set a fixed **Metric Near** and **Metric Far** range, and every output value maps to a fixed distance inside it, permanently. A subject standing two metres away produces the same grey in your studio as it does on stage. That is what makes the NDI feed genuinely measurable in TouchDesigner rather than merely pretty, and it is what makes the alpha threshold a real distance instead of an arbitrary number.

## The LiDAR points and Auto Range

A fixed lattice of sample points — **17** on the rear camera, **33** on the front TrueDepth camera (the extra points cluster over the middle, where a face sits in a selfie framing) — reads real depth straight from the sensor, independently of anything the network does.

Each point is scored by two things a single reading can't tell apart: how much of its neighbourhood came back a valid measurement at all, and how tightly those returns agree with each other. A point sitting flat on a wall or a performer scores high on both. A point straddling a depth edge, or landing in a dropout on glass or a dark surface, scores low — full coverage but wide disagreement, or no coverage at all.

- **Show Points** draws the lattice live over the preview — each number the point's current reading in metres, coloured by trust: **green** points are trusted and drive the autos, **yellow** are discounted, **red** are ignored (dropouts, glass, depth edges). Brightness tracks the same score, so a solid green number is a reading the sensor is sure of. It's an overlay drawn above the picture — it never appears in NDI, recordings or photos.
- **Auto Range** sets Metric Near and Metric Far automatically from the points the sensor trusts most. It's confidence-weighted, not a plain min/max — a point on a window or straddling an edge barely moves the range, while points flat on the subject or the back wall dominate it. The range then **eases** toward that target rather than snapping to it, because a jump in Near/Far is a visible global brightness step across the whole image.

**Window Lock** is what controls how fast Auto Range follows the scene. Low values re-centre readily, chasing the scene as it changes; high values hold the range steady through a momentary occlusion or someone walking through frame. (The same slider also damps the network's own internal representable window — see Window Lock under Model, below — so one control governs both, and turning it up makes the whole mode noticeably steadier at the cost of taking longer to react to a real change in the room.)

Auto Range is **on by default**. Turn it off to set Metric Near/Far by hand — the sliders stay live and readable either way, so switching off Auto Range mid-show picks up wherever it left the range.

## The fusion sliders

Two sliders in the settings bar (the **LiDAR ↔ RGB** box) set how the sensor and the network are blended, continuously:

- **LiDAR Weight** decides whose *geometry* wins when the sensor and the network disagree about where a surface actually is. At 100% the geometry is pinned to the sensor; at 0% the model's opinion stands. The default is 50% — the two sources weighted equally.
- **RGB Detail** scales the *fine structure* only the network can supply — the edges, folds and faces the 256×192 sensor cannot resolve. 0 is effectively the raw sensor, 1.00 is the stock model output, up to 2× exaggerates it.

The extremes double as inspection views: LiDAR Weight 100% with RGB Detail 0 shows you what the sensor alone sees, and 0% with 1.00 is the network verbatim — so you can still judge whether the fusion is earning its compute in a given room, now with everything in between.

## The controls

Swipe up from the bottom of the screen for the live settings bar. The full settings sheet (the ⓘ button) carries the same controls with explanations, plus a **Reset All Settings** button that restores this mode's defaults without touching any other mode.

### Metric

- **Auto Range** — see above.
- **Metric Near / Metric Far** — the distance range the output covers, from 0.3 m to 10 m. Everything nearer than Near clips to the brightest value, everything past Far clips to the darkest. While Auto Range is on these are shown read-only, so you can watch what the LiDAR is deciding; turn Auto Range off to set them by hand. **Narrow the range for finer resolution**: the full settings sheet shows the exact centimetres-per-step for your current range.
- **Show Points** — see above.

### Model

- **LiDAR ↔ RGB** — the two fusion sliders, described above.
- **Stabilize** — a trust-map filter on the LiDAR prompt, built around self-consistency: a reading that disagrees with its own previous frame is flicker and gets suppressed, while a reading that agrees with itself is treated as a genuine change and let through fully within about 1–2 frames. Small isolated patches and areas surrounded by low confidence are discounted, and everything tightens automatically while the phone is moving fast. Where trust is low, the network's RGB-derived reconstruction carries the area instead of the jittery sensor reading. A three-way switch sets how the LiDAR base itself behaves there — **Damp** (smoothed toward its last stable value), **Hold** (frozen until settled), **Fill** (inherits trusted neighbourhood depth) — plus a strength slider. On by default. **It works better in good light.** A brighter scene gives the LiDAR stronger returns, so readings agree with themselves sooner and earn trust faster; in dim scenes, expect the frame-level health gate below to lean more on the RGB network instead. If the image looks blotchy in low light, that's the sensor running low on signal, not a setting gone wrong — let the health gate do its job, or raise Min Valid LiDAR to hand off to RGB sooner.
- **Window Lock** — how firmly the network's own representable depth window holds still, and (as of this version) also the rate Auto Range eases at. It always *expands* instantly, so something new arriving close to the camera is never clipped; this only damps how fast it shrinks back. Higher is steadier.
- **Window Margin** — how far that window is widened beyond the measured extremes, so the true nearest and farthest surfaces stay comfortably reachable.
- **Min Valid LiDAR** — a whole-image health threshold, not a per-pixel one: the fraction of the frame the sensor needs to be confident about before it's trusted at all. This reads the same confidence score as the point lattice above, not a raw percentage of non-empty pixels — that's deliberate, because with Hole Fill on the sensor map is almost never actually empty, so a raw-pixel guard would never trip. Below the threshold the LiDAR is disregarded **completely** — faded out over about half a second rather than snapped off: the prompt flattens to a neutral scene-median plane (no spatial structure, so the network works from RGB alone), the depth window freezes at its last healthy value, and LiDAR Weight scales to zero in the fusion — the mode runs as a pure RGB monocular model until the sensor earns its way back in. A yellow **"LiDAR low · RGB only"** note appears on the HUD while this is active, so the look change is explained rather than mysterious; between the extremes everything scales continuously with health rather than snapping. If you're seeing that note more than you'd like, lower Min Valid LiDAR to tolerate a noisier sensor, or get the LiDAR a cleaner view of the room.
- **Hole Fill** — on by default. On, the iPhone's own ISP interpolates over LiDAR dropouts before the prompt is built, which matches what the network was trained on and looks better, but leaves almost no gaps for Min Valid LiDAR to measure — turning that guard near-inert. Off, you get raw sensor depth with real dropouts, which is noisier but makes Min Valid LiDAR an honest measurement of what the sensor is actually seeing.

### Output

- **Colormap** — defaults to Black & White; the full 25-map palette shared with LiDAR and Monocular Depth mode.
- **Auto Image** — automatically rides Brightness/Contrast/Gamma as the framed subject's distance changes, the same idea as Monocular Depth's auto-adjust but driven by absolute metres. By default (**Use LiDAR Points** on) it reads the *same* confidence-weighted point lattice as Auto Range, so the two autos share one measurement of the scene — a point on a window or a depth edge devalues itself for both at once. Turn the link off to fall back to a plain centre-crop of the depth output. A **Sensitivity** slider sets how big a change it takes before it reacts.
- **Sharpen & Detail** — one box, two sliders. Sharpen (0–1) is an unsharp pass over the output so edges read crisper; Detail (0–0.1) draws edge outlines, ported from LiDAR mode's control of the same name, running on the metric depth grid so its threshold means the same real distance step it does there.
- Boxes tinted **orange** in the bar mark postprocessing steps — the controls that spend real milliseconds per frame. If you're chasing frame rate, turn those down first.
- **Topo Lines** — contour lines at fixed depth intervals, also ported from LiDAR mode. The slider is spacing and runs backwards (more slider = tighter lines = more of them), and — because this mode's depth is metric — the spacing is genuine real centimetres, shown in the box as you drag it.
- **Invert**, **Brightness**, **Contrast**, **Gamma** — the standard output tone curve, shaping the colour-mapped image without touching the underlying distances.

### Depth

- **Smoothing** — the temporal filter's strength, 0 to 1.9. A 3-tap median plus an edge-aware EMA pass calms shimmer without smearing real motion; past 0.95 the slider adds a second EMA pass rather than pushing the first one harder, since a single pass tops out there. The phone's gyroscope also gates this automatically: during a fast pan or turnaround the filter releases (the gyro knows about the move a frame before the image does), then winds back up once the phone settles — so high Smoothing no longer smears big reframes.
- **Alpha Mask** — with a threshold, keys out the farthest depth values. Because depth is metric here, the threshold is a real distance rather than an arbitrary shade.

### NDI

- **Resolution** — the HD size the depth is upscaled to for the NDI stream and recordings, shared with LiDAR mode.

## The readout

The pill at the top shows frame rate and, next to it, the **live median inference time in milliseconds** — how long the network itself is taking per frame. It's colour-coded: green under 33 ms (comfortably keeping up), yellow under 66 ms, orange above that.

If the phone gets warm, you may see a small **"Warm · refine ÷2"** (or Hot / Very hot, with a higher divider) note under the pill. That's deliberate, not a fault: the app watches thermal state and automatically runs the network less often as the phone heats up. The live LiDAR base keeps updating at full rate regardless, so the picture stays current — it just refreshes its learned detail less frequently until the phone cools.

## Frame rate

The mode runs at **60 fps** on supported hardware — confirmed on device, not just a target. The depth sensor itself is capped near 30 by its format, but the camera is not, so video is the sync master: the picture runs at the camera's full rate, riding the live LiDAR base, while the network refreshes its detail asynchronously underneath — a full-rate picture with detail catching up as fast as it can, typically around 45 times a second. Where a device has no 60 fps depth-capable camera format, the mode automatically falls back to the best rate the hardware offers.

What this means in practice: motion is always at full frame rate; inference speed shapes how often the fine detail updates, never how smooth the picture is.

## Camera

A camera button toggles between the two cameras this mode can prompt with: the **rear LiDAR** (back wide, labelled 1x) and the **front TrueDepth camera**. There is no ultra-wide option — no depth sensor is bonded to that lens, so there's nothing to prompt the network with. Switching cameras is a different physical sensor, so it rebuilds the session and resets the point lattice, the temporal filter and the depth window, since they all describe a view that no longer applies.

The front TrueDepth camera is denser and more accurate up close but returns little past 2–3 m, which is why it uses the 33-point lattice clustered toward the centre and generally wants a much narrower Metric Near/Far than the rear camera. It's also further from the network's training data than the rear LiDAR, so expect the network to contribute a bit less out of a selfie framing than it does out back.

## NDI and recording

A dedicated NDI source named **TDLLM** so receivers can pick this feed out alongside the depth, camera and side-by-side senders, plus tap-to-photo and hold-to-record to the gallery. A photo saves exactly what is on screen — the fused depth render — as a single image. Ghost contours that used to trail moving edges (the last inference's detail lingering at its old position) are automatically suppressed during motion and repaint within a frame. If you lock the screen or switch apps, the mode rebuilds itself automatically on return — the same thing that happens if you leave the mode and come back in, which is what reliably comes back clean. The loading indicator appears the instant you return, and the fresh session is usually back within a couple of seconds; that brief wait is normal, not a fault.

Because the depth is metric and the range is fixed, the 8-bit NDI feed is directly interpretable in TouchDesigner: a pixel value of *v* (0–255) corresponds to a distance of `Far − (v / 255) × (Far − Near)` metres, with Brightness/Contrast/Gamma at neutral and Invert off. Note the values are inverted relative to raw metres — near reads bright — so the image matches every other depth mode in the app.

## Show Control

All of this mode's settings are exposed over OSC and MIDI on their own tab. See [Show Control Mode](show-control-mode.html) for the addresses.

Two of them deserve a warning before you put them on a fader: **Metric Near** and **Metric Far** are among the only parameters in TDLidar whose normalised 0–1 maps to an absolute physical distance rather than a look. Moving them mid-show re-maps what every output value means, and TouchDesigner will see the depth scale change under it. That is sometimes exactly what you want — but it is not a "look" control. If you'd rather TouchDesigner manage the range instead of a fader drifting it, remember Auto Range is doing that job on the phone already.
