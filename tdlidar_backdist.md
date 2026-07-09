---
title: "Back Distance"
parent: Operators
---
# Back Distance — `tdlidar_backdist`

> Measure the real-world distance to whatever the on-screen cross is pointing at — drive proximity zones and distance-reactive visuals up to ~20 m.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** rear LiDAR (iPhone Pro)

## What it does
Uses the rear LiDAR scanner to report the true distance, in metres, to the spot under a movable cross-hair you aim by tapping the live preview in the app. Unlike the front fader, this is **metric** — point it at a wall, a performer or a prop and read how far away it is, out to roughly 20 metres. Tap a new spot any time to re-aim. You also get a normalized 0–1 version for quick mapping.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/backdist/meters` | float | distance in metres (up to ~20 m) | LiDAR rate |
| `/tdlidar/backdist/norm` | float | same distance, normalized 0–1 | LiDAR rate |

## Outputs
`out1` (CHOP) — two channels: `tdlidar/backdist/meters` and `tdlidar/backdist/norm`. The node tile previews `out1`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, enable **Back Distance**, then **tap the live preview** to drop the cross on the surface you want to measure.
2. Drop the **Back Distance** op — `meters` shows the real distance to that spot.
3. Move the phone (or the subject) closer/further and watch `meters` change.
4. Wire `meters` into a Text TOP to read it live, or `norm` straight into a 0–1 parameter for a distance-reactive effect.

## Advanced patterns
- **Proximity zones (the main use):** run `meters` through stacked **Logic CHOP** thresholds — *< 1 m*, *1–3 m*, *> 3 m* — each enabling a different scene, like an invisible tripwire. A **Lag CHOP** before the Logic prevents flapping at the boundaries.
- **Distance-to-wall reactivity:** map `norm` (or `meters` via a **Range CHOP**) into blur radius, audio reverb size, or particle spread so the visual "breathes" as the phone approaches a surface.
- **Hold on dropout:** LiDAR returns can briefly fail on dark/shiny/edge surfaces; **Hold CHOP** (or Lag) `meters` so a momentary 0 doesn't snap your mapping.
- **Combine with motion:** AND a proximity zone with `tdlidar_accel` so an event only fires when the phone is both *close* and *moving* — e.g. a deliberate "push toward the wall".

## Gotchas
- `meters` is **real metric distance**, `norm` is 0–1 — pick the one your downstream expects; don't feed raw metres into a 0–1 parameter.
- It measures **only the spot under the cross**. Re-tap in the app to aim; the channel doesn't scan the whole frame.
- Range tops out around **20 m**, and very dark, glossy or grazing-angle surfaces can drop the return (reads near 0). Gate or hold to ride through gaps.
- This is the **rear LiDAR** path — needs a Pro iPhone. For a touch-free fader on any Face-ID phone use `tdlidar_frontdist` instead.
