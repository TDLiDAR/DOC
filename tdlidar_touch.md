---
title: "Touch"
parent: Operators
---
# Touch — `tdlidar_touch`

> Turn the phone's own screen into an XY pad and pressure fader you drag with a finger while it streams to TouchDesigner.

**Category:** Touch & Input · **Tier:** Free · **Needs:** any iPhone/iPad (no special hardware)

## What it does
Streams where your finger is on the screen and how hard it's pressing. The X and Y channels are the touch position (normalized 0–1), `radius` is the contact size, and `force` is the press pressure on devices that report it. Each channel can be toggled on its own in the app, so you can stream just X/Y as a trackpad, or add force for a pressure dimension. While a finger is down the values track it live; lift off and they hold their last value.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/touch/x` | float | 0–1 normalized screen X | touch rate |
| `/tdlidar/touch/y` | float | 0–1 normalized screen Y | touch rate |
| `/tdlidar/touch/radius` | float | contact size | touch rate |
| `/tdlidar/touch/force` | float | press pressure (0 = none) | touch rate |

(Each channel is an independent per-channel toggle in the app — disabled channels simply don't stream.)

## Outputs
`out1` (CHOP) — up to four channels named after the addresses without the leading slash: `tdlidar/touch/x`, `…/y`, `…/radius`, `…/force`. Only the channels you enabled in the app appear. The node tile previews `out1`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the TDLiDAR app, enable **Touch** and tick the channels you want (X/Y at minimum; add Force for pressure).
2. Drop the **Touch** op. Drag your finger on the phone screen and watch `out1` move.
3. Use `tdlidar/touch/x` and `tdlidar/touch/y` as a 2D control: export them onto the **Translate** of a Geo COMP, or the centre of a Circle/Ramp TOP, to drag a shape around with your finger.
4. Add `tdlidar/touch/force` onto an opacity or a Blur size for a pressure-driven fader.

## Advanced patterns
- **XY pad → anything:** rename to friendly names with a **Rename CHOP**, then export X/Y to two parameters. A **Math/Range CHOP** remaps 0–1 to whatever scale the target wants (e.g. 0–1 → -5…5 metres).
- **Pressure as a gate:** **Logic CHOP** on `force > 0` gives you a clean "finger is down" boolean to enable a Switch or arm a Trigger — so the visual only responds while you hold.
- **Smooth the drag:** a light **Lag CHOP** (lag ~0.05) on X/Y removes the stair-step from finger sampling without adding visible lag.
- **Velocity from position:** a **Slope CHOP** on X/Y turns position into swipe speed/direction — drive a particle force or a flick gesture from how fast you drag.

## Gotchas
- Channels only exist if you enabled them in the app — a missing `force` channel means the per-channel toggle is off (or the device doesn't report 3D-touch-style force), not that the op is broken.
- Values **hold their last position on lift-off** — they don't snap to 0. If you need "finger up" you must watch `force` (or radius) crossing 0, not X/Y.
- X/Y are **normalized 0–1**, not pixels. Range-map before feeding anything that expects metres or pixel coordinates.
