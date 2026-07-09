---
title: "QuaternionEuler"
parent: Operators
---
# QuaternionEuler — `tdlidar_attitude`

> The cleanest way to make a 3D object copy exactly how the phone is held.

**Category:** Motion · **Tier:** Free · **Needs:** any iPhone (no LiDAR required)

## What it does
Streams the phone's absolute orientation as three Euler angles — pitch (nose up/down), roll (tilt side to side) and yaw (turn left/right) — in radians. Unlike Gyro (which gives *speed* of turn), these are the actual *angles*, so they hold steady when the phone is still. Feed them into a Geo COMP and your 3D model tracks the phone like a handle.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/motion/pitch` | float | radians | ≤50 Hz |
| `/tdlidar/motion/roll` | float | radians | ≤50 Hz |
| `/tdlidar/motion/yaw` | float | radians | ≤50 Hz |

## Outputs
`out1` (CHOP) — three channels `pitch`, `roll`, `yaw` (radians). The node tile previews `out1`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable **Quaternion / Euler** in the app's Motion sensors and press **Calibrate** while holding the phone in your neutral pose.
2. Drop the **QuaternionEuler** op — three steady lines.
3. TouchDesigner's rotate parameters expect **degrees**, but this sensor is in **radians**. Add a **Math CHOP** after `out1`, set *Multiply* to **57.2958** (= 180/π). Now the channels read in degrees.
4. Drag the three converted channels onto a **Geo COMP**'s Rotate x/y/z (or use an Export). The model now mirrors the phone.

## Advanced patterns
- **Rad→deg once, reuse everywhere:** Keep a single Math CHOP ×57.2958 as your "degrees" bus and reference it from every Geo so you never double-convert.
- **Smooth the hand:** A **Lag CHOP** (lag ~0.08) or **Filter CHOP** between the Math and the Geo removes micro-jitter for a premium, weighty feel.
- **Map an angle to anything:** A **Range CHOP** turns, say, ±90° of roll into a 0–1 crossfade for a **Cross TOP** or a parameter sweep — tilt-to-mix.
- **Axis remap / re-zero:** Swap or negate channels with a **Rename/Math CHOP** if your model's "forward" differs from the phone's; the app's Calibrate handles the live re-zero of the human-held neutral.

## Gotchas
- **Radians, not degrees** — forgetting the ×57.2958 Math CHOP is the #1 mistake (your object barely moves).
- Euler angles can **gimbal-flip** near ±90° pitch; for a tripod rig keep within ~±70°, or use the `/tdlidar/arpose/quat` quaternion (via the 6DoF pose tox) for flip-free rotation.
- pitch/roll/yaw are **zeroed by Calibrate** — your neutral is wherever the phone was when you pressed it, not magnetic north.
