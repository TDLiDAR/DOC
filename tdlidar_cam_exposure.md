---
title: "Camera Exposure"
parent: Operators
---
# Camera Exposure — `tdlidar_cam_exposure`

> Let the room's real lighting drive your visuals — brighter room, brighter output; warmer light, warmer grade — without any extra hardware.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** any camera (back or front)

## What it does
iOS has no public lux meter, so this op streams the camera's auto-exposure settings as a stand-in light sensor. ISO, shutter time and EV offset together tell you how bright the scene is (the camera works harder — higher ISO, longer shutter — in the dark). White-balance Kelvin and tint tell you the colour of the light. Use these as a cheap ambient-light and colour proxy to make TD content react to the actual room.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/cam/iso` | float | ISO sensitivity (e.g. 30–3000+) | ~camera rate |
| `/tdlidar/cam/shutter_sec` | float | shutter time, seconds | ~camera rate |
| `/tdlidar/cam/ev_offset` | float | exposure bias (EV) | ~camera rate |
| `/tdlidar/cam/focus` | float | 0–1 (lens position) | ~camera rate |
| `/tdlidar/cam/wb_kelvin` | float | white balance, Kelvin | ~camera rate |
| `/tdlidar/cam/wb_tint` | float | white balance tint | ~camera rate |

## Outputs
`out1` (CHOP) with channels `tdlidar/cam/iso`, `tdlidar/cam/shutter_sec`, `tdlidar/cam/ev_offset`, `tdlidar/cam/focus`, `tdlidar/cam/wb_kelvin`, `tdlidar/cam/wb_tint`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, enable **Camera Exposure** and start streaming (any camera works).
2. Drop the **Camera Exposure** op. Check `OSC Port` is 9000.
3. Cover the lens or dim the lights — `tdlidar/cam/iso` should climb and `tdlidar/cam/shutter_sec` lengthen as the scene darkens.
4. Export `tdlidar/cam/iso` (after a Math CHOP to remap it to 0–1) onto a **Level TOP** brightness, or onto a Constant's opacity — your visual now dims and brightens with the room.

## Advanced patterns
- **Brightness-reactive output.** ISO rises in the dark, so map it *inverted*: a **Math CHOP** (range from, say, 100→3000 mapped to 1→0) gives a "room brightness" 0–1 you can drive anything with. Combine with `shutter_sec` for a steadier estimate.
- **Colour-grade by white balance.** Convert `wb_kelvin` to an RGB tint with a **Lookup CHOP** or a small Kelvin→RGB Math expression, then push it into a **Level TOP** or a Constant used as a tint — your visuals follow the room's warmth.
- **Smooth before you map.** Exposure hunts in steps; a **Lag CHOP** (~0.2–0.5) or **Filter CHOP** stops the brightness pumping when someone walks past the lens.
- **Trigger on a lighting cue.** A **Trigger CHOP** on a sudden ISO/EV jump (lights going out) can fire a scene change.

## Gotchas
- This is a **proxy, not a lux meter** — absolute values depend on the device, lens and even the auto-exposure target. Calibrate by remapping to 0–1 in *your* room; don't treat ISO as physical units.
- ISO and brightness are **inverse**: high ISO = dark room. Remember to invert when you want "brightness".
- Auto-exposure **hunts and steps** rather than gliding — always Lag/Filter before driving anything visible.
- Pair with **Ambient Light** (`tdlidar_ambient`) when you want a friendlier lumens/Kelvin pair instead of raw camera internals.
