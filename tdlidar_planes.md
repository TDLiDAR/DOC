---
title: "AR Planes"
parent: Operators
---
# AR Planes — `tdlidar_planes`

> A live "how much of the room have I mapped?" meter — fire a cue the moment enough floor or wall has been detected.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** ARKit (LiDAR device recommended for fast plane detection)

## What it does
As the phone looks around, ARKit finds flat surfaces — floors, walls, tables — and grows them into planes. This op streams two running totals: how many planes have been found, and their combined area in square metres. Together they make a simple progress signal: the more the performer scans, the higher the numbers climb. Use it to know when the room is "mapped enough" to start.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/scene/plane_count` | float | number of detected planes | on change / low Hz |
| `/tdlidar/scene/plane_area_m2` | float | total plane area, square metres | on change / low Hz |

## Outputs
`out1` (CHOP) with channels `tdlidar/scene/plane_count` and `tdlidar/scene/plane_area_m2`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, enable **AR Planes** and start streaming. Slowly pan the phone across the floor/walls.
2. Drop the **AR Planes** op. Confirm `OSC Port` is 9000.
3. Watch `tdlidar/scene/plane_area_m2` climb as more surface is detected.
4. Wire `plane_area_m2` through a **Math/Range CHOP** (e.g. 0→20 m² mapped to 0→1) into a progress bar TOP, or onto a Text TOP via a **CHOP to DAT** to read the live count.

## Advanced patterns
- **"Room ready" trigger.** A **Logic CHOP** (or **Trigger CHOP**) on `plane_area_m2 > threshold` fires a one-shot pulse when enough surface exists — use it to auto-advance from a "scanning…" screen to the live show.
- **Progress meter.** **Range CHOP** the area into 0–1 and drive a radial/linear progress visual so the audience sees the room filling in.
- **Stabilise the count.** `plane_count` can wobble as ARKit merges/splits planes; a **Lag CHOP** or a small `>=` Logic with hysteresis stops a flickering trigger.
- **Combine with AR Mesh.** Pair plane area with `tdlidar_armesh` face counts for a richer "scan completeness" estimate than either alone.

## Gotchas
- These are **cumulative totals**, not per-surface data — you get how much, not which planes or where. For geometry, that's the Mesh / Scene Build path.
- `plane_count` can **go down** when ARKit merges two planes into one; don't assume it only rises. Threshold on *area* for stability.
- Updates are **low-rate / on-change**, so don't expect smooth 60 Hz motion — treat it as a slow meter, not an animation source.
