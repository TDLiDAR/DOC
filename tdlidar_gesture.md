---
title: "Gesture"
parent: Operators
---
# Gesture — `tdlidar_gesture`

> Hold up an open hand, a fist, a peace sign or a point and snap between presets, scenes or states.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** rear camera (hand-derived; no LiDAR required)

## What it does
Recognizes four static hand shapes — **open** hand, **fist**, **peace** sign and **point** — and reports each as an on/off flag. Show the shape to the rear camera and its channel reads 1; drop the shape and it reads 0. These are discrete switches: perfect for jumping between presets, arming a mode, or flipping a layer without touching anything.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/gesture/open` | float | 0/1 | per frame |
| `/tdlidar/gesture/fist` | float | 0/1 | per frame |
| `/tdlidar/gesture/peace` | float | 0/1 | per frame |
| `/tdlidar/gesture/point` | float | 0/1 | per frame |

The four are one-hot-ish — normally only one reads 1 at a time, briefly none during a transition.

## Outputs
`out1` (CHOP) — four channels: `tdlidar/gesture/open`, `…/fist`, `…/peace`, `…/point`. The node tile previews `out1`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, enable **Hands** and show one hand to the **rear** camera.
2. Drop the **Gesture** op — internally an **OSC In CHOP → Select CHOP → `out1`** carrying the four gesture channels.
3. Make a fist: the `fist` channel jumps to 1, the rest read 0. Open your hand and `open` takes over.
4. Wire one channel (say `fist`) into a **Switch TOP** index or an op's *Activate* toggle to flip a state with that shape.

## Advanced patterns
- **Preset selector:** the four channels are a ready-made one-hot. Use a **Select CHOP** ordering them and an **Index/Trigger** to convert "which gesture" into a single 0–3 index that drives a **Switch TOP** between four looks.
- **Clean one-shots:** feed a channel through a **Trigger CHOP** so holding a fist fires exactly one bang on entry (advance a slide, reset a sim) instead of a held 1.
- **De-bounce flicker:** at gesture boundaries the flags can flicker. A short **Lag CHOP** + threshold, or a Trigger with a small *Re-Trigger Delay*, stops a wobbly transition from double-firing.
- **Combine with Pinch:** use **Gesture** to pick *which* parameter is live and **Pinch** as the continuous fader for it — gesture selects, pinch adjusts.

## Gotchas
- **These are held states, not momentary pulses.** A channel stays at 1 the whole time you hold the shape. If you want a single event, pass it through a Trigger CHOP rather than reading the raw 1.
- **Transitions briefly read all-zero.** Moving from one shape to another can leave a frame or two where nothing is 1 — don't treat "no gesture" as its own command without a small debounce.
- **First hand only**, same as Pinch. With two hands up it follows whichever the app classifies first.
- Channels go **stale** (hold last value) when no hand is detected; gate on the **Hand** op's `…/count` if that matters.
