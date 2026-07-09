---
title: "Depth"
parent: Operators
---
# Depth — `tdlidar_depth`

> Get the phone's depth-driven visual — raw, plasma, organic and friends — into TouchDesigner as a video TOP with zero OSC wiring.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** a depth camera on the phone (LiDAR rear or TrueDepth front) · same LAN as TD

## What it does
The TDLiDAR app turns the phone's depth camera into a moving visual — the **Depth** modes: Raw greyscale depth, Plasma (a depth-warped nebula), Organic (a depth-seeded reaction-diffusion), and the other colour-mapped looks. The app does the depth processing and colouring on-device and broadcasts the finished picture over NDI. This op receives that NDI source into a TOP, so you get a ready-made depth visual to composite, key or feed into 3D — instantly, without touching any sensor channels.

## NDI in (not OSC)
This op receives **NDI video**, not OSC. Pick the depth source from the dropdown:

| dropdown entry | what it is |
|---|---|
| `LOCALHOST (TDLidar)` | the depth visual stream — whatever Depth mode (raw/plasma/organic/…) the app is currently showing |

The source name follows the app's NDI name index (`TDLidar`, `TDLidar 2`, …) so multiple phones stay distinct. Whichever Depth mode is selected **in the app** is what arrives — switch modes on the phone and the TOP follows.

## Outputs
- `out_top` (TOP) — the live depth visual, full resolution and frame rate as the phone sends it.

## Parameters

| par | default | what it does |
|---|---|---|
| Source Name | (first found) | which NDI source to receive (the depth stream) |
| Receive Format | RGBA | colour format to decode into (RGBA preserves any alpha the mode sends) |

## Quick start (beginner)
1. In the TDLiDAR app, open the **Depth** mode and pick a look (Raw, Plasma, Organic, …). That starts the NDI source.
2. Drop the **Depth** op and choose the matching entry in the **Source Name** dropdown.
3. The tile previews the depth visual; `out_top` carries it live.
4. Wire `out_top` into a **Composite TOP** or straight onto a **Render** — you have depth visuals on screen in seconds.

## Advanced patterns
- **Recolour in TD:** receive the **Raw** depth mode and run `out_top` through a **Lookup TOP** (depth → custom ramp) to apply your own palette live, instead of baking the colour on the phone.
- **Displace 3D geometry:** depth maps to height — feed `out_top` into a **Displace TOP** or sample it in a GLSL/POP pass to push points by brightness, turning the flat depth image into relief that reacts to the real scene.
- **Key the subject:** modes that send a transparent background drop straight onto your composite when decoded as **RGBA** — or build a **Threshold/Chroma Key TOP** to isolate near depth from far.
- **Drive parameters from the picture:** an **Analyze TOP** (average/max) over `out_top` gives you a single number — overall nearness/activity — to modulate audio-reactive or feedback systems off the depth feed.

## Gotchas
- **NDI, not OSC.** No OSC Port here — an empty dropdown means a network/firewall issue or the app isn't streaming, not a port mismatch.
- **The app picks the look.** This op shows whatever Depth mode the phone is on; to change Plasma↔Organic↔Raw you switch *in the app* (or recolour Raw in TD).
- **Same LAN, mDNS allowed.** NDI discovery is Bonjour-based; client-isolated/guest Wi-Fi hides the source even with working internet.
- **It's a picture, not metric depth.** This is the colour-mapped *visual*, not per-pixel metres. For exact 3D points use the **Point Cloud** op; for room geometry use **Scene Build**.
