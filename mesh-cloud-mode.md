---
title: Mesh Cloud Mode
layout: default
parent: App Guide
nav_order: 7
---

# Mesh Cloud Mode

Mesh Cloud scans a space with ARKit's LiDAR mesh reconstruction and turns it into a point cloud you can walk around on the phone, clean up, and send to TouchDesigner or export as a file. It sits between Point Cloud mode (a live, un-edited stream) and Scene Build (RoomPlan's clean architectural model) — a capture you finish once, then review and hand off.

**Not just TouchDesigner.** The finished Mesh Cloud drives **Blender** (via the TDLiDAR Blender add-on) and **Arkestra** too — and exports as PLY for any 3D tool:

<div class="integrations" markdown="0">
  <a class="intg" href="https://www.patreon.com/posts/161889303" target="_blank" rel="noopener noreferrer">
    <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12.51 13.214c.046-.8.438-1.506 1.03-2.006a3.424 3.424 0 0 1 2.212-.79c.85 0 1.631.3 2.211.79.592.5.983 1.206 1.028 2.005.045.823-.285 1.586-.865 2.153a3.389 3.389 0 0 1-2.374.938 3.393 3.393 0 0 1-2.376-.938c-.58-.567-.91-1.33-.865-2.152M7.35 14.831c.006.314.106.922.256 1.398a7.372 7.372 0 0 0 1.593 2.757 8.227 8.227 0 0 0 2.787 2.001 8.947 8.947 0 0 0 3.66.76 8.964 8.964 0 0 0 3.657-.772 8.285 8.285 0 0 0 2.785-2.01 7.428 7.428 0 0 0 1.592-2.762 6.964 6.964 0 0 0 .25-3.074 7.123 7.123 0 0 0-1.016-2.779 7.764 7.764 0 0 0-1.852-2.043h.002L13.566 2.55l-.02-.015c-.492-.378-1.319-.376-1.86.002-.547.382-.609 1.015-.123 1.415l-.001.001 3.126 2.543-9.53.01h-.013c-.788.001-1.545.518-1.695 1.172-.154.665.38 1.217 1.2 1.22V8.9l4.83-.01-8.62 6.617-.034.025c-.813.622-1.075 1.658-.563 2.313.52.667 1.625.668 2.447.004L7.414 14s-.069.52-.063.831zm12.09 1.741c-.97.988-2.326 1.548-3.795 1.55-1.47.004-2.827-.552-3.797-1.538a4.51 4.51 0 0 1-1.036-1.622 4.282 4.282 0 0 1 .282-3.519 4.702 4.702 0 0 1 1.153-1.371c.942-.768 2.141-1.183 3.396-1.185 1.256-.002 2.455.41 3.398 1.175.48.391.87.854 1.152 1.367a4.28 4.28 0 0 1 .522 1.706 4.236 4.236 0 0 1-.239 1.811 4.54 4.54 0 0 1-1.035 1.626"/></svg> Blender add-on
  </a>
  <a class="intg" href="https://www.arkestra.app/" target="_blank" rel="noopener noreferrer">
    <img src="{{ '/assets/images/arkestra-logo-dark.png' | relative_url }}" alt="Arkestra" />
  </a>
</div>

## How it works

While scanning, ARKit builds up a 3D mesh of the room from the LiDAR sensor as you move the phone around. Rather than streaming that raw mesh, the app resamples it into a point cloud: space is divided into a voxel grid, and every occupied voxel becomes one averaged point. Coarser voxels mean fewer, larger-spaced points; finer voxels mean a denser cloud, up to a point cap the app enforces automatically by coarsening the grid until it fits.

## Scanning

The live scan screen shows the growing point cloud directly from the AR camera's point of view as you move. A **Scan** stage runs continuously; the phone is deliberately conservative about incorporating new mesh data while it's rotating quickly, since fast turns are the main source of drift in a live mesh scan. Move slowly and give each surface a moment to settle in.

**Finish** ends the scan and freezes the current cloud, handing off to the review screen.

## Review — the free-fly viewer

Once finished, the same view expands into a free-fly 3D viewer you can navigate independently of the phone's own orientation: an on-screen joystick moves you through the space, a zoom dial and pinch gesture control distance, and drag orbits the camera. This is a proper walkthrough of the capture, not just a locked preview.

An **edit mode** toggle turns on point cleanup: drag a rectangle over the view to select points (through-selection — it grabs everything under the rectangle regardless of depth), then trash the selection to delete it. Undo and redo cover edit history.

Four buttons handle what happens next:
- **Send** streams the current cloud once, over TCP, to TouchDesigner — the same wire format and receiving operator as Point Cloud mode's `tdlidar_pointcloud_tcp`, just pointed at Mesh Cloud's own port (default **9004**, independent of Point Cloud mode's own port setting) so the two modes don't collide if you use both.
- **Export** saves the cloud as a binary PLY file through the system share sheet, for Blender, MeshLab, CloudCompare or any 3D tool.
- **Rescan** discards the current cloud and starts a fresh scan.
- **Settings** opens the mode's full settings panel.

## Settings

**Quantization** sets the voxel size in centimetres — lower is denser (and heavier); the app automatically coarsens beyond this if the point cap is hit. **Point size** scales how large each point renders in the viewer only (cosmetic, not part of the sent/exported data). **Move**, **Look** and **Zoom** speed tune the free-fly navigation feel. **Max points** caps the cloud size.

A **Clean-up** section has three live toggles you can flip mid-review to compare the effect immediately: **Remove floating points** (on by default) drops voxels with no neighbour — pure noise specks; **Keep largest piece only** discards every fragment except the single largest contiguous piece of the scan; **Colour incomplete areas** shades well-surrounded points by distance from camera but marks sparse/edge points in warm amber, so you can see at a glance which parts of the scan are solid and which need another pass.

When you're in the review screen, the settings panel also shows the **TouchDesigner** section — the target IP and the TCP port Send will use.

## Needs

Mesh Cloud requires a LiDAR device with ARKit scene-mesh reconstruction support (the same hardware requirement as LiDAR depth mode).
