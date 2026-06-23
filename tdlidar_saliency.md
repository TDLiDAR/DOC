---
title: "Saliency"
parent: Operators
---
# Saliency — `tdlidar_saliency`

> Let the phone decide where the eye is drawn in a scene, then auto-aim a spotlight, mask or camera at that spot.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** any iPhone (rear camera)

## What it does
Runs Apple's attention-based saliency on the live camera feed and reports a single box around the most eye-catching region of the image. You get the box's centre, its width and height, and a confidence value — all normalized 0–1. Whatever the algorithm thinks a viewer would look at first, this op tracks. Great for unattended installs where you want a light or mask to "follow interest" instead of a fixed point.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/saliency/x` | float | box centre X, normalized 0–1 | camera rate |
| `/tdlidar/saliency/y` | float | box centre Y, normalized 0–1 (TD flips Y) | camera rate |
| `/tdlidar/saliency/w` | float | box width, normalized 0–1 | camera rate |
| `/tdlidar/saliency/h` | float | box height, normalized 0–1 | camera rate |
| `/tdlidar/saliency/conf` | float | confidence 0–1 | camera rate |

## Outputs
`out1` (CHOP) — five channels: `tdlidar/saliency/x`, `…/y`, `…/w`, `…/h`, `…/conf`. The node tile previews `out1`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, enable **Saliency** (Camera & Vision).
2. Point the rear camera at a busy scene and drop the **Saliency** op.
3. Watch `x`/`y` drift toward the most interesting part of the frame; `w`/`h` describe the box size.
4. Wire `x`/`y` into the translate of a small bright Circle TOP (over your camera feed) to see the attention point as a moving dot.

## Advanced patterns
- **Auto-aim a spotlight/mask:** map `x`/`y` (0–1) through a **Range CHOP** into your stage or UV coordinates and feed a radial gradient / Circle TOP used as a mask. Drive its radius from `w`/`h` so the highlight grows with the salient region.
- **Lag the centre (essential):** raw saliency jitters frame-to-frame. A **Lag CHOP** (0.15–0.3) on `x`/`y` gives a smooth, cinematic drift instead of a twitchy dot.
- **Confidence gate:** **Logic CHOP** on `conf` (e.g. ≥ 0.3) — only move the aim when the model is sure, and **Hold CHOP** the last position when it isn't, so the light doesn't wander during dropouts.
- **Crop-to-interest:** feed `x`/`y`/`w`/`h` into a Crop or Transform TOP on the NDI camera stream to auto-frame the most salient area for a picture-in-picture.

## Gotchas
- All values are **normalized 0–1**, not pixels — scale by your resolution before using as pixel positions.
- **TD flips Y** relative to the source: if the dot tracks upside-down vertically, use `1 - y` in a Math CHOP.
- The box can hop between competing regions; always **Lag** the centre (and ideally gate on `conf`) before driving anything visible.
- Saliency reflects *visual* attention only — it has no idea what's important to your show. Combine with another sensor (e.g. `tdlidar_rect_detect`) when you need semantic targeting.
