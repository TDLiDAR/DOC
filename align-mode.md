---
title: Align Mode
layout: default
parent: App Guide
nav_order: 10
---

# Align Mode

Align is a projection-mapping assist tool: point the phone at a real wall, floor or screen, tap it, and get that surface's true 3D corners streamed to TouchDesigner — a rough-in for a keystone/warp network (camSchnappr, Stoner, or your own corner-pin setup) instead of eyeballing correspondence points by hand. It doesn't replace TD's own calibration tools; it gives them a real starting geometry to work from.

## How it works

Align runs ARKit world tracking with plane detection and renders the phone's own camera feed live through native AR rendering — a real, smooth camera passthrough, not a snapshot-based preview. Every plane ARKit detects shows up as a translucent, tappable quad sized to the plane's actual measured extent.

## Picking and locking a surface

Tap a quad to lock it. The view swaps from showing every candidate plane to showing 4 yellow corner handles at the locked surface's actual corners, which track the camera in real time as you move the phone — the same rendering approach the app's Sensors mode plane visualizer uses, so the geometry never lags behind or "sticks" after you move.

Tap-hold and drag any corner handle to fine-tune it by hand — the handle turns white while held, and releases yellow again when you let go. A manually-dragged corner freezes in place instead of continuing to re-sync from live ARKit tracking, so your adjustment sticks.

**Reset** (the counter-clockwise arrow icon) unlocks the surface and returns to picking mode so you can choose a different one — available both in the normal view and centred at the bottom of the full-screen view.

## The preview

Double-tap the camera preview to expand it to full screen; double-tap again to shrink it back. The status pill at the top shows whether Align is still searching for surfaces, waiting for you to tap one, or has a surface locked.

## Settings

**Surface type** filters what Align looks for — Any surface, Walls Only, or Floors Only — useful when a room has both and you only want to align one kind at a time. **Surface transparency** is a slider controlling how see-through the candidate/locked quads render, so you can balance seeing the room behind them against seeing the quad itself clearly. The **TouchDesigner OSC** section sets the target IP and port — Align defaults to its own dedicated **port 9102**, separate from the main 9000 sensor bus, so it can run alongside Sensors or Cue Deck without a port clash.

## What's on the wire

Once a surface is locked, Align streams its detected state, its width/height in metres, and its 4 corners (each x/y/z in metres, world space) at roughly 10 Hz. See the [Align operator page]({{ '/tdlidar_align.html' | relative_url }}) for the exact addresses.

## Needs

ARKit world tracking — works on any recent iPhone; a LiDAR device detects planes faster but isn't required.
