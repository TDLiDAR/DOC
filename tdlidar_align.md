---
title: "Align"
parent: Operators
---
# Align — `tdlidar_align`

> Point the phone at a wall or screen, tap the surface, and get its real 3D corners on the wire — the rough-in geometry for a TouchDesigner keystone/warp network, without hand-picking correspondence points.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** ARKit world tracking (any recent iPhone; LiDAR speeds up plane detection but isn't required)

## What it does
The app's **Align** mode watches the room with ARKit plane detection and lets the performer tap a detected wall or floor to lock it. Once locked, the phone streams that surface's real-world width, height and 4 corner positions over OSC — measured in metres, not screen pixels. Drop this op, wire the corners to a **camSchnappr**, **Stoner** or your own corner-pin/warp setup, and you have a live starting point for keystoning a projector or screen onto the actual physical surface instead of eyeballing it. Align supplies the geometry; the fine keystone/blend solve still happens in your TD network.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/align/detected` | float | 0 or 1 — is a surface currently locked | ~10 Hz |
| `/tdlidar/align/size` | 2 float | width, height — metres | ~10 Hz, only while locked |
| `/tdlidar/align/corner0` … `/corner3` | 3 float each | x, y, z — world-space metres | ~10 Hz, only while locked |

(Pulled from `OSC-REFERENCE.md` / `AlignManager.swift` — do not invent addresses.)

## Outputs
`out1` (CHOP) with channels `tdlidar/align/detected`, `tdlidar/align/size1`, `tdlidar/align/size2`, and `tdlidar/align/corner0` … `corner3` (each expanding to 3 channels, `1..3`).

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9102 | UDP port to listen on — Align uses its **own port**, separate from the 9000 sensor bus, so it can run alongside Sensors/Cue Deck without collision |

## Quick start (beginner)
1. In the app, open **Align**, point it at the projector target IP/port (Settings → TouchDesigner OSC — default port **9102**).
2. Slowly pan the phone across the wall/screen until it renders as a translucent blue quad.
3. Tap the quad to lock it. Four yellow corner spheres appear.
4. Drop the **Align** op in TD with **OSC Port** set to 9102. `corner0..corner3` now carry the surface's real corners — wire them into your warp/corner-pin network's 4 control points.

## Advanced patterns
- **camSchnappr / Stoner corner feed.** Export the 4 corner CHOP channels straight into a corner-pin operator's control points as a starting pose, then hand-tune from there — much faster than eyeballing all 4 corners cold.
- **Auto-scale a projection plane.** Feed `size1`/`size2` into a Geometry COMP's scale so a placeholder plane always matches the physical surface's real metres, even if you re-scan a different wall.
- **Re-lock live.** `detected` drops to 0 the moment the performer taps Reset on the phone and picks a new surface — gate downstream logic on it so a warp network doesn't keep solving against stale corners mid-reset.
- **Multiple surfaces.** Align only tracks one locked surface at a time; for multi-surface rigs run one phone per surface (each on its own OSC port) or re-lock sequentially between setup passes.

## Gotchas
- **Not a replacement for TD's own keystone tools.** This gives you a rough starting geometry from the real room; do the fine blend/warp solve in TouchDesigner as usual.
- **Corners freeze when you drag one.** Tap-hold-dragging any corner on the phone marks the surface "manually edited" and stops it re-syncing from live ARKit tracking — tap **Reset** on the phone to unlock and re-scan from scratch.
- **`corner0..3`, `size1/2` are only meaningful while `detected == 1`.** Gate downstream logic on `detected`; channels go stale (last locked values) once nothing is locked.
- **Own port, not 9000.** Align's default is **9102** — if you don't see data, check you didn't leave the op on the sensor bus's 9000.
