---
title: "6DoF Device Pose"
parent: Operators
---
# 6DoF Device Pose — `tdlidar_devicepose`

> Mount the phone on a tripod (or hold it), walk it through the room, and have a TouchDesigner camera fly exactly the same path — the heart of projection-mapping and AR-aligned visuals.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** ARKit world tracking (LiDAR device recommended for solid tracking)

## What it does
This op streams the phone's full position and orientation in real space — where it is (in metres) and which way it's pointing. ARKit builds a world coordinate frame the moment tracking starts, then reports the camera's pose against it every frame. Wire that pose onto a Camera COMP and your virtual camera matches the real device, frame-for-frame. It is the camera-driver of the whole TDLiDAR family.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/arpose/tx` | float | metres (world X) | ~60 Hz |
| `/tdlidar/arpose/ty` | float | metres (world Y) | ~60 Hz |
| `/tdlidar/arpose/tz` | float | metres (world Z) | ~60 Hz |
| `/tdlidar/arpose/quat` | 4 float | orientation quaternion (x,y,z,w) | ~60 Hz |
| `/tdlidar/arpose/euler/pitch` | float | radians | ~60 Hz |
| `/tdlidar/arpose/euler/roll` | float | radians | ~60 Hz |
| `/tdlidar/arpose/euler/yaw` | float | radians | ~60 Hz |
| `/tdlidar/arpose/tracking` | int | 0 not available, 1 limited, 2 normal | on change |

## Outputs
`out1` (CHOP) carries the pose channels, named after the addresses without the leading slash:
`tdlidar/arpose/tx`, `…/ty`, `…/tz` (translation), `tdlidar/arpose/quat1…quat4` (quaternion), `tdlidar/arpose/euler/pitch`, `…/roll`, `…/yaw`, and `tdlidar/arpose/tracking`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the TDLiDAR app, enable **6DoF Device Pose** and start streaming.
2. Drop the **6DoF Device Pose** op into your network. Confirm `OSC Port` matches the app (9000).
3. Watch `out1` — `tx/ty/tz` should change as you move the phone, the euler channels as you tilt it.
4. Drop a **Camera COMP**. Export `tdlidar/arpose/tx → tx`, `ty → ty`, `tz → tz` onto its **Translate**, and `euler/pitch → rx`, `euler/roll → ry`, `euler/yaw → rz` onto its **Rotate**. Add a Geo/Render and your scene now moves with the phone.

## Advanced patterns
- **Use the quaternion, not euler, for clean rotation.** Euler channels gimbal-flip near ±90°. Feed `quat1…quat4` into the Camera COMP via a small CHOP-to-matrix setup (or a Python `tdu.Matrix` from the quat) to avoid the snap.
- **Align a TD scene to the real room.** Capture the pose at a known physical spot (e.g. phone resting on a marked floor cross), store those values, and offset all incoming pose by that snapshot so TD world-origin sits where you want it in the room.
- **Smooth the hand-shake.** Run `out1` through a **Lag CHOP** (lag ~0.05–0.12) or **Filter CHOP** before exporting to the camera — kills micro-jitter without adding visible latency.
- **Gate on tracking quality.** Use a **Logic CHOP** on `tdlidar/arpose/tracking >= 2` and only update / hold the camera when tracking is normal, so a "limited" relocalisation glitch doesn't throw your camera across the room.

## Gotchas
- `tracking` is an **int state**, not a quality bar — 0/1 mean ARKit doesn't trust the pose; only 2 is solid. Always gate on it for a show.
- Pose is relative to **wherever tracking started**, not a fixed room origin. Re-launching tracking resets the world frame; build a re-zero step into your patch.
- Euler is **radians**, not degrees — multiply by `57.2958` (or use a **Math CHOP**) before feeding any op that expects degrees.
- This is the one sensor you almost never want raw on a camera; without a Lag/Filter the projection visibly trembles.
