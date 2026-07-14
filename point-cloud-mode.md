---
title: Point Cloud Mode
layout: default
parent: App Guide
nav_order: 3
---

# Point Cloud Mode

Point Cloud mode streams a live, three-dimensional point cloud from the phone's LiDAR straight into TouchDesigner. Unlike the depth video in LiDAR mode, this is real geometry: each point carries an exact 32-bit position and colour. It travels over a TCP connection rather than being squeezed through a video codec, so the points arrive bit-accurate — what the phone measures is what TouchDesigner receives, with no compression artefacts.

**Not just TouchDesigner.** The same live point cloud drives **Blender** (via the TDLiDAR Blender add-on) and **Arkestra** too:

<div class="integrations" markdown="0">
  <a class="intg" href="https://www.patreon.com/posts/161889303" target="_blank" rel="noopener noreferrer">
    <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12.51 13.214c.046-.8.438-1.506 1.03-2.006a3.424 3.424 0 0 1 2.212-.79c.85 0 1.631.3 2.211.79.592.5.983 1.206 1.028 2.005.045.823-.285 1.586-.865 2.153a3.389 3.389 0 0 1-2.374.938 3.393 3.393 0 0 1-2.376-.938c-.58-.567-.91-1.33-.865-2.152M7.35 14.831c.006.314.106.922.256 1.398a7.372 7.372 0 0 0 1.593 2.757 8.227 8.227 0 0 0 2.787 2.001 8.947 8.947 0 0 0 3.66.76 8.964 8.964 0 0 0 3.657-.772 8.285 8.285 0 0 0 2.785-2.01 7.428 7.428 0 0 0 1.592-2.762 6.964 6.964 0 0 0 .25-3.074 7.123 7.123 0 0 0-1.016-2.779 7.764 7.764 0 0 0-1.852-2.043h.002L13.566 2.55l-.02-.015c-.492-.378-1.319-.376-1.86.002-.547.382-.609 1.015-.123 1.415l-.001.001 3.126 2.543-9.53.01h-.013c-.788.001-1.545.518-1.695 1.172-.154.665.38 1.217 1.2 1.22V8.9l4.83-.01-8.62 6.617-.034.025c-.813.622-1.075 1.658-.563 2.313.52.667 1.625.668 2.447.004L7.414 14s-.069.52-.063.831zm12.09 1.741c-.97.988-2.326 1.548-3.795 1.55-1.47.004-2.827-.552-3.797-1.538a4.51 4.51 0 0 1-1.036-1.622 4.282 4.282 0 0 1 .282-3.519 4.702 4.702 0 0 1 1.153-1.371c.942-.768 2.141-1.183 3.396-1.185 1.256-.002 2.455.41 3.398 1.175.48.391.87.854 1.152 1.367a4.28 4.28 0 0 1 .522 1.706 4.236 4.236 0 0 1-.239 1.811 4.54 4.54 0 0 1-1.035 1.626"/></svg> Blender add-on
  </a>
  <a class="intg" href="https://www.arkestra.app/" target="_blank" rel="noopener noreferrer">
    <img src="{{ '/assets/images/arkestra-logo-dark.png' | relative_url }}" alt="Arkestra" />
  </a>
</div>

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
