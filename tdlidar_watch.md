---
title: "Apple Watch"
parent: Operators
---
# Apple Watch — `tdlidar_watch`

> Make visuals pulse to a live heartbeat, or use the Digital Crown as a fader — the wearer becomes the controller.

**Category:** External · **Tier:** Free · **Needs:** Apple Watch (watchOS 10+) running the companion TDLiDAR Watch app, paired to the phone

## What it does
Bridges sensor data from a paired Apple Watch into TouchDesigner: heart rate, the watch's own accelerometer and rotation, and the Digital Crown position. The headline is heart rate — a real, live BPM you can map straight onto colour, scale or tempo so the visuals breathe with the performer (or an audience member). The accel/rotation channels give wrist movement, and the Crown is a precise physical knob on the wearer's arm. Everything routes through the phone, so it lands on the same `/tdlidar/…` OSC stream as every other sensor.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/motion/watch/*` | float | heart rate, accel, rotation, Digital Crown | watch rate |

> The watch publishes a bundle under `/tdlidar/motion/watch/…` (e.g. heart rate, accel x/y/z, rotation, crown). The op grabs `/tdlidar/motion/watch/*` generically; check `out1`'s channel list to see exactly what the current Watch app emits.

## Outputs
`out1` (CHOP) — every `/tdlidar/motion/watch/*` channel, named after its address without the leading slash (e.g. `tdlidar/motion/watch/…`). The node tile previews `out1`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Install the **TDLiDAR Watch app** on a paired Apple Watch, open it, and enable the Watch bridge in the iPhone app.
2. Drop the **Apple Watch** op. Within a few seconds the heart-rate channel under `out1` settles on a BPM.
3. Pipe the heart-rate channel through a **Math/Range CHOP** (e.g. 50–160 BPM → 0–1) and export that onto an emission rate, a hue, or a pulse scale.
4. Twist the **Digital Crown** and watch its channel sweep — export it as a fader onto any parameter.

## Advanced patterns
- **Heartbeat pulse:** range-map BPM to a beat *period*, drive an **LFO CHOP** or **Beat CHOP** so the scene actually throbs once per beat — far more visceral than mapping BPM to a static value. A slow **Lag CHOP** keeps the BPM number from jumping between readings.
- **Crown as a precision fader:** the Crown is smooth and absolute — export it straight to a parameter, or **Slope CHOP** it for a relative jog wheel. Pair with **Range CHOP** to scope it to a useful span.
- **Wrist gestures:** **Slope/Trigger CHOP** on the watch accel detects a flick or a raise-to-trigger, freeing a hand that's holding an instrument.
- **Biofeedback mapping:** combine heart rate with the phone's **Audio** RMS so the show responds to *both* the music and the performer's exertion.

## Gotchas
- Requires the **companion Watch app actively running** — without it the `/tdlidar/motion/watch/*` channels never arrive. A backgrounded or sleeping watch app stops the stream.
- Heart rate updates **slowly** (every few seconds, not per-frame) and can briefly read 0 while the optical sensor re-locks — Lag it and gate out the 0s with a Logic CHOP so a dropout doesn't black out your mapping.
- Watch → phone → TD adds a little latency; treat watch channels as *expressive trend* signals, not frame-tight triggers.
- These channels live under the **Motion** bundle (`/tdlidar/motion/watch/…`), not a `/tdlidar/watch/…` path — match the address exactly.
