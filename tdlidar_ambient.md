---
title: "Ambient Light"
parent: Operators
---
# Ambient Light — `tdlidar_ambient`

> Make your visuals breathe with the room — match brightness and colour temperature to wherever the phone is, automatically.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** any camera (back or front)

## What it does
This op gives you two friendly numbers about the room's lighting: how bright it is (in lumens) and what colour the light is (in Kelvin — low is warm/orange, high is cool/blue). It is the polished, ready-to-use version of the raw Camera Exposure feed: instead of ISO and shutter you get a brightness estimate and a colour-temperature you can map straight onto visuals.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/light/ambient_lumens` | float | room brightness, lumens | ~camera rate |
| `/tdlidar/light/kelvin` | float | colour temperature, Kelvin | ~camera rate |

## Outputs
`out1` (CHOP) with channels `tdlidar/light/ambient_lumens` and `tdlidar/light/kelvin`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, enable **Ambient Light** and start streaming.
2. Drop the **Ambient Light** op. Confirm `OSC Port` is 9000.
3. Turn the room lights up and down — `tdlidar/light/ambient_lumens` rises and falls.
4. Run it through a **Math CHOP** to remap to 0–1, then export onto a **Level TOP** brightness or a Constant TOP's value. Your output now tracks the room's brightness.

## Advanced patterns
- **Drive colour from Kelvin.** Convert `tdlidar/light/kelvin` to RGB with a **Math CHOP** (a Kelvin→RGB approximation) or a **Lookup CHOP** against a black-body gradient, then push that into a **Constant TOP** or a **Color TOP** so on-screen colour matches the real light's warmth.
- **Brightness envelope.** Remap `ambient_lumens` (Math/Range CHOP) into 0–1 and use it as a master dimmer/opacity so the piece is subtle in a bright gallery and bold in a dark room.
- **Smooth the flicker.** A **Lag CHOP** (~0.3) or **Filter CHOP** keeps brightness from jittering as the auto-exposure adjusts.
- **Threshold for night/day.** A **Logic CHOP** on a lumens threshold can switch whole scenes when someone hits the lights.

## Gotchas
- Lumens here is an **estimate derived from the camera**, not a calibrated photometer reading — good for relative changes, not absolute lux. Remap to 0–1 in your space.
- Kelvin from auto white-balance can **swing** when a strong coloured object fills the frame — Lag it before colour-mapping.
- Same data family as **Camera Exposure**; pick this op when you want lumens/Kelvin, the other when you want raw ISO/shutter/EV.
