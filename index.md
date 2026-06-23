---
title: Home
layout: default
nav_order: 1
permalink: /
---

# TDLiDAR for TouchDesigner

**Turn an iPhone into ~40 live sensors inside TouchDesigner — with one drag‑and‑drop.**

TDLiDAR is an iOS app + a TouchDesigner **operator family**. The app streams everything a modern iPhone can sense — motion, LiDAR depth, body/hand/face tracking, audio, camera vision, touch, and more — over your local network. The operators plug straight into your patch.

No networking knowledge required. No Python. Drop an op, point the phone, make visuals move.

---

## Table of contents
- [How it works](#how-it-works)
- [Install (5 minutes)](#install-5-minutes)
- [Your first patch](#your-first-patch)
- [The operators](#the-operators)
- [App‑side setup](#app-side-setup)
- [Troubleshooting — "no data"](#troubleshooting--no-data)
- [For advanced users](#for-advanced-users)
- [Reference docs](#reference-docs)

---

## How it works

```
 iPhone (TDLiDAR app)                    Your computer (TouchDesigner)
 ┌───────────────────┐   Wi‑Fi / LAN    ┌──────────────────────────────┐
 │ sensors → OSC ────┼─── UDP :9000 ───▶│  tdlidar_* operators → visuals│
 │ depth/RGB → NDI ──┼─── NDI ─────────▶│  NDI op → TOP                 │
 │ point cloud → TCP─┼─── TCP ─────────▶│  Point Cloud op → POP         │
 └───────────────────┘                  └──────────────────────────────┘
```

- **Most sensors** travel as **OSC** (small numeric messages) on **UDP port 9000**.
- **Video** (depth visuals, camera) travels as **NDI**.
- **The point cloud** travels as **TCP** (lossless 50k XYZ+RGB).

The phone and the computer just need to be on the **same network**.

---

## Install (5 minutes)

1. **Get the app** — install **TDLiDAR** from the App Store on an iPhone (LiDAR Pro models unlock the depth/scan sensors; everything else works on any recent iPhone).
2. **Get the family** — drag **`TDLiDAR_family.tox`** into any TouchDesigner project. The operators appear in the **TAB / OP Create** menu under the **TDLiDAR** family (blue).
3. **Connect** — on the phone, open TDLiDAR, set the OSC target to your computer's IP, start streaming. (The app shows your IP + port.)
4. **Drop an op** — TAB → TDLiDAR → e.g. **Attitude**. It listens on 9000 and its tile shows live data.

That's it. If the tile shows moving numbers, you're connected.

---

## Your first patch

The fastest "wow":

1. TAB → TDLiDAR → **Attitude** (phone tilt as pitch/roll/yaw).
2. Add a **Geometry COMP** + a **Box SOP**.
3. On the Geo's **Xform** page, drag the Attitude op's `pitch`/`roll`/`yaw` channels onto **Rotate x/y/z** (right‑click → Export CHOP). Multiply radians→degrees with a **Math CHOP** (×57.2957...).
4. Tilt the phone → the box rotates in real space.

Swap Attitude for **Pinch** (air‑fader), **Audio** (beat‑reactive), or **Body** (a puppet skeleton) and the same five steps drive anything.

---

## The operators

Every operator has its own page in the [Operators section]({% link operators.md %}). They share one rule: drop it, it listens on **OSC Port 9000**, the tile previews its live output.

**Motion** — [Acceleration](/DOC/operators/tdlidar_accel/) · [Gravity](/DOC/operators/tdlidar_gravity/) · [Gyro](/DOC/operators/tdlidar_gyro/) · [QuaternionEuler](/DOC/operators/tdlidar_attitude/) · [Magnetometer](/DOC/operators/tdlidar_magnetometer/) · [Barometer](/DOC/operators/tdlidar_barometer/) · [Activity](/DOC/operators/tdlidar_activity/)

**Device** — [Battery](/DOC/operators/tdlidar_battery/) · [Thermal State](/DOC/operators/tdlidar_thermal/) · [Low Power](/DOC/operators/tdlidar_lowpower/) · [Screen Brightness](/DOC/operators/tdlidar_brightness/)

**Body & Vision** — [Body](/DOC/operators/tdlidar_body/) · [Hand](/DOC/operators/tdlidar_hand/) · [Pinch](/DOC/operators/tdlidar_pinch/) · [Gesture](/DOC/operators/tdlidar_gesture/) · [Face](/DOC/operators/tdlidar_face/) · [ARBody](/DOC/operators/tdlidar_arbody/) · [DevicePose](/DOC/operators/tdlidar_devicepose/)

**Scene & Detect** — [Camera Exposure](/DOC/operators/tdlidar_cam_exposure/) · [Ambient Light](/DOC/operators/tdlidar_ambient/) · [AR Planes](/DOC/operators/tdlidar_planes/) · [AR Mesh](/DOC/operators/tdlidar_armesh/) · [QR/Barcode](/DOC/operators/tdlidar_qr/) · [Rectangle Detect](/DOC/operators/tdlidar_rect_detect/) · [Text (OCR)](/DOC/operators/tdlidar_ocr/) · [Saliency](/DOC/operators/tdlidar_saliency/) · [Front Distance](/DOC/operators/tdlidar_frontdist/) · [Back Distance](/DOC/operators/tdlidar_backdist/) · [Animal](/DOC/operators/tdlidar_animal/) · [Scene Room](/DOC/operators/tdlidar_scene_room/)

**Audio** — [Mic Level](/DOC/operators/tdlidar_mic_level/) · [Audio](/DOC/operators/tdlidar_audio/) · [Speech](/DOC/operators/tdlidar_speech/) · [Sound ID](/DOC/operators/tdlidar_sound/)

**Touch & Input** — [Touch](/DOC/operators/tdlidar_touch/) · [Apple Pencil](/DOC/operators/tdlidar_pencil/) · [Proximity](/DOC/operators/tdlidar_proximity/) · [NFC](/DOC/operators/tdlidar_nfc/) · [Volume / Remote](/DOC/operators/tdlidar_remote/)

**External** — [AirPods](/DOC/operators/tdlidar_airpods/) · [Apple Watch](/DOC/operators/tdlidar_watch/)

**Output / Utility** — [NDI](/DOC/operators/tdlidar_ndi_in/) · [Point Cloud](/DOC/operators/tdlidar_pointcloud_tcp/) · [Scene Build](/DOC/operators/tdlidar_scene_room/) · [Depth](/DOC/operators/tdlidar_depth/) · [Rectangle](/DOC/operators/tdlidar_rectangle/)

---

## App‑side setup

- **Port:** the app sends OSC to UDP **9000** by default. If you change it on the phone, change **OSC Port** on each op to match.
- **One sensor or many:** enable as many sensors as you want; they all multiplex onto the same port. Some camera sensors are mutually exclusive (you can't run two different camera engines at once).
- **Device support:** depth/scan/LiDAR ops need a Pro iPhone; Face needs TrueDepth; Apple Pencil needs an iPad; Apple Watch needs the companion Watch app. Each op page lists what it needs.

---

## Troubleshooting — "no data"

The #1 issue with anything OSC. Walk this list:

1. **Same network?** Phone and computer on the *same* Wi‑Fi/LAN. Guest networks and "client isolation" block it.
2. **Right IP?** The app must target *this computer's* IP. The app shows it.
3. **Port match?** App port == op's **OSC Port** (default 9000).
4. **Sensor enabled + streaming?** The app must be actively streaming that sensor.
5. **Firewall?** macOS/Windows firewall can block inbound UDP — allow TouchDesigner.
6. **String sensors** (Speech, OCR, QR payload, NFC, Sound ID label) need an **OSC In DAT**, not a CHOP — if you see numbers but no text, that's why.
7. **Stale channels?** Body/Hand only update while a subject is in frame (`/detected`). Check the op's Gotchas.

---

## For advanced users

- **Wire format is open** — see [OSC-REFERENCE](/DOC/osc-reference/). Every op is just an OSC In CHOP/DAT + a Select; nothing is locked. Build your own ops against the same addresses.
- **Smoothing:** Lag/Filter CHOP on noisy motion; the 1€ filter pattern for pose.
- **Triggers:** Trigger/Logic CHOP on momentary channels (Remote, NFC, Gesture, beat/onset).
- **Geometry:** Body/Hand/Animal output POPs you can instance geometry onto; 6DoF Pose drives a Camera COMP for projection mapping.
- **Coexistence:** consult the app's conflict rules — AR‑world sensors (Pose/Planes/Mesh/Ambient/Back Distance) share one session; Body pairs only with Hands; ARKit Body and the depth session don't mix.

---

## Reference docs

- [OSC-REFERENCE](/DOC/osc-reference/) — the complete OSC wire spec (every address, type, range, rate).
- [Operators]({% link operators.md %}) — one page per operator.
