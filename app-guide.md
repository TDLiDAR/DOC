---
title: App Guide
layout: default
nav_order: 3
has_children: true
---

# App Guide

The TDLiDAR iPhone app is the data source. It captures the phone's sensors and streams them to your computer over the local network, where TouchDesigner (or any other receiver) picks them up. This section documents the app's UI and modes.

If you just want addresses and units, jump to the OSC Reference. If you want to wire operators, see the Operators pages. This section is about driving the phone.

## The four modes

The app does four different jobs, and you pick which one with the mode switcher. Each produces a different kind of output and travels over a different transport, so each gets its own page:

```mermaid
flowchart LR
  APP(["TDLiDAR app"])
  APP --> L["LiDAR<br/>colour-mapped depth"]
  APP --> PC["Point Cloud<br/>lossless 3D points"]
  APP --> SB["Scene Build<br/>RoomPlan scan"]
  APP --> SN["Sensors<br/>motion · body · audio · …"]
  L -- "NDI" --> TOP["NDI In → TOP"]
  PC -- "TCP" --> POP["Point Cloud → POP"]
  SB -. "on device" .-> REV["review model"]
  SN -- "OSC · 9000" --> OPS["tdlidar_* operators"]
```

- **LiDAR** — the original depth-streaming mode. Colour-mapped depth, and optionally the RGB camera, sent as an NDI video stream. See [LiDAR Mode]({% link lidar-mode.md %}).
- **Point Cloud** — a live, lossless 3D point cloud sent over TCP. See [Point Cloud Mode]({% link point-cloud-mode.md %}).
- **Scene Build** — Apple's RoomPlan room scanner. See [Scene Build Mode]({% link scene-build-mode.md %}).
- **Sensors** — dozens of individual sensors (motion, body, face, audio, touch and more) streamed as OSC. This is what feeds the operator family. See [Sensors Mode]({% link sensors-mode.md %}).

## The shared interface

A few controls appear no matter which mode you're in.

The **mode switcher** sits at the top. It collapses to a single blue arrow to stay out of the way; tap it to expand the list of modes — LiDAR, Sensors, Point Cloud, Scene Build — and tap a mode to switch.

The **Settings gear** opens the settings sheet for the current mode. The settings are mode-aware: in LiDAR you get depth and NDI controls, in Sensors you get the sensor list and per-sensor tuning, etc.

In LiDAR mode a small **frame-rate pill** shows the actual frames per second the phone is achieving. It is more than a readout — tapping it opens Settings, and its number is a quick health check (if it drops below 30 fps, you may have too many sensors or a busy Wi-Fi).

Most modes also have a **slide-up dock** at the bottom (a compact chevron) that reveals the mode's primary controls without leaving the live view.

## Connecting

Connection is the same idea in every mode — same network, point the app at your computer — but the transport differs. Because it is the single most common source of trouble, it has its own page.

## Where to go next

- [Connecting]({% link connecting.md %}) — network setup, ports, discovery, wired mode, troubleshooting.
- [LiDAR Mode]({% link lidar-mode.md %}) — every depth, tone, colour and NDI control.
- [Point Cloud Mode]({% link point-cloud-mode.md %}) — TCP streaming, the viewer, PLY export.
- [Scene Build Mode]({% link scene-build-mode.md %}) — scanning a room with RoomPlan.
- [Sensors Mode]({% link sensors-mode.md %}) — enabling sensors and per-sensor tuning.
- [Performance & Pro]({% link performance-pro.md %}) — Supercharge, background, Pro features, recording.
