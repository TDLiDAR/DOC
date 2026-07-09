---
title: "Rectangle"
parent: Operators
---
# Rectangle — `tdlidar_rectangle`

> A live size-meter: a rectangle that grows and shrinks with **any** TDLiDAR channel you point it at — the quickest way to *see* a sensor working, and a building block to copy.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** nothing special — any TDLiDAR sensor that's streaming

## What it does
This is a demo / meter op. It listens to **one** OSC channel of your choosing and makes a square's size follow that value: bigger value, bigger rectangle. It draws the result two ways — a **Rectangle TOP** (`out_top`) you can see immediately, and a **Rectangle POP** (`out_pop`) you can drop into a 3D scene. Point it at brightness, mic level, a pinch, body distance — anything — and you get an instant visual readout. It's also the simplest worked example of "OSC channel in → geometry out" to copy when building your own ops.

## OSC in
You choose the channel. The default is the phone's screen-brightness fader:

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/device/brightness` | float | 0–1 | 2 Hz |

Set the **Size Channel** parameter to any address from `OSC-REFERENCE.md` (e.g. `tdlidar/audio/rms`, `tdlidar/pinch`, `tdlidar/body/distance`). The op reads that one channel off the OSC In CHOP and uses its value to drive size.

## How size is computed
`size = Base Size + (value × Scale)` — applied uniformly so the shape stays **square**. With the defaults (`Base 0.2`, `Scale 0.8`) a 0–1 channel sweeps the rectangle from 0.2 up to 1.0.

## Outputs
- `out_top` (TOP) — a Rectangle TOP whose width/height track the channel (the at-a-glance meter).
- `out_pop` (POP) — a Rectangle POP at the same size, for use in a 3D/Geometry scene.
- `out1` (CHOP) — the chosen channel's raw value plus the computed size, for chaining.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |
| Size Channel | `tdlidar/device/brightness` | which OSC channel drives the size |
| Base Size | 0.2 | the rectangle's size when the channel reads 0 |
| Scale | 0.8 | how much the channel value adds to the size (`size = Base + value×Scale`) |

## Quick start (beginner)
1. Enable any sensor in the app (the default uses **device sensors** → brightness).
2. Drop the **Rectangle** op. Confirm **OSC Port** is 9000.
3. Drag the phone's brightness slider — `out_top` shows a square growing and shrinking with it.
4. Change **Size Channel** to, say, `tdlidar/audio/rms`, enable Mic Level in the app, and the square now pulses with sound.

## Advanced patterns
- **Smooth it:** at 2 Hz (brightness) or with noisy audio the size jumps. Put a **Lag/Filter CHOP** on the chosen channel before it drives size (lag ~0.2–0.3 s) for a calmer meter.
- **Remap the range:** if your channel isn't 0–1 (e.g. metric `body/distance`), use **Base Size** + **Scale** — or a **Math/Range CHOP** ahead of it — to fit its real min/max into a visible size band.
- **Stack meters:** drop several copies, each on a different **Size Channel**, lay their `out_top`s side by side, and you've built a quick multi-channel dashboard to confirm what's actually streaming.
- **Copy it as a template:** the internal chain (OSC In CHOP → pick channel → `Base + value×Scale` → Rectangle TOP/POP) is the minimal pattern for turning a sensor into geometry — clone it as the starting point for your own visualiser ops.

## Gotchas
- **One channel only.** It reads a single address; multi-value messages (quaternions, skeletons) won't map sensibly — point it at a scalar.
- **Square, uniform size.** Width and height move together by design; it's a meter, not a free aspect-ratio rectangle.
- **Pick a channel that's actually streaming.** If the chosen sensor is off in the app, the value sits at 0 (or stale) and the rectangle parks at **Base Size** — that's expected, not a bug.
- **Match the units to Base/Scale.** A channel outside 0–1 will blow the size out or barely move it until you tune **Base Size** and **Scale** (or remap upstream).
