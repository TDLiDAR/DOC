---
title: "Barometer"
parent: Operators
---
# Barometer — `tdlidar_barometer`

> Lift the phone a flight of stairs, or feel a door slam, through air-pressure changes.

**Category:** Motion · **Tier:** Free · **Needs:** any iPhone (no LiDAR required)

## What it does
Streams two slow channels from the phone's air-pressure sensor: a **relative altitude** in metres (how far up or down the phone has moved since the app started) and the **raw pressure** in kilopascals. It's sensitive enough to register a single floor of a building, an elevator ride, or the pressure pop of a closing door. Updates arrive about once a second, so treat it as a gentle, ambient input.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/motion/altitude/relative` | float | metres (relative) | ~1 Hz |
| `/tdlidar/motion/altitude/pressure` | float | kPa | ~1 Hz |

## Outputs
`out1` (CHOP) — **2 channels**: `relative` (m) and `pressure` (kPa). The node tile previews `out1`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable **Barometer** in the app's Motion sensors.
2. Drop the **Barometer** op — two near-flat lines (don't expect motion; it's a quiet sensor).
3. Carry the phone up some stairs: `relative` climbs by roughly +3 m per floor.
4. Wire `relative` into a **Range CHOP** (e.g. 0–10 m → 0–1) and drive a vertical gradient or a particle-system height — literally "the higher you carry the phone, the higher the visual rises".

## Advanced patterns
- **Slow-input smoothing:** At ~1 Hz the signal is steppy. A **Filter CHOP** (or Lag with a long lag) interpolates between samples so a height-driven parameter glides instead of jumping once a second.
- **Pressure-event trigger:** Subtract a slow Lag copy of `pressure` from the live `pressure` to isolate sudden bumps (a door slam, an HVAC kick), then **Trigger CHOP** the difference.
- **Floor detection:** Quantize `relative` with a **Math CHOP** (*Divide* by ~3, then *Floor*) into integer floor numbers and use a **Switch TOP** to change scene per floor for a multi-storey installation.
- **Combine with motion:** Pair with **Acceleration**/**Gravity** so "moving up *and* accelerating" reads as an elevator vs walking.

## Gotchas
- **Two channels, not three** — this op breaks the X/Y/Z pattern of the other motion ops. Address `relative` and `pressure` by name.
- **~1 Hz only.** It is far slower than the ≤50 Hz motion sensors; never expect responsive, frame-rate control without a Filter CHOP to interpolate.
- `relative` is **relative to app launch** (it starts near 0), not an absolute sea-level altitude. Restart the app to re-zero.
