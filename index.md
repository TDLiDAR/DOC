---
title: Home
layout: default
nav_order: 1
permalink: /
---

<div class="td-hero" markdown="0">
  <img class="td-hero-icon" src="{{ '/assets/images/tdlidar-icon.png' | relative_url }}" alt="TDLiDAR app icon">
  <h1 class="td-hero-title">TDLiDAR</h1>
  <p class="td-tagline">Turn an iPhone into ~40 live sensors inside TouchDesigner — with one drag-and-drop.</p>
  <div class="td-cta">
    <a class="td-appstore-badge" href="https://apps.apple.com/us/app/tdlidar/id6760954732" target="_blank" rel="noopener noreferrer">
      <img src="{{ '/assets/images/app-store-badge.svg' | relative_url }}" alt="Download TDLiDAR on the App Store">
    </a>
  </div>
  <hr class="td-hero-rule">
</div>

[Install]({% link install.md %}){: .btn .btn-primary .mr-2 }
[App Guide]({% link app-guide.md %}){: .btn .mr-2 }
[Operators]({% link operators.md %}){: .btn .mr-2 }
[FAQ]({% link faq.md %}){: .btn }
{: .text-center }

TDLiDAR is an iOS app **plus** a TouchDesigner **operator family**. The app streams everything a modern iPhone can sense — motion, LiDAR depth, body/hand/face tracking, audio, camera vision, touch, and more — over your local network. The operators plug straight into your patch.

No networking knowledge required. No Python. Drop an op, point the phone, make visuals move.

---

## How it works

```mermaid
flowchart LR
  subgraph PHONE["📱 iPhone — TDLiDAR app"]
    direction TB
    S["Sensors&nbsp;→&nbsp;OSC"]
    V["Depth / RGB&nbsp;→&nbsp;NDI"]
    P["Point cloud&nbsp;→&nbsp;TCP"]
  end
  subgraph DESK["🖥️ Computer — TouchDesigner"]
    direction TB
    O["tdlidar_* operators → visuals"]
    N["NDI In → TOP"]
    C["Point Cloud → POP"]
  end
  S -- "UDP :9000" --> O
  V -- "NDI" --> N
  P -- "TCP" --> C
```

- **Most sensors** travel as **OSC** (small numeric messages) on **UDP port 9000**.
- **Video** (depth visuals, camera) travels as **NDI**.
- **The point cloud** travels as **TCP** (lossless 50k XYZ+RGB).

The phone and the computer just need to be on the **same network**.

> New here? Start with **[Install]({% link install.md %})**, then skim the **[App Guide]({% link app-guide.md %})**. Stuck on "no data"? Jump to the **[FAQ]({% link faq.md %})**.
{: .note }

---

## Install (5 minutes)

