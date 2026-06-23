---
title: "Apple Pencil"
parent: Operators
---
# Apple Pencil — `tdlidar_pencil`

> Brush a TouchDesigner parameter with real pen pressure and tilt — draw weight and angle straight into your visuals.

**Category:** Touch & Input · **Tier:** Free · **Needs:** iPad + Apple Pencil

## What it does
Streams the Apple Pencil's position on the iPad screen plus its pressure, tilt and the direction it's pointing, and fires a flag when you double-tap the barrel. Pressure makes a natural fader (press harder = more), tilt gives you a second analogue axis from the same hand, and azimuth is the compass direction the pen leans. It's the most expressive single input in the family because one stroke carries position, weight and angle at once.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/pencil/x` | float | 0–1 normalized screen X | pen rate |
| `/tdlidar/pencil/y` | float | 0–1 normalized screen Y | pen rate |
| `/tdlidar/pencil/pressure` | float | 0–1 press force | pen rate |
| `/tdlidar/pencil/tilt` | float | pen tilt angle | pen rate |
| `/tdlidar/pencil/azimuth` | float | lean direction (radians) | pen rate |
| `/tdlidar/pencil/barreltap` | float | double-tap flag (momentary) | on tap |

## Outputs
`out1` (CHOP) — channels named after the addresses without the leading slash: `tdlidar/pencil/x`, `…/y`, `…/pressure`, `…/tilt`, `…/azimuth`, `…/barreltap`. The op grabs `/tdlidar/pencil/*` generically, so whichever of these the app emits will appear.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. On an iPad with a paired Pencil, enable **Apple Pencil** in the TDLiDAR app.
2. Drop the **Apple Pencil** op. Touch the Pencil to the screen and watch `out1` light up.
3. Press harder and softer — `tdlidar/pencil/pressure` swings 0→1. Export it onto a Blur size, a line width, or an emission rate for an instant pressure brush.
4. Add `tdlidar/pencil/x` / `…/y` onto a Geo COMP Translate to position the brush as you draw.

## Advanced patterns
- **Two-axis brush:** map `pressure` to one parameter and `tilt` to another (e.g. pressure → opacity, tilt → hue) so a single stroke paints with weight *and* angle.
- **Direction from azimuth:** `azimuth` is radians — a **Math CHOP** ×57.2958 gives degrees you can feed into a rotate, so the pen's lean steers a streak or comb direction.
- **Barrel-tap as a mode switch:** **Trigger CHOP** on `barreltap` → **Count CHOP** to cycle a Switch TOP between brush presets, hands stay on the pen.
- **Stroke smoothing:** a **Filter CHOP** on X/Y gives clean vector strokes; pair with a **Slope CHOP** to derive stroke speed for speed-reactive thickness.

## Gotchas
- iPad + Pencil only — on an iPhone these channels never appear.
- App support for Pencil may be **partial**; the tox deliberately grabs `/tdlidar/pencil/*` generically, so you'll see exactly the channels the app currently emits and no error for the missing ones. Check `out1`'s channel list to confirm what's live.
- `barreltap` is **momentary** (a brief 1 then back to 0) — always route it through a Trigger/Count, don't read it as a held state.
- X/Y are **normalized 0–1**, not points; range-map before using as pixel/metric coordinates.
