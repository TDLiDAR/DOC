---
title: "Low Power"
parent: Operators
---
# Low Power — `tdlidar_lowpower`

> Let the phone's own Low Power Mode toggle remotely drop your show a quality tier.

**Category:** Device · **Tier:** Free · **Needs:** any iPhone (no special hardware)

## What it does
Tells you whether iOS Low Power Mode is on or off as a simple 0/1 switch. When Low Power Mode is active the phone caps performance, so this is a clean remote flag for "the device is in a constrained state — back off." It also doubles as a deliberate hardware switch: flip Low Power on the phone to signal TouchDesigner to enter an energy-saving look from across the room, no UI needed.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/device/lowpower` | float | 0/1 | on change + 2 Hz |

## Outputs
- `out1` (CHOP) — one channel: `tdlidar/device/lowpower` (0 = off, 1 = on).

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable device sensors in the app.
2. Drop `tdlidar_lowpower`.
3. On the phone, toggle Low Power Mode in Settings (or the Control Center battery button) and watch `out1` flip between 0 and 1.
4. Wire `out1` into a Switch TOP to swap between a "full" and a "lite" version of your scene.

## Advanced patterns
- **Drop a quality tier remotely:** use the 0/1 directly as a **Switch TOP/SOP index** to pick a cheaper render branch, or as a multiplier on instance count / particle birth. One physical toggle on the phone = one remote quality step.
- **Debounce the edge:** if you want it to act as a *momentary* command rather than a held state, feed it into a **Trigger CHOP** to catch only the rising edge (0→1) and pulse a one-shot action.
- **Combine with thermal:** Logic CHOP (OR) this against `tdlidar_thermal ≥ 1` so the show degrades when *either* the phone is hot *or* in Low Power Mode.
- **Latch a look:** route the rising edge into a Count CHOP or a Constant you hold, so toggling Low Power cycles through several presets instead of a single on/off.

## Gotchas
- It is a *held* state, not a pulse — it stays at 1 the whole time Low Power Mode is on. Use a Trigger CHOP if you need an edge.
- Float, but only ever 0 or 1; compare with Logic CHOP.
- Low Power Mode on the phone *also* throttles the camera/CPU itself, so expect your other TDLiDAR sensors to slow down at the same moment this flips to 1 — that's the phone, not a bug.
