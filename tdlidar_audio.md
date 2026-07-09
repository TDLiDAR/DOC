---
title: "Audio"
parent: Operators
---
# Audio — `tdlidar_audio`

> The classic audio-reactive op: band levels to scale shapes, beats to trigger, FFT to instancing.

**Category:** Audio · **Tier:** Free · **Needs:** microphone access on the phone

## What it does
Turns the phone's microphone into a full music-analysis feed for TouchDesigner. You get coarse low/mid/high band levels, a 20-band FFT spectrum, transient triggers (`beat`, `onset`), and a set of pitch/timbre descriptors (pitch, note, MIDI, spectral centroid, flux, rolloff, loudness). It is everything you need to build a stage-grade audio-reactive show driven by one phone, no audio interface required.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/audio/rms` | float | mic RMS volume | audio rate |
| `/tdlidar/audio/{low,mid,high}` | float | band levels | audio rate |
| `/tdlidar/audio/fft…` | float | 20-band FFT spectrum | audio rate |
| `/tdlidar/audio/{beat,onset}` | float | transient triggers (momentary) | on event |
| `/tdlidar/audio/{pitch,note,midi,centroid,flux,rolloff,loudness}` | float | pitch / timbre channels | audio rate |

> The legacy `/tdlidar/audio/drums/{low,mid,high}` channels were removed (they never changed).

## Outputs
- `out1` (CHOP) — every audio channel above, named by address: `tdlidar/audio/low`, `…/mid`, `…/high`, the 20 `…/fft…` channels, `…/beat`, `…/onset`, `…/pitch`, `…/note`, `…/midi`, `…/centroid`, `…/flux`, `…/rolloff`, `…/loudness`, `…/rms`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable Audio in the app and grant microphone access.
2. Drop `tdlidar_audio`.
3. Play music near the phone — `out1` fills with moving channels.
4. Export `tdlidar/audio/low` onto a Geo `scale` for an instant bass-pumping object; export `…/high` for the snappy detail.

## Advanced patterns
- **Bands → scale:** Math/Range each of `low/mid/high` to a sensible scale range and drive separate objects; a Lag CHOP (fast attack, slow release) makes them breathe instead of flicker.
- **Beat → trigger:** feed `…/beat` (or `…/onset`) into a **Trigger CHOP**, then a Speed/Ramp or a Count CHOP, to fire flashes, advance presets, or kick particle bursts on each transient. These are momentary (1→0), so don't expect a held value.
- **FFT → instancing:** the 20 `…/fft` channels are a ready-made spectrum. Use a Select CHOP to grab them, a CHOP-to TOP/SOP, and drive instance height/colour per band for a classic spectrum-bar wall.
- **Timbre routing:** `centroid`/`rolloff`/`flux` describe brightness and change — map `centroid` to hue and `flux` to roughness/displacement for visuals that track tone, not just volume. `note`/`midi` let you key visuals to pitch.

## Gotchas
- `beat` and `onset` are **momentary** — they pulse to 1 for one frame and drop. Catch them with a Trigger CHOP; you can't read them as a held state.
- The FFT bands are raw and spiky; Lag/Filter or normalize (Analyze CHOP max → Math divide) before driving anything smooth.
- It's a *microphone*, so it hears the room, not a clean line feed — expect bleed, handling noise, and a non-zero RMS floor. Gate quiet input with a Limit CHOP.
- If all you need is one volume channel, use the lighter `tdlidar_mic_level` instead.
