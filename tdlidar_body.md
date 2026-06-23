---
title: "Body"
parent: Operators
---
# Body — `tdlidar_body`

> Stand in front of the rear camera and your live skeleton drives points, bones and instanced geometry in TouchDesigner.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** rear camera (Vision 3D body pose runs on any modern iPhone; no LiDAR required)

## What it does
Tracks a single person standing in the rear camera's view and reconstructs a 17-joint skeleton — head, shoulders, elbows, wrists, hips, knees, ankles and the spine. The op plots those joints as points and draws the bones between them, ready to render or to drive instanced geometry. The pose shape (where each joint sits on screen) is rock-solid; the depth of each joint is an estimate, so the op leans on the accurate screen coordinates and gently fakes depth.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/body/detected` | float | 1 when a body is in frame, else 0 (skeleton goes stale at 0) | per frame |
| `/tdlidar/body/skeleton` | 51 float | 17 joints × (x,y,z), camera-relative **metres** | per frame |
| `/tdlidar/body/skeleton/img` | 34 float | 17 joints × (x,y), **normalized** image coords, top-left origin | per frame |
| `/tdlidar/body/<joint>` | x,y,z,conf | per-joint, metres + confidence | per frame |
| `/tdlidar/body/<joint>/img` | x,y | per-joint normalized image | per frame |
| `/tdlidar/body/distance` | float | subject distance, metres | per frame |

Joint order (index → name): 0 root, 1 spine, 2 neck, 3 head, 4 head_top, 5 l_shoulder, 6 l_elbow, 7 l_wrist, 8 l_hip, 9 l_knee, 10 l_ankle, 11 r_shoulder, 12 r_elbow, 13 r_wrist, 14 r_hip, 15 r_knee, 16 r_ankle.

## Outputs
- `out_pop` (POP) — 17 points carrying the reconstructed joint positions plus a bone line topology connecting them. This is the skeleton you render.
- `out1` (CHOP) — the raw incoming channels (`tdlidar/body/detected`, `…/distance`, and every per-joint channel) for triggering and mapping.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app's OSC port) |
| Max Joint Z | 0.3 | clamps how far any joint is allowed to push in/out of the screen plane — keeps the noisy depth from blowing the skeleton apart |
| Z Scale | 1.0 | overall multiplier on the (clamped) depth, 0 = perfectly flat |
| Aspect | auto | screen aspect ratio so the skeleton isn't stretched (match your camera 3:4 / 9:16) |
| Pos Scale | 1.0 | uniform size of the whole skeleton in your scene |
| Show Bones | On | draw the bone lines between joints (off = points only) |
| Flatten Z | Off | force every joint to Z=0 — the cleanest, most stable look |
| Move Mode | Centred | how the figure is anchored (centred on root vs. raw image position) |

## Quick start (beginner)
1. In the TDLiDAR app, enable **Body** and point the **rear** camera at a person standing a few metres back.
2. Drop the **Body** op into your network. When someone is in frame, `out_pop` fills with a stick figure that moves with them.
3. Wire `out_pop` into a **Geometry COMP** and render it through a **Camera** + **Render TOP** to see the skeleton on screen.
4. Add a **Line MAT** on the Geo so the bones draw as glowing strokes.

## Advanced patterns
- **Instanced geometry on joints:** feed `out_pop` (the 17 points) into a **Geometry COMP** set to *Instancing → POP*, and put a sphere SOP/Geo on each point. Now every joint is a 3D object you can scale, colour or trail.
- **Smooth the jitter:** a per-frame Vision pose dances slightly. Put a **Lag CHOP** (lag ~0.08) or **Filter CHOP** on `out1` before the Script that builds the POP, or lag the POP positions, for a calmer figure that still tracks fast moves.
- **Trigger on approach:** take `tdlidar/body/distance` from `out1` → **Logic/Trigger CHOP** to fire when the subject steps inside, say, 2 m (reveal a layer, start a clip, raise intensity).
- **Gate on presence:** `tdlidar/body/detected` is your "is anyone there" switch — multiply visuals by it, or use it on a **Switch TOP** so the scene drops to a hold state when the frame empties.

## Gotchas
- **X/Y are accurate, Z is noisy.** Don't trust raw `…/skeleton` metres for depth — that's exactly why the op rebuilds shape from `…/skeleton/img` and clamps Z. If the figure looks chaotic in depth, turn on **Flatten Z** or lower **Max Joint Z**.
- **Channels go stale, not zero, when nobody's there.** When `…/detected` reads 0 the joint channels just stop updating (they hold their last value). Always gate on `…/detected` rather than assuming the skeleton resets.
- **One body only.** This is single-subject Vision pose. For a true multi-joint metric rig use the **AR Body** op instead (ARKit, but it takes over the depth session).
- **`/tdlidar/body/…` is a shared bucket.** Hands, Pinch and Gesture all live under the same prefix — enabling them alongside Body just adds channels, it won't break the skeleton.
