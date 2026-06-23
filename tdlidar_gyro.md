---
title: "Gyro"
parent: Operators
---
# Gyro — `tdlidar_gyro`

> Spin or twist the phone and drive things by *how fast* it's turning, not where it points.

**Category:** Motion · **Tier:** Free · **Needs:** any iPhone (no LiDAR required)

## What it does
Streams the phone's rotational velocity — how quickly it's turning around each of its three axes, in radians per second. Hold the phone still and all three read ~0; flick your wrist and the matching axis spikes, then settles back to 0 the instant you stop. It measures *rate of turn*, so it's perfect for detecting flicks, twists and spins rather than absolute orientation.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/motion/rotation/x` | float | rad/s (rotationRate) | ≤50 Hz |
| `/tdlidar/motion/rotation/y` | float | rad/s (rotationRate) | ≤50 Hz |
| `/tdlidar/motion/rotation/z` | float | rad/s (rotationRate) | ≤50 Hz |

## Outputs
`out1` (CHOP) — three channels `rotx`, `roty`, `rotz` (rad/s). The node tile previews `out1`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable **Gyro** in the app's Motion sensors.
2. Drop the **Gyro** op — three flat lines near 0.
3. Twist the phone around different axes and watch each channel spike.
4. Wire one channel into a **Speed CHOP**'s input, or straight into the *rotate* speed of a spinning Geo, so the visual spins as fast as the phone does.

## Advanced patterns
- **Velocity → angle:** Feed a channel into a **Speed CHOP** to integrate rad/s into an accumulating angle — turn the phone and the visual keeps the rotation you "wound" into it.
- **Flick gesture:** **Math CHOP** *Length* of the three axes → **Trigger CHOP** (threshold ~3 rad/s, short re-trigger delay) to fire one bang per sharp flick. Add a Logic CHOP per axis to distinguish a roll-flick from a yaw-flick.
- **Spin energy:** A **Filter CHOP** on the magnitude gives a smooth "how much is it spinning" envelope for driving glow, blur or feedback amount.
- **Deadband:** Hands are never perfectly still — a **Logic CHOP** in *>* mode (or a Limit CHOP) gates out tiny <0.1 rad/s noise before it reaches a trigger.

## Gotchas
- This is **velocity**, not position — it returns to 0 when the phone stops moving, even if the phone is now facing a new way. For "which way is it pointing", use **QuaternionEuler**.
- Spikes are momentary; downstream logic that expects a held value needs a Speed/Filter/Trigger in between.
- rotationRate is **zeroed by the app's Calibrate** snapshot, so a press of Calibrate re-references the resting baseline.
