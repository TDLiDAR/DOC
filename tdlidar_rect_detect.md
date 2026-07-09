---
title: "Rectangle Detect"
parent: Operators
---
# Rectangle Detect — `tdlidar_rect_detect`

> Point the phone at a doorway, screen or sign and get its on-screen centre — auto-aim a spotlight or mask at the biggest quad in view.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** any iPhone (rear camera)

## What it does
Runs Apple's Vision rectangle *detection* on the live camera feed and reports how many four-sided shapes it sees plus the centre of the strongest one. Use it when the phone is looking at something rectangular — a poster, a window, a monitor, a card — and you want TouchDesigner to track where that thing is in frame without any markers. The count doubles as a presence gate; the centre is a moving aim point.

> This is the **detector**, not the display "Rectangle" op (that one just sizes a TOP to a channel). Different tool, similar name.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/detect/rect/count` | float | quad count (0, 1, 2 …) | camera rate |
| `/tdlidar/detect/rect/cx` | float | centre X, normalized 0–1 | camera rate |
| `/tdlidar/detect/rect/cy` | float | centre Y, normalized 0–1 | camera rate |

## Outputs
`out1` (CHOP) — three channels: `tdlidar/detect/rect/count`, `tdlidar/detect/rect/cx`, `tdlidar/detect/rect/cy`. The node tile previews `out1`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the TDLiDAR app, enable **Rectangle Detect** (Camera & Vision).
2. Point the rear camera at a clear rectangle — a laptop screen, a framed picture, a doorway.
3. Drop the **Rectangle Detect** op. When a quad is found `count` ticks to 1 and `cx`/`cy` settle on its centre.
4. Wire `cx`/`cy` into the *translate* of a small Circle TOP (over a Composite TOP) so a dot rides the detected rectangle's centre on screen.

## Advanced patterns
- **Aim something at it:** map `cx`/`cy` (0–1) through a **Range CHOP** into your real coordinate space — e.g. pan/tilt of a virtual light, the offset of a magnifier TOP, or UV of a Displace. A short **Lag CHOP** (0.1) keeps the aim from snapping between competing rectangles.
- **Presence gate:** **Logic CHOP** on `count` (≥ 1) gives you a clean on/off — use it to fade a mask in only while a target is actually in frame, and to hold the last good `cx`/`cy` (Hold CHOP) when detection drops.
- **Pick the dominant quad:** the centre already tracks the strongest detection, but in busy scenes `count` flickers. Filter `count` and only trust the centre when `count` has been ≥ 1 for a few frames (Lag + Logic).
- **Combine with Saliency:** AND this gate with `tdlidar_saliency` so you only react to rectangles that are *also* where the eye is drawn.

## Gotchas
- Centre coords are **normalized 0–1**, not pixels — multiply by your TOP resolution before using them as pixel positions.
- `count` and the centre go **stale** when nothing is detected (they hold the last value); always gate on `count ≥ 1` before trusting `cx`/`cy`.
- Detection prefers high-contrast, fully-visible quads. A rectangle clipped by the frame edge or low-contrast against its background may not register.
- Y origin follows the app's image convention — if a tracked dot moves the wrong way vertically, invert with `1 - cy` in a Math CHOP.
