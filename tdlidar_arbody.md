---
title: "AR Body"
parent: Operators
---
# AR Body — `tdlidar_arbody`

> Stand in front of the rear camera and drive a true full-rig, metre-accurate 3D skeleton at 60 Hz.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** rear camera with ARKit body tracking (A12+ device; **exclusive** with the depth/LiDAR session)

## What it does
Uses ARKit's dedicated body-tracking to reconstruct a person as a full **91-joint** skeleton in real **world-space metres**, updated at 60 Hz — wrists, fingers-of-the-hand root, spine chain, feet and toes, all properly parented into a rig with a joint **hierarchy** and per-joint **orientation**. Compared to the Vision-based **Body** op this is far cleaner and genuinely 3D (depth you can trust), but it takes over the camera's AR session, so you can't run it alongside the rear depth modes. Choose this when you want a believable 3D performer; choose **Body** when you need the rear depth pipeline running too.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/arbody/skeleton…` | float | true 91-joint, **world-space metric** skeleton | 60 Hz |
| `/tdlidar/arbody/orientation` | 4 float | body orientation quaternion (x,y,z,w) | 60 Hz |
| `/tdlidar/arbody/hierarchy` | — | joint parent indices for the rig | on start / change |

## Outputs
- `out_pop` (POP) — 91 points in world metres with the rig's bone topology built from `…/hierarchy`, each joint carrying its `…/orientation` so you can drive a properly oriented rig.
- `out1` (CHOP) — the raw skeleton, orientation quaternion and hierarchy channels for mapping and constraint work.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |
| Pos Scale | 1.0 | uniform world scale of the rig in your scene |
| Show Bones | On | draw bones from the joint hierarchy (off = points only) |
| Up Axis | Y | match TD's up-axis to ARKit's so the figure stands upright |

## Quick start (beginner)
1. In the app, enable **AR Body** and aim the **rear** camera at a person, full body in frame, a few metres back.
2. Drop the **AR Body** op. A clean 91-point skeleton appears in `out_pop`, standing in real-world scale.
3. Wire `out_pop` into a **Geometry COMP** → **Render TOP** with a **Line MAT** to draw the rig.
4. Walk around: because positions are true metres, the figure moves through your scene at life size.

## Advanced patterns
- **Drive an instanced or skinned character:** the `…/hierarchy` parent indices plus per-joint `…/orientation` give you a real rig — map them onto a character's bones (joint-by-joint constraint, or instance geometry onto each POP point oriented by its quaternion) for a retargeted performer.
- **World-anchored visuals:** since joints are in world metres, attach effects to a wrist or foot and they stay correctly scaled and placed as the subject moves toward/away — no per-frame rescaling like the monocular Body op needs.
- **Light smoothing only:** ARKit body is already stable at 60 Hz, so a tiny **Lag CHOP** (lag ~0.03) is usually all you want — heavier filtering just adds latency you don't need here.
- **Orientation-driven motion:** use `…/orientation` (the body quaternion) on a **Math CHOP** to know which way the person faces and turn a whole scene, camera or instance field to follow them.

## Gotchas
- **Exclusive with the depth session.** AR Body owns the rear AR camera; you can't run the LiDAR/rear depth modes at the same time. Switch deliberately.
- **It's metric world-space, not normalized.** Don't expect 0–1 image coords like the Vision **Body** op — these are real metres, so set **Pos Scale** / **Up Axis** to seat the figure correctly in your scene.
- **Needs the full body and decent space.** ARKit body tracking wants a clear view of a whole person; partial or very close subjects drop tracking.
- **Hierarchy arrives once.** `…/hierarchy` defines the rig structure and only resends on start/change — build your bone topology from it rather than expecting it every frame.
