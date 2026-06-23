---
title: "Acceleration"
parent: Operators
---
# Acceleration — `tdlidar_accel`

> Shake, punch or throw the phone and have it fire an event in TouchDesigner.

**Category:** Motion · **Tier:** Free · **Needs:** any iPhone (no LiDAR required)

## What it does
Streams how hard and which way the phone is being moved, *after* gravity has been subtracted — so a phone sitting still reads ~0 on all three axes. The three channels are the X (left/right), Y (up/down) and Z (toward/away) push in G-force. A sharp shake or impact spikes the values; the magnitude of the spike is your trigger.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/motion/accel/x` | float | G, gravity removed (userAcceleration) | ≤50 Hz |
| `/tdlidar/motion/accel/y` | float | G, gravity removed | ≤50 Hz |
| `/tdlidar/motion/accel/z` | float | G, gravity removed | ≤50 Hz |

## Outputs
`out1` (CHOP) — three channels `accx`, `accy`, `accz`. The node tile previews `out1`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app's OSC port) |

## Quick start (beginner)
1. In the TDLiDAR app, enable **Acceleration** (Motion / Sensors).
2. Drop the **Acceleration** op into your network — you should see three flat lines near 0 in the tile.
3. Pick the phone up and wave it: the lines jump. That's live data.
4. Wire `out1` into a **Math CHOP** set to *Combine CHOPs → RMS Power* (or *Length*) to collapse the three axes into one "how much movement" channel, then feed that into anything (an opacity, a feedback amount, an LFO rate).

## Advanced patterns
- **Impact trigger:** Math CHOP (RMS of the 3 axes) → **Trigger CHOP** with a high *Trigger Threshold* (e.g. 1.5 G) and a short *Re-Trigger Delay* so one shake = one bang. Use the trigger to fire a Beat CHOP reset, swap a Switch TOP, or kick a particle burst.
- **Smooth then react:** A raw accel signal is spiky. A light **Lag CHOP** (lag 0.05) tames jitter without killing the spike; a heavier Filter CHOP gives you a slow "energy" envelope for ambient reactivity.
- **Direction-aware hits:** Keep the axes separate and run three Logic CHOPs to tell a left-swipe from an up-toss; map each to a different scene.
- **Throw detection:** During free-fall all axes drop toward 0; pair a low-magnitude window with a following spike (the catch) to detect a toss-and-catch.

## Gotchas
- This is **userAcceleration** — gravity is already removed, so resting ≈ 0 on every axis. If you want "which way is down", use the **Gravity** op instead.
- Values are momentary spikes, not held states — don't expect a level you can read between movements. Always pass through a Trigger or Filter for stable downstream logic.
- Units are **G** (1 G ≈ gravity), not m/s². A brisk shake easily exceeds 2–3 G.
