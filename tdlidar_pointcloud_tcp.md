---
title: "Point Cloud"
parent: Operators
---
# Point Cloud — `tdlidar_pointcloud_tcp`

> Capture the room the phone is looking at as 50,000 real 3D points — exact XYZ and colour — straight into a TouchDesigner POP.

**Category:** Camera & Vision · **Tier:** Pro · **Needs:** LiDAR device (back depth) · the phone and TD on the same LAN

## What it does
Streams a live point cloud — up to 50,000 points, each with a precise position (metres) and an RGB colour — from the phone into TouchDesigner. It deliberately uses a raw **TCP** connection instead of NDI: NDI would crush the data to lossy 8-bit 4:2:2 video and mangle the coordinates, whereas TCP carries every point **bit-exact**. The points land as a POP that you drop onto a Geometry COMP to render volumetrically, or to instance a shape on every point.

## TCP in (not OSC)
This op does **not** use OSC. It opens a binary **TCP/IP DAT** in server mode and the phone connects to it.

| setting | value |
|---|---|
| Transport | TCP/IP DAT, `mode = server`, `bytes = on`, `format = all` |
| Port | **9002** (binary point stream — separate from the 9000 OSC bus) |
| Wire format | little-endian frames: `uint32 payloadLen · "TDP1" · uint32 frameId · uint32 count · count × { float32 x,y,z ; uint8 r,g,b,a }` |

Each frame is length-prefixed and tagged `TDP1`, so the receiver can resync if a packet splits across reads.

## The pipeline
The op is a small chain, not a single node:

1. **TCP/IP DAT** (`tcp_pc_in`) — receives the byte stream; a callback accumulates bytes and splits out whole `TDP1` frames.
2. **Parser** — numpy unpacks each frame into `x y z` float arrays + `r g b` (0–1) and stashes them in op storage.
3. **Reshape Script CHOP** (`tcp_reshape`) — emits the latest frame as channels `tx ty tz r g b`, N samples (one sample per point).
4. **CHOP To POP** — set `surftype = points`, `specifypos = off`, `chanssel = precisenames` so `tx/ty/tz → P` and `r/g/b → Color` map automatically.
5. **Geometry COMP** (`geo_pc`) — renders the POP.

## Outputs
- `out_pop` (POP) — N points carrying position (`P`) and colour (`Color`). This is the cloud you render or instance from.
- `out1` (CHOP) — the reshaped per-point channels (`tx ty tz r g b`) if you'd rather drive something CHOP-side.

## Parameters
| par | default | what it does |
|---|---|---|
| TCP Port | 9002 | port the DAT serves on (match the app's point-cloud port) |
| Point Scale | 1.0 | uniform scale of the cloud in your scene |
| Apply Colour | On | map the streamed RGB onto the POP's Color (off = render a flat/shaded cloud) |

## Quick start (beginner)
1. In the TDLiDAR app, open the **Point Cloud** mode and enable TCP streaming (point it at the phone's IP — TD is the server).
2. Drop the **Point Cloud** op. Confirm the **TCP Port** is 9002 and the DAT shows a connected peer.
3. `out_pop` fills with a coloured cloud of the room. Drop a **Geometry COMP**, point it at `out_pop`, and render through a **Camera** + **Render TOP**.
4. Orbit the TD camera — you're now looking *around* a live 3D capture of the real space.

## Advanced patterns
- **Instance geometry per point:** set the Geometry COMP to *Instancing → POP* and put a small sphere/cube Geo on it — every one of the 50k points becomes a lit 3D object you can scale and trail.
- **Colour by depth, not camera:** ignore the streamed RGB (turn **Apply Colour** off) and recolour with a ramp keyed off the point's Z in a POP/GLSL pass — gives a clean depth-gradient look that reads better than raw photo colour.
- **Freeze and sculpt:** a Cache/Cache Select on the POP lets you snapshot one frame and keep manipulating it after the performer moves on — useful for held tableaux.
- **Thin it for performance:** 50k points × instanced geometry is heavy. A POP filter (random delete to ~10–20k) keeps frame rate up for live work; render the full cloud only for stills/recording.

## Gotchas
- **TCP, port 9002 — not the OSC 9000 bus.** Don't try to read this with an OSC In CHOP; it's binary video-rate data on its own socket.
- **TD is the server, the phone connects.** If nothing arrives, check the phone is pointed at TD's IP and that port 9002 is open through the firewall — and that the DAT is `bytes = on` (text mode silently drops the data).
- **Bit-exact, but bandwidth-hungry.** 50k points × 16 bytes × frame rate is a steady stream; on flaky Wi-Fi frames stutter. Wire the connection for a show.
- **Alpha is sent but unused.** The wire carries RGBA; the standard chain uses RGB only. If you want the `a` byte, read it in the parser — it isn't on `out_pop` by default.
- **One frame at a time.** The POP always shows the latest frame; it is not an accumulating scan. Use a Cache if you want to hold or build up geometry over time.
