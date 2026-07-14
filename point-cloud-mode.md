---
title: Point Cloud Mode
layout: default
parent: App Guide
nav_order: 3
---

# Point Cloud Mode

Point Cloud mode streams a live, three-dimensional point cloud from the phone's LiDAR straight into TouchDesigner. Unlike the depth video in LiDAR mode, this is real geometry: each point carries an exact 32-bit position and colour. It travels over a TCP connection rather than being squeezed through a video codec, so the points arrive bit-accurate — what the phone measures is what TouchDesigner receives, with no compression artefacts.

## How it works

The phone captures the LiDAR depth, projects it into 3D points, attaches the colour-camera pixel at each point, and sends a frame of points over TCP to your computer. On the TouchDesigner side the Point Cloud operator receives the frames and rebuilds them as a POP — a point operator — that you can render directly, instance geometry onto, or feed into any POP chain.

## The live screen and its buttons

The screen shows the **on-device viewer**: a rendered preview of the cloud as it is captured, which you can orbit and frame on the phone so you know what you're sending. The **gear** opens the settings, and the main capture toggle starts and stops the stream.

## Streaming settings

**TD IP** and **TD Port** tell the phone where to send the cloud — your computer's address and the TCP port. Use the network discovery dropdown to pick your machine by name, or type the IP. Match the port to the Point Cloud operator in TouchDesigner.

**Stream Point Cloud over TCP** is the on switch. Turn it on and the phone begins sending frames; turn it off to stop.

**Density** controls how many points each frame carries. Fewer points are lighter on the network and on the receiver and keep the frame rate high; more points give a denser, more detailed cloud at a higher cost. Set it to the most your scene and machine can comfortably carry.

## Viewer settings

The on-device viewer has its own appearance controls that affect only the preview on the phone, not the data you send. **Point Size** sets how large each point draws. **Zoom** and **Field of View** frame the preview camera. **Viewer Flips** are three toggles — X, Y and Z — that mirror the preview along each axis, useful when the cloud comes in mirrored relative to how you expect to see it; flip the matching axis to set it right.

## Capturing a still cloud (PLY)

Besides streaming, you can capture a single frozen cloud to a file. Pick a **PLY Density** — five hundred, one thousand, two thousand or five thousand points — and tap **Save Point Cloud as PLY**. The app writes a standard PLY file you can open in Blender, MeshLab, CloudCompare or any 3D tool, or bring back into TouchDesigner later. Lower densities make small, shareable files; higher densities preserve more of the scan.

## Receiving it in TouchDesigner

The Point Cloud operator in the family is a complete receiver: it opens the TCP port, parses the incoming frames, and outputs a POP of the points with their positions and colours. Drop it, set its port to match the app, start streaming on the phone, and the points appear. From there it is ordinary POP geometry — render it, instance boxes or sprites onto every point, or run it through the rest of your network.

### To Use the Point Cloud TCP send functionality in TouchDesigner, Please refer to the [TDLiDAR Operator Family](https://www.patreon.com/posts/163658679)
