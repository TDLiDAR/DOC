---
title: "Front Distance"
parent: Operators
---
# Front Distance — `tdlidar_frontdist`

> Wave your hand toward the front of the phone to ride a fader — the contactless air-fader for live control.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** front TrueDepth (Face ID iPhone)

## What it does
Uses the front TrueDepth camera to measure how close the nearest thing is to the screen and gives you one clean number from 0 to 1. Move your hand (or yourself) toward the phone and the value rises; pull away and it falls. The near/far range is adjustable in the app, so you can tune how big a gesture spans the full 0–1 sweep. This is the headline "air-fader": one hand, no touch, one continuous control.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/frontdist/norm` | float | 0–1 normalized nearest distance (front TrueDepth) | depth rate |

## Outputs
`out1` (CHOP) — one channel: `tdlidar/frontdist/norm` (0–1). The node tile previews `out1`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, enable **Front Distance** and set the near/far range to a comfortable arm's reach.
2. Drop the **Front Distance** op — `out1` sits somewhere in 0–1.
3. Move your hand toward and away from the phone's screen; the value sweeps.
4. Wire `out1` straight into any 0–1 parameter — a crossfader, opacity, a Blur amount — and you've got a hands-free fader.

## Advanced patterns
- **Map with Range CHOP:** the channel is already 0–1 but rarely fills the whole range cleanly; a **Range CHOP** (set *From Range* to the min/max you actually reach, *To Range* 0–1) gives you full-throw control and lets you invert (1→0) so "closer = less".
- **Smooth the hand-shake:** a **Lag CHOP** (0.05–0.15) or Filter CHOP removes the micro-jitter of a hovering hand without adding noticeable latency to a deliberate move.
- **Zones instead of a fader:** **Logic CHOP** thresholds carve the sweep into bands (far / mid / near) to switch scenes by hand height, like a stepped selector.
- **Two-handed feel:** combine with `tdlidar_pinch` or a Touch op so distance sets coarse value and a pinch confirms/latches it.

## Gotchas
- This is **normalized 0–1**, *not* metres — if you need real distance use the rear **Back Distance** op instead.
- The full 0–1 sweep depends on the **range set in the app**; if the value barely moves, widen (or narrow) that range, or remap with a Range CHOP.
- TrueDepth has a limited near/far working distance — too close or too far reads as a clamped 0 or 1. Keep gestures inside arm's reach.
- It reports the **nearest** surface, so anything closer than your hand (a sleeve, the desk edge) hijacks the value. Keep the front field clear.
