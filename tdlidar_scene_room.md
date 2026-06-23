---
title: "Scene Build"
parent: Operators
---
# Scene Build — `tdlidar_scene_room`

> Walk the phone around a room, let Apple's RoomPlan scan it, and the finished room — walls, floor, doors, windows, furniture — rebuilds itself as geometry inside TouchDesigner.

**Category:** Camera & Vision · **Tier:** Pro · **Needs:** LiDAR device (iPhone/iPad Pro) for RoomPlan · the phone and TD on the same LAN

## What it does
The phone runs Apple's **RoomPlan** scan: you sweep it around a space and it builds a clean, parametric model of the room — flat walls (with holes where doors and windows are), a floor plane, and bounding boxes for the furniture. The app sends that structured geometry to TouchDesigner, where this op parses it and reconstructs the room as POP/SOP geometry you can light, texture and fly a camera through. On the phone you can review and rotate the finished model; in TD you get the same room as buildable geometry, not a photo.

## Scene stream in (not OSC sensors)
This op consumes the **RoomPlan geometry stream**, not the per-channel OSC sensor bus. The room categories arrive on the addresses listed in `OSC-REFERENCE.md`:

| address | type | meaning |
|---|---|---|
| `/tdlidar/scene/plane_count` | float | number of detected planes (walls + floor) |
| `/tdlidar/scene/plane_area_m2` | float | total detected plane area, m² (scan completeness) |
| `/tdlidar/scene/mesh_anchors` | float | LiDAR mesh anchor count |
| `/tdlidar/scene/mesh_faces` | float | LiDAR mesh face total |

The full per-element geometry (each wall's transform + size, its door/window holes, floor outline, and each object's box) rides the same scan stream and is rebuilt by the op into surfaces — the table above is the scan-status summary you can read directly.

> The scan geometry is served on its **own dedicated port** (the app defaults to a Scene Build port separate from the 9000 sensor bus). Match the op's **Scene Port** to whatever the app's Scene Build settings show.

## Outputs
- `out_pop` / `out_sop` — the reconstructed room: triangulated/extruded walls with door and window openings cut out, the floor plane, and a box per detected object.
- `out_top` (TOP) — a rendered preview of that room so the tile shows the scan taking shape.
- `out1` (CHOP) — the scan-status channels (`plane_count`, `plane_area_m2`, `mesh_anchors`, `mesh_faces`) for gating and readouts.

## Parameters
| par | default | what it does |
|---|---|---|
| Scene Port | (app's Scene Build port) | port the scan stream arrives on (match the app) |
| Wall Thickness | small | how far each wall is extruded so walls render as solids, not zero-depth planes |
| Wall / Floor / Door / Window / Opening / Object Colour | per-category | colour each room element separately |
| Out Transform | identity | position/rotate/scale the whole room in your scene |

## Quick start (beginner)
1. In the TDLiDAR app, open **Scene Build** and scan a room — sweep slowly until walls and floor fill in, then **Finish** to review the model.
2. Drop the **Scene Build** op and set its **Scene Port** to match the app.
3. As the scan streams, `out_pop`/`out_sop` build the room; the tile preview shows walls and floor appearing.
4. Wire `out_sop`/`out_pop` into a **Geometry COMP**, add a **Camera** + **Render TOP**, and fly through your scanned room.

## Advanced patterns
- **Project onto real walls:** because the geometry is metric and matches the actual room, you can align a TD projector camera to a real projector and map content onto the reconstructed walls — true geometry-based projection mapping.
- **Texture by category:** the per-category colours are a starting point — swap in materials per element (brick on walls, wood on floor, glow on objects) by splitting the POP by category before the Geo.
- **Use the openings:** doors and windows are cut as real holes — light placed "outside" a window streams through the gap, and you can spawn particles/portals at opening locations.
- **Gate on completeness:** read `plane_area_m2` / `mesh_faces` from `out1` and only trigger the show once the scan has covered enough of the room (e.g. area past a threshold) so you don't reveal a half-built space.

## Gotchas
- **RoomPlan, not a point cloud.** This is clean parametric geometry (flat walls, boxes), not the raw 50k-point capture — for dense points use the **Point Cloud** op instead.
- **Dedicated port, not 9000.** The scan rides its own Scene Build port; pointing the op at the 9000 OSC bus gets you nothing. Match the app's Scene Build setting.
- **Needs LiDAR + a finished scan.** Non-Pro/non-LiDAR phones can't run RoomPlan. The room only firms up after a complete sweep; mid-scan it's partial by design.
- **Walls are thin planes unless extruded.** Set **Wall Thickness** above zero or walls render as paper and z-fight; the floor wants its own (separate) treatment.
- **Re-scanning resets the room.** Starting a new scan replaces the geometry — build a clear/rebuild step into your patch if you switch rooms live.
