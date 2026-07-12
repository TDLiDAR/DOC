---
title: Monocular Depth Mode
layout: default
parent: App Guide
nav_order: 7
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

## The camera cycle

A camera button cycles through up to three cameras: **Front** (the TrueDepth/selfie camera, run at its full field of view with Center Stage switched off so you get the maximum wideness), **1x** (the main back wide camera), and **0.5x** (the back ultra-wide camera, on phones that have one — it's skipped on the cycle if the device doesn't). Switching cameras resets the temporal filter's history, since the old smoothing state described a different view.

## Auto-adjust

Auto-adjust automatically rides the Brightness/Contrast/Gamma sliders so a scene stays readable as lighting or subject distance changes. It is driven primarily by the depth output itself — how close the framed subject is, sampled from the centre of the frame — with the camera's own light reading (ISO/shutter) folded in as a much smaller secondary input. As a subject gets closer or the scene gets brighter, the sliders come **down** (toward less contrast/gamma) rather than up, so a face filling the frame keeps its features instead of blowing out to solid white. A **Sensitivity** control sets how big a change in distance/light it takes before auto-adjust reacts.

## Manual controls

Whether or not auto-adjust is on, the same sliders are available by hand: **Brightness**, **Contrast**, **Gamma**, **Invert**, plus a colour map picker shared with LiDAR mode. **Sharpen** runs an unsharp pass over the on-screen depth so object edges read crisper in the preview. **Smoothing** sets how strongly the temporal filter blends frame to frame — higher values steady a noisy scene without smearing real motion, thanks to the edge-aware release built into the filter. **Alpha Mask** with a threshold makes the farthest depth values transparent, useful for keying the subject out of the background downstream.

## Output

Monocular Depth streams over **NDI** under its own fixed source name, separate from the LiDAR depth and RGB camera NDI sources, so it's easy to pick out on the network even when other modes are also broadcasting. A **Resolution** picker (shared with LiDAR mode) sets the size the depth is upscaled to for the NDI stream and recordings. It also has its own on-device recorder — start/stop a recording the same way as LiDAR mode, and finished clips save to the Photos library the same way.

## What you can do with it

Anything you'd do with the LiDAR depth visual — colour-mapping, keying, driving displacement — works the same way, but from a phone with no depth hardware at all, or from the front TrueDepth camera in situations LiDAR mode's back-camera-only depth can't reach. It trades sensor-measured precision for camera-agnostic reach: pick LiDAR mode when you have LiDAR and want the most accurate depth, and Monocular Depth when you need depth from a front camera, an older phone, or the ultra-wide.
