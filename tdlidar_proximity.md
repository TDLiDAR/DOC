---
title: "Proximity"
parent: Operators
---
# Proximity — `tdlidar_proximity`

> Wave your hand over the top of the phone for an instant, free, hardware blackout button — no extra gear.

**Category:** Touch & Input · **Tier:** Free · **Needs:** any iPhone (uses the built-in proximity sensor)

## What it does
Streams the phone's proximity sensor as a single 0/1: it reads 0 when nothing is near the top of the screen and 1 the moment something (your hand, a card, your face) comes close. It's the same sensor that blanks the screen during a phone call, so it costs nothing and works on every iPhone. Because it's a clean physical on/off, it makes a perfect panic/blackout trigger you can hit without looking — pass a hand over the phone and cut the visuals.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/proximity` | float | 0 far / 1 near | on change |

## Outputs
`out1` (CHOP) — one channel: `tdlidar/proximity` (0 or 1). The node tile previews `out1`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the TDLiDAR app, enable **Proximity** (Touch & Input / Sensors).
2. Drop the **Proximity** op. `out1` sits at 0.
3. Hold your hand flat over the top of the phone (where the earpiece is) — `out1` jumps to 1, lift away and it drops to 0.
4. Wire `out1` into the *index* of a **Switch TOP** (input 0 = your show, input 1 = a black TOP). Hand over = instant blackout.

## Advanced patterns
- **Blackout / panic cut:** **Switch TOP** or **Cross TOP** driven by `tdlidar/proximity` — 0 shows the program, 1 cuts to black. Put a short **Lag CHOP** (fall ~0.2 s) on it for a soft fade out/in instead of a hard cut.
- **Momentary vs latched:** for a *toggle* (cover once to black out, cover again to restore) run it through a **Trigger CHOP** → **Count CHOP** with *Limit Max 1, wrap* so each cover flips the state and you don't have to keep your hand there.
- **Cue advance:** combine with **Count CHOP** to step a Switch through scenes — each hand-pass moves to the next look (a free "next" button).
- **Logic combine:** **Logic CHOP** to AND/OR proximity with another sensor — e.g. only allow the blackout while a Remote button is also held, to avoid accidental cuts.

## Gotchas
- **Triggering proximity blanks the phone's own screen** (it's the call-sensor). That's expected — the OSC keeps flowing, but you won't see the app's preview while your hand is over it. Don't panic when the display goes dark.
- It's a coarse near/far sensor with a short range (a few cm) and no distance — it's 0 or 1, never in between. For a graded "how close", use **Front Distance** or **Back Distance** instead.
- Updates fire **on change**, so the channel can sit unchanged for a long time; that's normal, not a dropped connection.
