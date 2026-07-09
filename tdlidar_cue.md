---
title: "Cue"
parent: Operators
---
# Cue — `tdlidar_cue`

> Turn the phone into a hands-free trigger pad — big edge-to-edge buttons you can rename, recolour and re-address, firing a clean momentary gate at TouchDesigner.

**Category:** Touch & Input · **Tier:** Free · **Needs:** nothing beyond the app's normal OSC connection

## What it does
The app's **Cue Deck** mode is a grid of large, MIDI-pad-style buttons on the phone screen. It starts with 4 pads filling the display edge-to-edge; each has its own name, colour and OSC address, editable on the phone. Pressing a pad sends **1** to its address and releasing sends **0** — a clean gate any OSC In CHOP reads as a momentary trigger — with a firm haptic tick on press so it feels physical, like tapping the Apple keyboard. Use it as a stagehand's cue button, a scene-advance trigger, a blackout switch, or anything else that just needs a reliable manual fire from across the room.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/cue/<n>` (default per-pad address; user-editable per pad) | float | 1 on press, 0 on release | on press/release only |

Each pad's address is set on the phone (Settings → tap a pad), so the exact addresses on your rig depend on how the pads were configured — check the Cue Deck settings screen on the device, or use an OSC In DAT/CHOP with a wildcard select if you want to catch every pad without hardcoding each one.

## Outputs
`out1` (CHOP) — one channel per configured pad's address, each a momentary 0/1 gate.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on — Cue Deck reuses the app's normal sensor OSC bus rather than a dedicated port, so it can share a connection with Sensors mode |

## Quick start (beginner)
1. In the app, open **Cue Deck**. It starts with 4 large pads; tap the **+** to add more, or the gear to rename/recolour/re-address any pad.
2. Confirm **OSC Port** is 9000 in both the app's Cue Deck settings and your OSC In CHOP.
3. Drop an **OSC In CHOP** with the pad's address (or a wildcard covering `/tdlidar/cue/*`).
4. Press a pad on the phone — the channel jumps 0→1, then back to 0 on release.

## Advanced patterns
- **Scene advance.** Wire a pad's channel into a **Trigger CHOP** or a counter, so each press steps a Switch/Select TOP to the next cue in a show file.
- **Blackout / panic button.** Dedicate one pad to a full-black override — a **Logic CHOP** on that channel gates every other output to zero regardless of what else is running.
- **Debounce is already handled on-device.** Each pad has its own re-trigger lockout (default 0.3 s) set on the phone — a fast double-press inside that window is ignored at the source, so you don't need extra debounce logic in TD.
- **Colour-match your patch.** The pad colours are cosmetic on the phone only (nothing streams over OSC for colour) — pick colours on-device that match what each pad triggers in your patch, for the operator's own sanity during a show.

## Gotchas
- **Addresses are whatever you set them to.** Unlike the fixed-address sensor ops, Cue Deck's addresses are user-editable per pad — always check the phone's settings screen (or the OSC Reference notes on your specific rig) rather than assuming the defaults.
- **Shares port 9000 with Sensors mode.** If you also run Sensors mode sensors, they multiplex onto the same OSC bus as Cue Deck — no conflict, just be aware the port isn't dedicated to cues alone.
- **Momentary, not held.** The channel is 1 only for the duration of the physical press; there's no "toggle" mode. For a persistent on/off, latch it yourself with a Logic/Counter CHOP.
- **No AR, no camera.** Cue Deck doesn't need or use the camera — it works identically on any iPhone, LiDAR or not.