1. **Get the app** — install **TDLiDAR** from the [App Store](https://apps.apple.com/us/app/tdlidar/id6760954732) on an iPhone (LiDAR Pro models unlock the depth/scan sensors; everything else works on any recent iPhone).
2. **Get the family** — drag **`TDLiDAR_family.tox`** into any TouchDesigner project. The operators appear in the **TAB / OP Create** menu under the **TDLiDAR** family (blue). See [Install]({% link install.md %}).
3. **Connect** — on the phone, open TDLiDAR, set the OSC target to your computer's IP, start streaming. (The app shows your IP + port.)
4. **Drop an op** — TAB → TDLiDAR → e.g. **Attitude**. It listens on 9000 and its tile shows live data.

That's it. If the tile shows moving numbers, you're connected.

---

## Your first patch

The fastest "wow":

1. TAB → TDLiDAR → **[QuaternionEuler]({% link tdlidar_attitude.md %})** (phone tilt as pitch/roll/yaw).
2. Add a **Geometry COMP** + a **Box SOP**.
3. On the Geo's **Xform** page, drag the op's `pitch`/`roll`/`yaw` channels onto **Rotate x/y/z** (right‑click → Export CHOP). Multiply radians→degrees with a **Math CHOP** (×57.2957).
4. Tilt the phone → the box rotates in real space.

Swap it for **[Pinch]({% link tdlidar_pinch.md %})** (air‑fader), **[Audio]({% link tdlidar_audio.md %})** (beat‑reactive), or **[Body]({% link tdlidar_body.md %})** (a puppet skeleton) and the same five steps drive anything.

---

## The operators

Every operator has its own page in the [Operators section]({% link operators.md %}). They share one rule: drop it, it listens on **OSC Port 9000**, and the tile previews its live output.

**Motion** — [Acceleration]({% link tdlidar_accel.md %}) · [Gravity]({% link tdlidar_gravity.md %}) · [Gyro]({% link tdlidar_gyro.md %}) · [QuaternionEuler]({% link tdlidar_attitude.md %}) · [Magnetometer]({% link tdlidar_magnetometer.md %}) · [Barometer]({% link tdlidar_barometer.md %}) · [Activity]({% link tdlidar_activity.md %})

**Device** — [Battery]({% link tdlidar_battery.md %}) · [Thermal State]({% link tdlidar_thermal.md %}) · [Low Power]({% link tdlidar_lowpower.md %}) · [Screen Brightness]({% link tdlidar_brightness.md %})

**Body & Vision** — [Body]({% link tdlidar_body.md %}) · [Hand]({% link tdlidar_hand.md %}) · [Pinch]({% link tdlidar_pinch.md %}) · [Gesture]({% link tdlidar_gesture.md %}) · [Face]({% link tdlidar_face.md %}) · [AR Body]({% link tdlidar_arbody.md %}) · [Device Pose]({% link tdlidar_devicepose.md %})

**Scene & Detect** — [Camera Exposure]({% link tdlidar_cam_exposure.md %}) · [Ambient Light]({% link tdlidar_ambient.md %}) · [AR Planes]({% link tdlidar_planes.md %}) · [AR Mesh]({% link tdlidar_armesh.md %}) · [QR / Barcode]({% link tdlidar_qr.md %}) · [Rectangle Detect]({% link tdlidar_rect_detect.md %}) · [Text (OCR)]({% link tdlidar_ocr.md %}) · [Saliency]({% link tdlidar_saliency.md %}) · [Front Distance]({% link tdlidar_frontdist.md %}) · [Back Distance]({% link tdlidar_backdist.md %}) · [Animal]({% link tdlidar_animal.md %}) · [Scene Room]({% link tdlidar_scene_room.md %})

**Audio** — [Mic Level]({% link tdlidar_mic_level.md %}) · [Audio]({% link tdlidar_audio.md %}) · [Speech]({% link tdlidar_speech.md %}) · [Sound ID]({% link tdlidar_sound.md %})

**Touch & Input** — [Touch]({% link tdlidar_touch.md %}) · [Apple Pencil]({% link tdlidar_pencil.md %}) · [Proximity]({% link tdlidar_proximity.md %}) · [NFC]({% link tdlidar_nfc.md %}) · [Volume / Remote]({% link tdlidar_remote.md %})

**External** — [AirPods]({% link tdlidar_airpods.md %}) · [Apple Watch]({% link tdlidar_watch.md %})

**Output / Utility** — [NDI]({% link tdlidar_ndi_in.md %}) · [Point Cloud]({% link tdlidar_pointcloud_tcp.md %}) · [Scene Build]({% link tdlidar_scene_room.md %}) · [Depth]({% link tdlidar_depth.md %}) · [Rectangle]({% link tdlidar_rectangle.md %})

---

## App‑side setup

- **Port:** the app sends OSC to UDP **9000** by default. If you change it on the phone, change **OSC Port** on each op to match.
- **One sensor or many:** enable as many sensors as you want; they all multiplex onto the same port. Some camera sensors are mutually exclusive (you can't run two different camera engines at once).
- **Device support:** depth/scan/LiDAR ops need a Pro iPhone; Face needs TrueDepth; Apple Pencil needs an iPad; Apple Watch needs the companion Watch app. Each op page lists what it needs.

The full walkthrough — every mode and every setting — is in the **[App Guide]({% link app-guide.md %})**, including deep references for **[every LiDAR setting]({% link lidar-settings.md %})** and **[every Point Cloud control]({% link point-cloud-settings.md %})**.

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

> More answers in the **[FAQ]({% link faq.md %})**.
{: .tip }

---

## For advanced users

- **Wire format is open** — see the [OSC Reference]({% link osc-reference.md %}) and the plain-language [OSC Sensor Guide]({% link osc-sensor-guide.md %}). Every op is just an OSC In CHOP/DAT + a Select; nothing is locked. Build your own receivers against the same addresses (Max, PD, Blender, Unity, openFrameworks, a browser).
- **Smoothing:** Lag/Filter CHOP on noisy motion; the 1€ filter pattern for pose.
- **Triggers:** Trigger/Logic CHOP on momentary channels (Remote, NFC, Gesture, beat/onset).
- **Geometry:** Body/Hand/Animal output POPs you can instance geometry onto; 6DoF Pose drives a Camera COMP for projection mapping.
- **Coexistence:** consult the app's conflict rules — AR‑world sensors (Pose/Planes/Mesh/Ambient/Back Distance) share one session; Body pairs only with Hands; ARKit Body and the depth session don't mix.

---

## Reference docs

- [App Guide]({% link app-guide.md %}) — every mode and setting, top to bottom.
- [LiDAR — Every Setting]({% link lidar-settings.md %}) — the depth/tone/colour/NDI controls and how each changes the look.
- [Point Cloud — Effects & Settings]({% link point-cloud-settings.md %}) — viewer, cleanup, streaming and PLY capture.
- [Operators]({% link operators.md %}) — one page per operator.
- [OSC Reference]({% link osc-reference.md %}) — the complete OSC wire spec (every address, type, range, rate).
- [OSC Sensor Guide]({% link osc-sensor-guide.md %}) — plain-language tour of the wire format.
- [FAQ]({% link faq.md %}) — quick answers, especially for "no data".

---

<p class="text-center text-grey-dk-000">
  Made by <a href="https://www.patreon.com/aristideslab" target="_blank" rel="noopener noreferrer">Aristides Lab</a> ·
  <a href="https://apps.apple.com/us/app/tdlidar/id6760954732" target="_blank" rel="noopener noreferrer">App Store</a> ·
  <a href="https://github.com/TDLiDAR/DOC" target="_blank" rel="noopener noreferrer">GitHub</a>
</p>
