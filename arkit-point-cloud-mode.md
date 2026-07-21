---
title: ARKit Point Cloud Mode
layout: default
parent: App Guide
nav_order: 7
---

# ARKit Point Cloud Mode

> This feature is in beta — if you experience errors please [report them](https://tdlidar.github.io/DOC/#report).
{: .warning }

ARKit Point Cloud captures every human figure ARKit can see as a live point-cloud silhouette, using person segmentation cross-referenced against the LiDAR depth. It is a **Pro** feature and requires an **iPhone Pro model with the LiDAR scanner** (the same hardware requirement as LiDAR Mode and Mesh Cloud). It is **capture-only** — there is no NDI or OSC output; the point cloud stays on the phone as a photo or video you save to your library.

## How it works

Each scan is **transient and world-locked**: ARKit's person segmentation masks out everyone in frame, the LiDAR depth places those masked pixels in 3D space, and the result is rebuilt as a point cloud fixed in the room — not attached to the camera. Every new scan **fully replaces** the previous one rather than accumulating on top of it, so what you see is always this instant's figures, not a build-up of every pass.

## The live screen and its buttons

The screen shows the point cloud live as ARKit tracks it. **Pause** freezes the last captured frame in place while the camera feed itself keeps running, so you can physically walk around the frozen cloud and view it from other angles before **Resume** picks the scan back up. **Reset** clears the current frame outright. A **torch** toggle is available for scanning in dark rooms, where the camera image alone would otherwise struggle.

## Capturing

The shutter behaves like the other modes: **tap** for a photo, **hold** to start a video and tap again to stop — both save straight to **Photos**.

## Settings bar

Swipe up from the bottom of the screen (or tap the Settings chevron) to open the live settings bar — roughly **60 parameters** across **13 groups**:

- **Scan** — capture interval, depth sync, turn-reject + limit, shake-reject + limit.
- **Cloud** — points budget, quantize/voxel size, point size, 13 point shapes, wobble, frame blend.
- **Color** — Spectrum / Static / RGB camera sampling, hue, speed, gradient, saturation.
- **Clean** — Apple depth filter, confidence, smooth, accumulate, fill, stabilize, upsample, grazing, edge, despeckle, de-float, keep-largest.
- **Trail** — length, decay, recede, spread.
- **Echo** — count, gap, dim, push.
- **Light Paint** — amount, band, fade, hardness, motion gate.
- **Motion** — velocity glow + rainbow, age fade, edge dim.
- **Sculpt** — frames, lock.
- **Riptide** — amount, push, respond, reach.
- **Vanish** — amount, threshold, hold, reveal.
- **Gravity** — rate, force, life, scatter, drift.
- **View** — camera passthrough, cam dim, floor grid, torch, steady pause.

## Needs

ARKit Point Cloud requires an iPhone Pro model with the LiDAR scanner and is gated behind Pro. It has no streaming output (no NDI, no OSC) — capture only, to Photos.
