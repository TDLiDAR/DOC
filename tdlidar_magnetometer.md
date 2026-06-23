---
title: "Magnetometer"
parent: Operators
---
# Magnetometer — `tdlidar_magnetometer`

> Read the raw magnetic field around the phone — wave a magnet, sense a motor, or detect a metal object passing by.

**Category:** Motion · **Tier:** Free · **Needs:** any iPhone (no LiDAR required)

## What it does
Streams the raw magnetic field measured at the phone, as three components (X, Y, Z) in microtesla. It picks up the Earth's field plus anything magnetic nearby — speakers, motors, a handheld magnet, large steel structures. It is **not** a finished compass heading; it's the unprocessed field, which makes it great for reacting to magnetic *changes* in a space rather than for "point me north".

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/motion/magnetic/x` | float | µT (raw field, not heading) | ≤50 Hz |
| `/tdlidar/motion/magnetic/y` | float | µT (raw field, not heading) | ≤50 Hz |
| `/tdlidar/motion/magnetic/z` | float | µT (raw field, not heading) | ≤50 Hz |

## Outputs
`out1` (CHOP) — three channels `magx`, `magy`, `magz` (µT). The node tile previews `out1`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable **Magnetometer** in the app's Motion sensors.
2. Drop the **Magnetometer** op — three lines hovering at the local field strength (often tens of µT).
3. Move a fridge magnet near the phone and watch a channel swing hard.
4. Wire a **Math CHOP** *Length* of the three axes into a parameter so total field strength drives a visual — proximity-to-metal as a controller.

## Advanced patterns
- **Field-change trigger:** Subtract a slow **Lag CHOP** copy of a channel from the live channel (a **Math CHOP** *Subtract*) to get only the *change*; feed that into a **Trigger CHOP** so a magnet swipe fires an event while the steady background field is ignored.
- **DIY compass-ish:** A **Math CHOP** *atan2* of `magx`/`magy` gives a rough planar heading, but expect drift and tilt error — for real orientation use the **QuaternionEuler** op instead.
- **Calibrate-by-subtract:** Sample the resting field once into a Constant CHOP and subtract it so your installation responds to *added* magnetism, not the room's baseline.
- **Multi-sensor scene:** Combine with **Proximity** or **Acceleration** so "object near + metallic" gates a specific cue.

## Gotchas
- **Not a heading.** These are raw µT components — there is no north channel here. Don't treat any one channel as a compass bearing.
- The reading drifts with the phone's own electronics and is easily swamped by nearby ferrous metal — calibrate against the resting baseline if absolute level matters.
- Magnetic stays **raw** — it is *not* zeroed by the app's Calibrate button (unlike pitch/roll/yaw and rotation).
