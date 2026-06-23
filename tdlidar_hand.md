---
title: "Hand"
parent: Operators
---
# Hand — `tdlidar_hand`

> Raise one or both hands in front of the rear camera and have fully-rigged finger skeletons drive your scene.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** rear camera (Vision hand pose; no LiDAR required)

## What it does
Tracks up to two hands and rebuilds each as a 21-landmark skeleton — wrist, plus four joints per finger out to each fingertip. The op plots those landmarks as points and draws the finger bones, locking each bone to a fixed length so a finger never stretches or warps as the hand moves toward or away from the camera. Use it to puppet geometry, scrub parameters with a fingertip, or spawn particles from the tips.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/body/hands/count` | int | number of hands (0–2) | per frame |
| `/tdlidar/body/hands/distance` | float | hand distance, **metres** | per frame |
| `/tdlidar/body/hands/frame` | int | heartbeat — increments **every frame** so TD knows data is live even when the hand is still | per frame |
| `/tdlidar/body/hands/<side>/<landmark>` | x,y,z + rx,ry,rz | 21 landmarks × (x,y **normalized** image, z **metres**) + unit bone direction | per frame |

21 landmark order: wrist, thumb_cmc, thumb_mcp, thumb_ip, thumb_tip, index_mcp, index_pip, index_dip, index_tip, middle_mcp, middle_pip, middle_dip, middle_tip, ring_mcp, ring_pip, ring_dip, ring_tip, pinky_mcp, pinky_pip, pinky_dip, pinky_tip. `<side>` is `left` / `right`, classified by the wrist's screen position.

## Outputs
- `out_pop` (POP) — up to 2 × 21 points with the finger-bone topology, lengths locked by rigid IK. This is the hand skeleton you render or instance onto.
- `out1` (CHOP) — `tdlidar/body/hands/count`, `…/distance`, `…/frame`, and the per-landmark channels for mapping and triggers.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |
| Lock Bone Lengths | On | rigid-IK clamp so fingers keep constant length regardless of noisy Z (off = raw landmarks) |
| Z Scale | 1.0 | depth multiplier; 0 = flatten the hand into the screen plane |
| Aspect | auto | screen aspect ratio so the hand isn't stretched |
| Pos Scale | 1.0 | overall size of the hand in your scene |
| Show Bones | On | draw finger bones (off = landmark points only) |

## Quick start (beginner)
1. In the app, enable **Hands** and point the **rear** camera at your hand.
2. Drop the **Hand** op. Raise a hand — `out_pop` fills with a 21-point hand skeleton that follows your fingers.
3. Wire `out_pop` into a **Geometry COMP** → **Render TOP** with a **Line MAT** to draw the finger bones.
4. Watch `tdlidar/body/hands/count` on `out1`: 0, 1 or 2 tells you how many hands are live.

## Advanced patterns
- **Particles from fingertips:** select the five `*_tip` landmarks (thumb_tip, index_tip, …) out of `out_pop` into a **Particle GPU / source POP**, so sparks stream off each fingertip you move.
- **Per-finger curl:** for one finger, take the unit bone directions (`rx,ry,rz` of its mcp→pip→dip segments) and feed them through a **Math CHOP** dot-product to measure how bent the finger is — a clean 0→1 curl value per finger, great for triggering states.
- **Heartbeat = liveness gate:** because `…/frame` increments every frame even when the hand is dead still, compare it to its own delayed value (Lag/Logic) to know the stream is alive vs. the app paused — don't infer "no movement" as "no data".
- **Smooth:** a **Lag CHOP** on `out1` (or lag on the POP positions) softens tracking noise before driving anything visible.

## Gotchas
- **Side is by screen position, not chirality.** `left`/`right` is decided by where the wrist sits on screen, so it's stable whether you show palm, back or edge — but it is **not** anatomical handedness. Don't assume `right` = the person's right hand.
- **Z is metric but noisy; X/Y normalized are the trustworthy shape.** Rigid IK (Lock Bone Lengths) exists precisely so depth jitter doesn't warp the fingers — leave it on unless you specifically want raw landmarks.
- **Data arrives every frame, even when motionless.** The `…/frame` heartbeat means channels keep ticking; gate downstream logic on `…/count`, not on "did a value change".
- Shares the `/tdlidar/body/…` bucket with Body, Pinch and Gesture.
