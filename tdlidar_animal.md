---
title: "Animal"
parent: Operators
---
# Animal — `tdlidar_animal`

> Point the camera at a cat or dog and get its pose as live 2D points you can plot, trail or react to.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** any iPhone (rear camera); `distance` needs a depth camera

## What it does
Detects a cat or dog in the camera feed and streams the screen position of its body keypoints — nose, neck, the four paws and the tail — as normalized 2D coordinates. It's the Body op's animal cousin, but **2D only**: there's no depth per joint, so you treat the points as flat screen markers. When a depth camera is running it also reports the animal's distance. Plot the points in a Script SOP to draw the pose, trail them, or trigger off where a paw lands.

## OSC in
Each keypoint carries an `x` and a `y` (normalized image coords, **no Z**):

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/animal/nose` | x, y float | 0–1 each | camera rate |
| `/tdlidar/animal/neck` | x, y float | 0–1 each | camera rate |
| `/tdlidar/animal/frontpaw/left` | x, y float | 0–1 each | camera rate |
| `/tdlidar/animal/frontpaw/right` | x, y float | 0–1 each | camera rate |
| `/tdlidar/animal/backpaw/left` | x, y float | 0–1 each | camera rate |
| `/tdlidar/animal/backpaw/right` | x, y float | 0–1 each | camera rate |
| `/tdlidar/animal/tail/…` | x, y float | 0–1 each | camera rate |
| `/tdlidar/animal/distance` | float | metres (only when a depth camera is active) | depth rate |

> In TD each address becomes channels named by the address without the leading slash, with the two values indexed (`tdlidar/animal/nose:x`, `…:y` / `…1`, `…2`).

## Outputs
- `out1` (CHOP) — the raw keypoint channels (each x, y) plus `tdlidar/animal/distance`.
- `out_pop` / a **Script SOP** — the keypoints assembled as 2D points so you can draw the pose directly.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, enable **Animal** (Camera & Vision).
2. Point the rear camera at a cat or dog and drop the **Animal** op.
3. Watch the keypoint channels in `out1` populate (x/y per body part); the plotted points show the pose.
4. Drive a small Circle TOP from the **nose** x/y (over your camera feed) to see one keypoint tracking live.

## Advanced patterns
- **Plot the pose:** feed the keypoints into a **Script SOP** as 2D points and connect nose→neck→tail / neck→paws to draw a simple skeleton. **Keep it 2D** — there's no per-joint Z, so don't run rigid IK on it (forcing 3D solves distorts a 2D pose).
- **Trail a paw:** **Lag CHOP** then **Trail CHOP** a single paw's x/y to leave a motion ribbon, or feed it into a feedback TOP for paint-with-the-cat effects.
- **Pose triggers:** **Logic CHOP** on relative positions (e.g. a front paw above the neck Y) to fire an event when the animal raises a paw or jumps.
- **Distance reactivity:** when a depth camera is active, map `tdlidar/animal/distance` through a **Range CHOP** to scale the visual as the animal approaches.

## Gotchas
- Keypoints are **2D, normalized 0–1, no depth** — unlike the human Body op there is no metric Z. Treat them as flat screen markers and do not feed them into a 3D rig solver.
- Coordinates are normalized to the image; multiply by resolution for pixels, and invert Y (`1 - y`) if a plotted point tracks the wrong way vertically.
- Channels go **stale** when no animal is detected (they hold the last pose). Gate downstream logic on a detection signal rather than assuming the points are live.
- `distance` only exists while a **depth camera** is running; on plain RGB capture that channel won't update.
