---
title: "Thermal State"
parent: Operators
---
# Thermal State — `tdlidar_thermal`

> Drop TouchDesigner's render load *before* the phone overheats and silently throttles your frame rate.

**Category:** Device · **Tier:** Free · **Needs:** any iPhone (no special hardware)

## What it does
Streams the phone's thermal state as a single number from 0 (cool) to 3 (critical). iOS quietly slows the camera and CPU as the device heats up, which sandbags your sensor rates without warning. By watching this channel you can pre-emptively degrade quality in TD — fewer instances, lower-res TOPs — so the show stays smooth instead of stuttering when the phone finally throttles.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/device/thermal` | float | 0 nominal, 1 fair, 2 serious, 3 critical | on change + 2 Hz |

## Outputs
- `out1` (CHOP) — one channel: `tdlidar/device/thermal` (0–3).

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable the device sensors in the app (thermal rides the device bundle).
2. Drop `tdlidar_thermal`.
3. Watch `out1`: it sits at 0 when the phone is cool and climbs as it warms during a long session.
4. Wire it into a Text TOP so you can literally see "Thermal: 2" on a monitoring page.

## Advanced patterns
- **Auto-degrade ladder (the main use):** run `out1` through a chain of **Logic CHOP** thresholds — *≥1*, *≥2*, *≥3* — each output enabling a cheaper variant of your scene. At 1 drop bloom, at 2 halve instance count, at 3 cut to a static fallback. Switch between variants with a Switch TOP/SOP driven by those logic channels.
- **Hysteresis:** thermal can hover on a boundary and chatter. Put a Lag CHOP (slow rise, slow fall) before the Logic CHOP, or add a Trigger CHOP with separate on/off thresholds so you don't flap between quality tiers.
- **One master "quality" value:** Math/Range CHOP map 0–3 → 1.0–0.4 to get a single multiplier you can pipe into resolution, particle count, and FFT band gains at once.
- **Combine with Low Power:** OR this with `tdlidar_lowpower` so either an overheating *or* a power-saving phone drops you a tier.

## Gotchas
- The value is a float but only ever lands on 0/1/2/3 — always compare with Logic CHOP, never test for exact equality after doing arithmetic on it.
- Updates are coarse (on-change + 2 Hz). It is a *trend* signal, not a per-frame value; don't try to animate anything fast from it.
- Thermal lags reality — by the time it reads 2, the phone has *already* started slowing the camera. Build your degrade ladder to react at 1, not 3.
