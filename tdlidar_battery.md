---
title: "Battery"
parent: Operators
---
# Battery — `tdlidar_battery`

> Wire a long-running install meter that quietly shows whether the phone is on mains or slowly running itself flat.

**Category:** Device · **Tier:** Free · **Needs:** any iPhone (no special hardware)

## What it does
Reports the phone's own battery so your TouchDesigner project can react when the device starts to drain. You get a level from empty to full plus a charging state (unknown / unplugged / charging / full). It is the simplest way to keep an unattended installation honest: dim the show, post a warning, or log when the phone has fallen off the charger.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/motion/battery/level` | float | 0–1 | on change + ~2 Hz |
| `/tdlidar/motion/battery/state` | float | 0 unknown, 1 unplugged, 2 charging, 3 full | on change |

## Outputs
- `out1` (CHOP) — two channels: `tdlidar/motion/battery/level` (0–1) and `tdlidar/motion/battery/state` (0–3).

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, the Battery sensor is part of the device/motion bundle — enable it.
2. Drop `tdlidar_battery` into your network.
3. Look at `out1`: `level` reads as a 0–1 fader (0.62 = 62%) and `state` jumps to 2 the moment you plug the phone in.
4. Wire `level` into a Text TOP (`%` formatted) for an at-a-glance battery readout on a status page.

## Advanced patterns
- **"On mains vs running down" meter:** feed `state` into a Logic CHOP set to *equals 1* (unplugged). When it goes high, you know the phone is now living on its own battery — start a countdown or fade your show toward a safe idle.
- **Low-battery trigger:** Math/Range CHOP isn't needed; just a Logic CHOP on `level` *less than 0.15* to fire a "swap the phone" alert (email DAT, OSC out to a controller, or a flashing Text TOP).
- **Smooth the meter:** the level only updates ~2 Hz, so put a Lag CHOP (lag 1–2 s) after it if you are driving an animated gauge — otherwise the needle steps.
- **Drain rate:** a Slope CHOP on `level` gives you discharge speed; a sudden steeper slope often means a heavy GPU mode kicked in on the phone.

## Gotchas
- `state` is a float even though it encodes 4 discrete states — compare with Logic CHOP, don't expect clean integers if you do math on it.
- `level` only ticks on change plus a slow 2 Hz heartbeat; do not build a smooth animation directly off it without a Lag CHOP.
- A phone reporting `state = 3` (full) while still plugged in is normal — it is not draining, so don't treat "full" as "about to die."
