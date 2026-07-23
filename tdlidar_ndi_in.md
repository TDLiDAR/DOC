---
title: "NDI"
parent: Operators
---
# NDI — `tdlidar_ndi_in`

> Pull the phone's live camera or depth picture straight into TouchDesigner as a video TOP — no OSC wiring, no cables.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** the phone and the TD machine on the same LAN (NDI is video over the network, not OSC)

## What it does
Receives the video the TDLiDAR app is broadcasting over NDI and lands it as a TOP. The phone advertises one or more NDI sources on the local network — its depth visual (raw/plasma/organic/etc.), its plain RGB camera, or the point-cloud viewport — and this op picks one by name and decodes it into a texture you can composite, key and colour-map like any other TOP. The tox tile previews whatever the chosen source is sending.

## NDI in (not OSC)
This op does **not** listen on an OSC port. It listens for **NDI** video sources on the LAN. Pick the source with the dropdown:

| dropdown entry | what it is |
|---|---|
| `LOCALHOST (TDLidar)` | the depth visual stream (raw / plasma / organic / colour-mapped depth) |
| `LOCALHOST (TDLidar PC)` | the point-cloud viewport render |
| `LOCALHOST (TDLOSC-1 Cam)` | the plain RGB camera feed |
| `TDLMN` | Monocular Depth mode's colour-mapped depth (camera-only, no LiDAR needed) — always this fixed name, distinct from the LiDAR depth source above |
| `TDLLM` | LiDAR + Monocular Depth mode's fused, metric depth — requires the rear LiDAR scanner, always this fixed name |

Source names change with the app's NDI name index (e.g. `TDLidar`, `TDLidar 2`, …) so several phones can co-exist on one network. The dropdown auto-populates from whatever is actually broadcasting — if it's empty, nothing is being sent yet.

## Outputs
- `out_top` (TOP) — the decoded NDI video, full resolution and frame rate as the phone sends it. This is the texture you composite.

## Parameters

| par | default | what it does |
|---|---|---|
| Source Name | (first found) | which NDI source to receive — the dropdown of advertised names |
| Receive Format | RGBA | colour format to decode into (use RGBA to keep any alpha key the app sends) |

## Quick start (beginner)
1. In the TDLiDAR app, start a stream (Depth, Camera or Point Cloud) — that's what turns the NDI source on.
2. Drop the **NDI** op. Open the **Source Name** dropdown and pick the matching entry (e.g. `LOCALHOST (TDLidar)`).
3. The tile previews the phone's picture; `out_top` now carries it live.
4. Wire `out_top` into a **Composite TOP** or **Level TOP** and you're mixing the phone feed into your scene.

## Advanced patterns
- **Colour-map a depth stream:** if you're receiving the raw depth visual, drive a **Lookup TOP** (depth → ramp) off `out_top` to recolour it in TD instead of on the phone — lets you change the palette live without touching the device.
- **Key it onto a scene:** the app can send a transparent background on some modes. Decode as **RGBA** and the alpha drops the phone subject straight onto your composite — or build your own key with a **Chroma Key / Threshold TOP**.
- **Feed depth into 3D:** a depth TOP can displace geometry — run `out_top` into a **Displace TOP** or sample it in a GLSL/POP setup to push points by brightness, turning the 2D depth picture into relief.
- **Sync with the sensors:** the NDI picture and the OSC sensor ops share a clock loosely (both come off the same phone). Composite the NDI feed *under* skeleton/face ops driven by OSC for a matched camera-plus-data look.

## Gotchas
- **This is NDI, not OSC.** There is no OSC Port here — if the dropdown is empty, the problem is the network/firewall or the app simply isn't streaming, not a port mismatch.
- **Same LAN, mDNS allowed.** NDI discovery uses mDNS/Bonjour; a guest Wi-Fi or client-isolation router will hide the source even when both devices have internet.
- **Source name follows the app's index.** If you renamed the phone's NDI source (or run two phones), the dropdown entry changes — re-pick it; an export pinned to a stale name goes black.
- **Bandwidth is real video.** Full-res NDI is tens of Mbit/s; on a busy wireless network expect frame drops. Wire the phone (USB-Ethernet) or use 5 GHz for a clean show.
