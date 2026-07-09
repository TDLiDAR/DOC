---
title: "Gravity"
parent: Operators
---
# Gravity — `tdlidar_gravity`

> Know which way is down — tilt the phone and your visuals know its orientation in the real world.

**Category:** Motion · **Tier:** Free · **Needs:** any iPhone (no LiDAR required)

## What it does
Streams a unit vector that always points toward the ground, expressed in the phone's own coordinate frame. As you tilt and rotate the device, the three numbers redistribute so their combined length stays 1 — they tell you exactly how the phone is held relative to gravity. Flat on a table reads roughly (0, 0, −1); stood upright on its edge shifts the weight onto Y.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/motion/gravity/x` | float | unit vector, points to ground | ≤50 Hz |
| `/tdlidar/motion/gravity/y` | float | unit vector, points to ground | ≤50 Hz |
| `/tdlidar/motion/gravity/z` | float | unit vector, points to ground | ≤50 Hz |

## Outputs
`out1` (CHOP) — three channels `gravx`, `gravy`, `gravz` (combined length ≈ 1). The node tile previews `out1`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable **Gravity** in the app's Motion sensors.
2. Drop the **Gravity** op. Lay the phone flat — one channel sits near ±1 and the others near 0.
3. Tilt the phone slowly and watch the weight pour between the three channels.
4. Wire a channel into a **Geo COMP**'s rotation, or into a **Ramp/Gradient** position, so your visual "falls" the way the phone tilts.

## Advanced patterns
- **Stable horizon:** Because it's already normalized and gravity-locked, gravity is far steadier than raw accel for orientation. A tiny **Lag CHOP** (0.1) removes hand tremor without lag you'd notice.
- **Tilt angle:** Feed `gravx`/`gravz` into a **Math CHOP** *atan2* (or a CHOP Expression) to get a single continuous tilt angle in radians, then ×57.2958 for degrees — great for spinning a dial or sweeping a colour LUT.
- **Pour / liquid sims:** Use the vector directly as a force direction in a particle GPU sim so "down" follows the real phone.
- **Face-up / face-down switch:** Logic CHOP on `gravz` (> 0.8 vs < −0.8) to flip between two scenes by turning the phone over — a tactile, screen-free preset switch for installations.

## Gotchas
- This is **orientation**, not movement. Shaking the phone barely changes gravity; for hits use the **Acceleration** op.
- It's a **unit vector** — magnitudes are bounded to ~±1, so don't expect big swings; remap with a **Range** par if you need a wider output.
- Gravity stays **raw** (it is *not* zeroed by the app's Calibrate button, unlike pitch/roll/yaw).
