---
title: "Pinch"
parent: Operators
---
# Pinch — `tdlidar_pinch`

> Pinch thumb to index in mid-air and it becomes a smooth fader for any TouchDesigner parameter.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** rear camera (hand-derived; no LiDAR required)

## What it does
Reports a single number: how far apart your thumb tip and index tip are, measured on the first detected hand. The distance is normalized against the size of your palm, so it reads roughly the same whether your hand is near or far from the camera — fingers touching is near 0, fingers spread is the high end. It's the air-fader: one clean value you map to volume, blur, speed, anything.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/pinch` | float | thumbTip↔indexTip distance, **palm-normalized** (~0 closed, larger = open) | per frame |

## Outputs
`out1` (CHOP) — one channel, `pinch`. The node tile previews `out1`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, enable **Hands** (Pinch rides the hand pass) and show one hand to the **rear** camera.
2. Drop the **Pinch** op — internally it's just an **OSC In CHOP → Select CHOP → `out1`** picking the `tdlidar/pinch` channel.
3. Touch thumb to index, then spread them: the `pinch` value sweeps from ~0 upward.
4. Wire `out1` into the parameter you want to ride (e.g. a **Blur TOP** *Filter Size*) and pinch to drive it live.

## Advanced patterns
- **Map to a useful range:** raw pinch isn't 0–1. Run `out1` → **Math CHOP** (or **Range CHOP**) to remap your comfortable closed/open span onto exactly the target range, and clamp the ends so it can't overshoot.
- **Smooth the fader:** a **Lag CHOP** (lag ~0.1) turns a slightly jittery pinch into a buttery fader; raise the lag for slow, cinematic moves.
- **Pinch-to-click:** threshold the value with a **Logic / Trigger CHOP** so a full pinch fires a one-shot bang (advance a slide, take a snapshot) instead of acting as a continuous fader.
- **Two faders from two hands:** Pinch uses only the *first* hand. For an independent second axis, derive thumb↔index distance from the **Hand** op's `right` landmarks and run it through the same Range → Lag chain.

## Gotchas
- **It's normalized, not metric — and not 0–1.** Always remap with a Math/Range CHOP to your own observed closed/open values; don't assume a fixed scale.
- **First hand only.** If two hands are up, pinch follows whichever the app classifies first; it won't average or pick the closer one.
- **Goes stale when no hand is detected.** With no hand in frame the channel holds its last value rather than dropping to 0 — gate on the **Hand** op's `…/count` if a held fader between gestures would cause trouble.
