---
title: Mesh Cloud Mode
layout: default
parent: App Guide
nav_order: 8
---

# Mesh Cloud Mode

Mesh Cloud scans a space with ARKit's LiDAR mesh reconstruction and turns it into a point cloud you can walk around on the phone, clean up, and send to TouchDesigner or export as a file. It sits between Point Cloud mode (a live, un-edited stream) and Scene Build (RoomPlan's clean architectural model) — a capture you finish once, then review and hand off.

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
