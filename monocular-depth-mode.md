---
title: Monocular Depth Mode
layout: default
parent: App Guide
nav_order: 4
---

# Monocular Depth Mode

Monocular Depth runs live depth estimation from a single ordinary camera — no LiDAR, no TrueDepth, no special hardware. It works on **any iPhone**, front or back camera, and streams a stable colour-mapped depth image over NDI, the same way LiDAR mode does. Where LiDAR mode measures depth directly with a sensor, Monocular Depth *infers* it from the image using an on-device machine-learning model.

## How it works

A CoreML depth model runs on every camera frame and produces a per-pixel depth estimate, which the app colour-maps and streams over NDI exactly like the LiDAR depth visual. Because a single camera has no direct distance measurement, the model's estimate can drift or flicker frame to frame — Monocular Depth applies a temporal filter (a stabilized normalization range, a 3-tap median and an edge-aware smoothing pass) specifically to keep the image steady enough to composite live, without the flashing that raw per-frame monocular depth normally shows.

## The models

The mode switch lets you pick between three models, shown in the app as size/quality tiers — **Small**, **Medium** and **High**. All three produce *relative* depth: values run from nearest to farthest across the scene rather than being real-world distances, which is exactly what you want for colour-mapping, keying and displacement, where relative shape matters more than absolute metres.

- **Small** — the lightest, fastest model. Lowest latency and the highest frame rate, so it's the one to reach for on older phones or whenever you want the smoothest live feel. Depth is a little coarser than the larger tiers.
- **Medium** — the balanced default: a good mix of detail and speed on most phones.
- **High** — the most detailed and most stable depth, at the highest compute cost. Best on newer phones where you can trade some frame rate for cleaner edges and steadier depth; it's the heaviest of the three, so expect it to run slower than Small or Medium.

Higher tiers cost more per frame, so keep an eye on the **fps** readout at the top of the screen and pick the tier that holds a frame rate your composite is happy with.

Switching models **unloads the old model first** — the view fades to black with a "Loading depth model…" spinner until the new model's first frame arrives (the first-ever load of a model can take several seconds while iOS compiles it; later loads are fast).

## The camera cycle

A camera button cycles through up to three cameras: **Front** (the TrueDepth/selfie camera, run at its full field of view with Center Stage switched off so you get the maximum wideness), **1x** (the main back wide camera), and **0.5x** (the back ultra-wide camera, on phones that have one — it's skipped on the cycle if the device doesn't). Switching cameras resets the temporal filter's history, since the old smoothing state described a different view.

## Auto-adjust

Auto-adjust automatically rides the Brightness/Contrast/Gamma sliders so a scene stays readable as lighting or subject distance changes. It is driven primarily by the depth output itself — how close the framed subject is — with the camera's own light reading (ISO/shutter) folded in as a much smaller secondary input. As a subject gets closer or the scene gets brighter, the sliders come **down** (toward less contrast/gamma) rather than up, so a face filling the frame keeps its features instead of blowing out to solid white. A **Sensitivity** control sets how big a change in distance/light it takes before auto-adjust reacts.

Rather than a blind centre crop, the driver reads a fixed lattice of sample points across the frame — **17** on the rear cameras, **33** on the front camera (denser toward the middle, where a face sits in a selfie framing). Each point is weighted by local agreement — how tightly its neighbourhood's readings agree with each other — so a point sitting on a depth edge or in cluttered detail devalues itself automatically rather than dragging the average toward noise.

**Tune From** picks which part of the lattice actually drives the tuning:

- **Centre** — just the single centre dot, closest to the old behaviour.
- **Corners** — the four corner dots, useful when the subject fills the middle of the frame and you want the tuning to track the background instead.
- **Middle** — the eight dots in the middle area around the centre.
- **All** — the whole lattice, confidence-weighted — the default, and the most representative of the whole scene.

**Show Points** draws the lattice live over the preview, each dot showing its current 0–1 nearness reading, coloured by agreement — the same overlay LiDAR + Monocular Depth mode has. It's a SwiftUI layer drawn above the picture, so it's structurally incapable of showing up in NDI, recordings or photos — visible on screen only, never in the output.

## Manual controls

Whether or not auto-adjust is on, the same sliders are available by hand: **Brightness**, **Contrast**, **Gamma**, **Invert**, plus a colour map picker shared with LiDAR mode. **Sharpen** runs an unsharp pass over the on-screen depth so object edges read crisper in the preview. **Smoothing** sets how strongly the temporal filter blends frame to frame — higher values steady a noisy scene without smearing real motion, thanks to the edge-aware release built into the filter. **Alpha Mask** with a threshold makes the farthest depth values transparent, useful for keying the subject out of the background downstream.

## Output

Monocular Depth streams over **NDI** under its own fixed source name, separate from the LiDAR depth and RGB camera NDI sources, so it's easy to pick out on the network even when other modes are also broadcasting. A **Resolution** picker (shared with LiDAR mode) sets the size the depth is upscaled to for the NDI stream and recordings. It also has its own on-device recorder — the same **shutter** as LiDAR mode: a quick tap saves a photo, and a tap-and-hold (0.37 s) starts a recording; let go and it keeps going, tap to stop. Finished clips save to the Photos library the same way.

A photo saves whichever view is on screen — the colour-mapped depth render, or the plain camera view if you've flipped to RGB — as a single image. The depth photo is true 3:4, matching what you see live (earlier versions saved it vertically squashed).

**What you see is what records.** The on-device preview, the recording and the NDI stream are all fed from the **same processed frame** — the Sharpen effect and the chosen Resolution apply to all three identically, so a recording looks exactly like the screen. The camera also targets a locked **60 fps** on all three cameras (the model then runs as fast as the phone can sustain).

An **RGB / Depth** button (Pro, right of NDI) flips the whole output live between the colour-mapped depth and the plain RGB camera — preview, NDI and recording all switch together, instantly, even mid-stream. Its label and icon show what you'll switch **to** ("RGB" + a person icon while on depth; "Depth" + a dotted-background person while on RGB). It starts on depth each time you enter the mode.

**Hardware buttons.** The volume buttons and (on iPhone 16) the Camera Control click can each run an action here — Photo (default), Record, RGB / Depth switch, Camera Flip, or Adjust (volume nudges the Camera-Control slider value). Set them under Settings → **Hardware Buttons**; while an action is set the volume buttons don't change system volume in this mode. On iPhone 16 the **Camera Control slide** also scrolls a setting of your choice — Colormap (default), Brightness, Contrast, Gamma, Smoothing or Camera — picked with **Camera Control Scrolls** in this mode's settings sheet.

A small **"photo/video saved to gallery"** note appears briefly under the fps pill whenever a capture finishes saving.

## What you can do with it

Anything you'd do with the LiDAR depth visual — colour-mapping, keying, driving displacement — works the same way, but from a phone with no depth hardware at all, or from the front TrueDepth camera in situations LiDAR mode's back-camera-only depth can't reach. It trades sensor-measured precision for camera-agnostic reach: pick LiDAR mode when you have LiDAR and want the most accurate depth, and Monocular Depth when you need depth from a front camera, an older phone, or the ultra-wide.
