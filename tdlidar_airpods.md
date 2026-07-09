---
title: "AirPods"
parent: Operators
---
# AirPods — `tdlidar_airpods`

> Nod, shake or turn your head while wearing AirPods to aim and trigger visuals completely hands-free.

**Category:** External · **Tier:** Free · **Needs:** motion-capable AirPods (Pro / 3rd-gen / Max) paired to the phone

## What it does
Streams the head pose from motion-sensing AirPods — pitch (nod), roll (head tilt), and yaw (look left/right) — plus the raw quaternion and motion vectors. Because the sensors are in your ears, your head becomes a controller you can use while your hands are busy on instruments or props. Look around to aim a spotlight, nod to confirm a cue, shake to reject one — all without a screen.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/airpods/pitch` | float | radians (nod) | head rate |
| `/tdlidar/airpods/roll` | float | radians (tilt) | head rate |
| `/tdlidar/airpods/yaw` | float | radians (turn) | head rate |
| `/tdlidar/airpods/pitch_deg` | float | degrees | head rate |
| `/tdlidar/airpods/roll_deg` | float | degrees | head rate |
| `/tdlidar/airpods/yaw_deg` | float | degrees | head rate |
| `/tdlidar/airpods/quat` | 4 float | orientation quaternion | head rate |
| `/tdlidar/airpods/accel` | float | head acceleration | head rate |
| `/tdlidar/airpods/gravity` | float | gravity vector | head rate |
| `/tdlidar/airpods/tilt` | float | derived tilt | head rate |

## Outputs
`out1` (CHOP) — channels named after the addresses without the leading slash: `tdlidar/airpods/pitch`, `…/roll`, `…/yaw`, the `…_deg` variants, `tdlidar/airpods/quat1…quat4`, `tdlidar/airpods/accel`, `…/gravity`, `…/tilt`. The node tile previews `out1`.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Put in motion-capable AirPods, pair them, and enable **AirPods** in the TDLiDAR app.
2. Drop the **AirPods** op. Turn your head — `yaw` (or `yaw_deg`) swings; nod and `pitch` moves.
3. Export `tdlidar/airpods/yaw_deg` and `…/pitch_deg` onto the **Rotate** of a Camera or Geo COMP to aim a virtual look with your head.
4. Add a **Lag CHOP** (lag ~0.08) first so the aim glides instead of twitching.

## Advanced patterns
- **Nod / shake triggers:** a **Slope CHOP** on `pitch` (nod) or `yaw` (shake) gives head *velocity*; a **Trigger CHOP** on a threshold detects a deliberate nod-down or quick shake — map nod = confirm/advance, shake = cancel/reset.
- **Head-look to aim:** map `yaw_deg`/`pitch_deg` through a **Math/Range CHOP** onto a light or camera angle for a hands-free spotlight; clamp the range so an over-turn doesn't whip past your stage.
- **Use the quaternion for clean rotation:** feed `quat1…quat4` into a CHOP-to-matrix (or Python `tdu.Matrix`) to drive orientation without euler gimbal-flip near ±90°.
- **Re-zero on cue:** snapshot the current yaw/pitch when you press a key and subtract it, so "straight ahead" maps to wherever the performer is actually facing at show start.

## Gotchas
- Needs **motion-capable AirPods** (Pro, 3rd-gen, Max). Standard AirPods report no head pose — the channels simply won't appear.
- The radian channels are **radians**; use the `…_deg` versions (or ×57.2958) for anything that wants degrees.
- Head orientation is relative to **how you were facing when tracking started / last re-centred** — build a re-zero step so "forward" is correct for the performer, not the AirPods' notion of north.
- Raw head pose is jittery — always Lag/Filter before driving a camera or light.
