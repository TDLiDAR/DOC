---
title: "Mic Level"
parent: Operators
---
# Mic Level — `tdlidar_mic_level`

> A one-channel mic-volume fader for when you want "loud → big" and nothing fancier.

**Category:** Audio · **Tier:** Free · **Needs:** microphone access on the phone

## What it does
Streams a single number: the loudness (RMS) of whatever the phone's microphone is hearing. It is the lightweight cousin of the full **Audio** op — no FFT, no beat detection, just one channel that rises when the room gets louder. Reach for it when you only need a volume envelope to scale or pulse something and don't want the overhead of the whole audio bundle.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/audio/rms` | float | ~0–1 (mic RMS volume) | audio rate |

## Outputs
- `out1` (CHOP) — one channel: `tdlidar/audio/rms`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable Mic Level (or Audio) in the app and grant microphone access.
2. Drop `tdlidar_mic_level`.
3. Make noise near the phone — `out1` jumps up and settles back down.
4. Export `out1` onto a Geo `scale` or a Circle TOP radius for an instant level-reactive shape.

## Advanced patterns
- **Smooth envelope:** raw RMS is jittery. A Lag CHOP with a fast attack and slow release (e.g. lag 0.02 / 0.25) turns it into a musical "follow" envelope that pumps with the room.
- **Auto-gain / normalize:** an Analyze CHOP (max over a window) feeding a Math CHOP divide keeps the reactivity consistent whether the room is quiet or loud.
- **Cheap beat:** Trigger CHOP on `out1` with a sensible threshold gives you a rough transient pulse without the full Audio op's `/beat`. For real beat/onset detection, switch to `tdlidar_audio`.
- **Map a range:** Math/Range CHOP 0–1 → your parameter's range; gate the bottom with a Limit CHOP so room hiss doesn't keep the effect twitching.

## Gotchas
- This is *only* the RMS channel. If you find yourself wanting bands, FFT, beat, or pitch, use `tdlidar_audio` instead of bolting them on here.
- RMS sits at a small non-zero floor from ambient noise — set a threshold/limit so silence reads as silence.
- It is unsmoothed and noisy; never drive a slow visible parameter directly without a Lag/Filter CHOP.
