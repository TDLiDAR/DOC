---
title: "Screen Brightness"
parent: Operators
---
# Screen Brightness — `tdlidar_brightness`

> Steal the phone's own brightness slider and use it as a spare 0–1 fader for anything in your scene.

**Category:** Device · **Tier:** Free · **Needs:** any iPhone (no special hardware)

## What it does
Streams the phone's screen brightness as a single 0–1 value. On its face it tells you how bright the display is, but its real use is as a free hardware fader: the brightness slider in iOS Control Center is smooth, always available, and tactile, so you can repurpose it to drive any parameter in TouchDesigner without building a custom control surface.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/device/brightness` | float | 0–1 | 2 Hz |

## Outputs
- `out1` (CHOP) — one channel: `tdlidar/device/brightness` (0–1).

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable device sensors in the app.
2. Drop `tdlidar_brightness`.
3. Drag the brightness slider on the phone (Control Center) and watch `out1` glide 0→1.
4. Export `out1` onto any TOP/SOP parameter to use it as a live fader.

## Advanced patterns
- **Spare fader (the point of this op):** Math/Range CHOP remap 0–1 to whatever range your parameter needs — e.g. 0–1 → 0.5–4.0 for a feedback gain, or 0–1 → 0–360 for a hue rotate.
- **Smooth the steps:** updates arrive at 2 Hz, so the raw signal stairsteps. Put a Lag/Filter CHOP after it (lag ~0.3 s) before driving anything visible, or it will feel notchy.
- **Detents:** if you want discrete stops instead of a continuous fade, run it through a Math CHOP (`floor(value*N)/N`) to quantize to N steps you can feel on the slider.
- **Pair with another sensor:** use brightness as the *amount* and a gesture/beat as the *trigger* — brightness sets how strong an effect lands when an onset fires.

## Gotchas
- 2 Hz only — it is not a per-frame control. Always Lag/Filter it before it touches anything the audience sees.
- iOS auto-brightness will move this value on its own as room light changes; turn auto-brightness *off* on the phone if you want it to behave purely as a manual fader.
- Raising brightness drains the battery faster — watch `tdlidar_battery` if you park the slider near 1 for a long install.
