---
title: "AR Mesh"
parent: Operators
---
# AR Mesh — `tdlidar_armesh`

> A live scan-completeness gauge — watch the LiDAR mesh of the room grow, and trigger the show once it's thick enough.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** LiDAR device (back-camera scene reconstruction)

## What it does
On a LiDAR phone, ARKit reconstructs the room as a triangle mesh while you scan. This op streams two running totals: how many mesh chunks (anchors) exist and how many triangle faces they hold. As the performer sweeps the space, both numbers climb — giving you a numeric "how much of the room is captured" signal without sending any actual geometry.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/scene/mesh_anchors` | float | number of mesh chunks | on change / low Hz |
| `/tdlidar/scene/mesh_faces` | float | total triangle faces | on change / low Hz |

## Outputs
`out1` (CHOP) with channels `tdlidar/scene/mesh_anchors` and `tdlidar/scene/mesh_faces`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. On a LiDAR device, enable **AR Mesh** in the app and start streaming. Slowly walk the phone around the space.
2. Drop the **AR Mesh** op. Confirm `OSC Port` is 9000.
3. Watch `tdlidar/scene/mesh_faces` climb — the more you scan, the higher it goes.
4. Push `mesh_faces` through a **Math/Range CHOP** into a progress bar, or **CHOP to DAT** → Text TOP to show the live face count on screen.

## Advanced patterns
- **Scan-complete trigger.** A **Logic CHOP** on `mesh_faces > threshold` fires once the room is meshed enough — auto-start the piece, or unlock a "scan finished" state.
- **Completeness meter.** **Range CHOP** `mesh_faces` to 0–1 against a target face count and drive a fill animation so performer and audience see progress.
- **Rate of capture.** A **Slope CHOP** on `mesh_faces` tells you how fast new geometry is arriving — when the slope drops near zero, the room is essentially fully scanned.
- **Combine with AR Planes.** Cross-check `tdlidar_planes` area against mesh faces for a more reliable "ready" call.

## Gotchas
- These are **counts, not geometry** — for the actual room mesh use the **Scene Build** (RoomPlan) tox or the Point Cloud tox; this op only reports totals.
- **LiDAR only.** Non-LiDAR phones won't produce scene mesh, so these channels stay flat/absent.
- **Low-rate / on-change** updates — a meter, not a smooth animation driver. Lag if you map it to anything continuous.
