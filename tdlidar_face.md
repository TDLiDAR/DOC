---
title: "Face"
parent: Operators
---
# Face — `tdlidar_face`

> Make a face at the front camera and drive a 3D character's expressions, eyebrows, jaw and gaze live.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** front **TrueDepth** camera (iPhone X and later)

## What it does
Uses the front TrueDepth camera and ARKit to read your face in real time and break it into 52 expression sliders (blendshapes) — each eye blink, brow raise, jaw drop, smile, cheek puff and more, every one a smooth 0–1 value. Alongside the blendshapes it reports where your head is pointed (a full transform) and where your eyes are looking (gaze). Wire it straight into a rigged face mesh to puppet a digital character, or pick out single channels to drive any parameter with an eyebrow or a jaw.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/face/*` | float | 52 ARKit blendshapes (0–1), head transform, gaze / look-at | per frame |

The `*` expands to one channel per blendshape (e.g. `…/jawOpen`, `…/browInnerUp`, `…/eyeBlinkLeft`, `…/mouthSmileLeft`) plus the head-pose and gaze channels — all numeric, so an **OSC In CHOP** reads them all.

## Outputs
- `out1` (CHOP) — every face channel: the 52 named blendshapes (0–1), the head transform, and the gaze/look-at values, named by their address.
- `out_top` (TOP) — optional readout overlay of the strongest blendshapes for quick sanity-checking on stage.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |
| Smooth | 0.0 | built-in lag on all channels (0 = raw, higher = calmer expressions) |

## Quick start (beginner)
1. In the app, enable **Face** and let the **front** camera see your face (it switches to TrueDepth automatically).
2. Drop the **Face** op. Blink, smile, raise a brow — the matching channels in `out1` move.
3. Find a single channel by name (e.g. `tdlidar/face/jawOpen`) with a **Select CHOP** and wire it to any parameter to drive it with your jaw.
4. To puppet a mesh, export the same-named blendshapes onto a face geometry (below).

## Advanced patterns
- **Drive a face mesh's blendshapes:** name your character's morph targets to match ARKit's 52 blendshape names, then map `out1` channel-by-channel onto the geometry's blend weights (a **Rename / Select CHOP** to line names up, into the SOP/COMP that holds the morphs). Now your face puppets the model 1:1.
- **Eyebrow / jaw to params:** pull `browInnerUp`, `browOuterUpLeft/Right` or `jawOpen` out with a Select CHOP, run through **Math/Range CHOP** to taste, and ride a filter cutoff, a particle rate, an exposure.
- **Gaze to aim:** take the look-at / gaze channels into a **Math CHOP** and feed a **Geometry COMP** *Look At* or a light's direction, so wherever you glance, an object or beam follows.
- **Smooth expressions:** the **Smooth** par (or an external **Lag CHOP** on `out1`) removes micro-jitter so a driven mesh doesn't quiver between frames.

## Gotchas
- **Front camera only, and exclusive of the rear depth modes.** Face takes the TrueDepth sensor, so it can't run at the same time as the rear LiDAR depth pipeline — switching to Face pauses that.
- **Blendshapes are 0–1, not symmetric.** Resting face ≈ 0 on most channels; many come in left/right pairs (`eyeBlinkLeft` / `eyeBlinkRight`) — don't assume a single channel covers both sides.
- **Lighting matters.** TrueDepth needs your face reasonably lit and roughly facing the phone; extreme angles or a turned-away head make channels drift or hold.
- All face data is numeric — read it with an **OSC In CHOP**, not a DAT.
