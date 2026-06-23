---
title: FAQ
layout: default
nav_order: 7
---

# Frequently Asked Questions

Quick answers to the questions people ask most about the TDLiDAR app and its TouchDesigner operator family. If something isn't connecting, jump straight to ["No data" troubleshooting](#no-data-troubleshooting).

---

## Getting started & connection

### What do I need to get going?

Three things: an iPhone with the **TDLiDAR** app from the App Store, a computer running **TouchDesigner**, and both devices on the **same local network**. Drag `TDLiDAR_family.tox` into your TD project, point the app's OSC target at your computer's IP, hit start streaming, and drop an operator from the **TAB → TDLiDAR** menu. If the operator's tile shows moving numbers, you're connected.

### Which iPhone do I need? Does it need LiDAR?

Any reasonably recent iPhone works for the bulk of the sensors — motion, audio, touch, camera vision, body/hand tracking, and more all stream over OSC on any modern device. **LiDAR is only required for the depth-based features**: the LiDAR/NDI depth modes, the Point Cloud (TCP), Scene Build (RoomPlan), Back Distance, and AR Mesh. Those need a **Pro iPhone** with the LiDAR scanner. Face tracking needs the **TrueDepth** front camera (any Face ID iPhone).

> Non-LiDAR iPhones still run almost everything — motion, body, hand, face, audio, OCR/QR, touch, AR pose, planes, and the full sensor suite. You only lose the depth-camera-specific modes.
{: .note }

### How do I connect the app to TouchDesigner?

The connection idea is the same in every mode: same network, point the app at your computer. The transport differs by mode:

- **Sensors** stream as **OSC over UDP port 9000** — this is what feeds the operator family.
- **LiDAR depth / RGB video** streams as **NDI** (auto-discovered, no IP to type).
- **The point cloud** streams as **TCP** (lossless 50k XYZ+RGB points).

In the app, open the mode you want, set the OSC target to your computer, and start streaming. The app displays your target IP and port on screen so you can sanity-check them.

### What ports does it use?

OSC sensors go to **UDP 9000** by default. NDI discovers itself automatically (nothing to configure). The point cloud uses a **TCP** connection. If you change the OSC port in the app, change the **OSC Port** parameter on each operator to match.

### How does the app find my computer? Can I just type the IP?

The app uses **Bonjour / network discovery** to find TouchDesigner receivers on your LAN, so in most cases you can pick your computer from a list rather than typing anything. You can also **enter the IP address manually** if discovery doesn't surface it (common on locked-down or segmented networks). The app shows the IP and port it's currently targeting.

### Do I need to know anything about networking or Python?

No. There's no networking knowledge required and **no Python** — the family self-installs when you drag the `.tox` in. Drop an operator, point the phone, and you're making visuals move.

---

## "No data" troubleshooting

### I dropped an operator but it shows nothing. Where do I start?

"No data" is the #1 issue with anything OSC. Walk this list in order:

1. **Same network?** Phone and computer must be on the *same* Wi-Fi/LAN.
2. **Local Network permission?** On iOS the app **cannot reach your computer at all** without it. iOS asks once on first launch; if it was denied (or the prompt was dismissed), OSC, NDI and the point cloud all silently fail. Open **iOS Settings → TDLiDAR** (the app's own settings page) and turn **Local Network** on. If in doubt, toggle it off and back on.
3. **Right IP?** The app must target *this computer's* IP. The app shows what it's pointed at.
4. **Port match?** App OSC port == the operator's **OSC Port** parameter (default 9000).
5. **Sensor enabled and streaming?** The app must be actively streaming that specific sensor.
6. **Firewall?** macOS/Windows firewalls can block inbound UDP — allow TouchDesigner.
7. **String sensor?** Speech, OCR, QR payload, NFC, and Sound ID labels need an **OSC In DAT**, not a CHOP.
8. **Stale channels?** Body/Hand only update while a subject is in frame.

> **iOS Local Network permission is the most common silent failure.** If you tapped "Don't Allow" the first time the app launched, nothing will ever arrive and there is no error — iOS simply blocks the packets. Find it under **Settings → TDLiDAR → Local Network** (it also appears under Settings → Privacy & Security → Local Network).
{: .warning }

### My phone and computer are on the same Wi-Fi but still nothing. Why?

The usual culprit is **network isolation**. Guest networks and routers with **"client isolation" (AP isolation)** deliberately stop devices from talking to each other — perfect for blocking exactly the traffic you need. Move both devices onto your normal (non-guest) Wi-Fi, or a network where client-to-client traffic is allowed.

> Conference, café, hotel, and many university Wi-Fi networks isolate clients by default. A cheap travel router or a personal hotspot is the reliable fallback for live shows.
{: .tip }

### How do I know I've got the right IP?

The app displays the IP and port it is sending to. That IP must be **your computer's** address on the LAN — not the phone's, not the router's. On macOS check System Settings → Network; on Windows run `ipconfig`. If the computer has multiple interfaces (Wi-Fi plus Ethernet), make sure the IP matches the interface that's actually on the same subnet as the phone.

### It connected yesterday and now it doesn't.

DHCP probably reassigned your computer a different IP overnight. Re-check the IP in the app and update the target. To avoid this, give your computer a **static/reserved IP** in your router, or use **Wired Mode** (see below) which sidesteps Wi-Fi entirely.

### Could my firewall be the problem?

Yes — this is common on a fresh computer. macOS and Windows firewalls block **inbound UDP** by default, which silently drops every OSC packet. Allow **TouchDesigner** through the firewall (macOS: System Settings → Network → Firewall → Options; Windows: allow the app on private networks). NDI also needs inbound access through the firewall.

### I see numbers but no text — Speech/OCR/QR/NFC/Sound shows nothing in a CHOP.

That's expected. **String payloads cannot travel through a CHOP** — a CHOP only carries numbers. Speech transcripts, OCR text, QR/barcode payloads, NFC tag data, and Sound ID labels are **strings**, so you must read them with an **OSC In DAT**. The string operators in the family are already wired this way; if you're rolling your own, point an OSC In DAT at the same `/tdlidar/…` address. See the [OSC Sensor Guide]({{ '/osc-sensor-guide.html' | relative_url }}).

### Why do Body/Hand channels freeze or go stale?

Body, Hand, and Animal pose channels **only update while a subject is actually in the camera frame**. When nobody is detected, the app stops sending fresh skeleton/landmark values and they hold their last position. Each pose operator exposes a `/detected` (or `/count`/`frame` heartbeat) channel — gate your patch on that so you can tell "subject is gone" apart from "data is frozen." The Hand op's `…/hands/frame` counter increments every frame so you always know data is live.

---

## Modes

### What's the difference between LiDAR, Point Cloud, Scene Build, and Sensors?

The app does four different jobs and each travels over a different transport:

| Mode | What it produces | Transport | Receives in TD as |
|---|---|---|---|
| **LiDAR** | Colour-mapped depth (+ optional RGB) | NDI | TOP (via NDI op) |
| **Point Cloud** | Live lossless 3D point cloud | TCP | POP (via Point Cloud op) |
| **Scene Build** | RoomPlan room scan | — | review on device |
| **Sensors** | Dozens of individual sensors | OSC/UDP 9000 | CHOP/DAT (the operator family) |

Pick a mode with the mode switcher at the top of the app. The **Sensors** mode is the one that feeds the operator family.

### Can I run several modes at once?

The four **modes** are mutually exclusive on the phone — switching from, say, LiDAR to Sensors tears down the previous capture cleanly before starting the next (so the switch is safe and won't leave a stream half-running). Within **Sensors mode**, however, you can enable **as many individual sensors as you like** — they all multiplex onto the same UDP port 9000.

### Which sensors can't run together?

Anything that needs a **camera engine** conflicts with another camera engine — the phone can only point one capture pipeline at a time. The app **greys out** the conflicting choices so you can't pick an impossible combination. As a rough map:

- **AR-world sensors** (6DoF Pose, AR Planes, AR Mesh, Ambient Light, Back Distance) share one ARKit session and coexist.
- **Body** pairs only with **Hands** (they share the same Vision pass).
- **ARKit Body** and the **depth session** are mutually exclusive.
- **Face** uses the front TrueDepth camera, so it conflicts with back-camera engines.

Non-camera sensors (motion, audio, touch, device, AirPods, Watch) run freely alongside any of these.

---

## Devices & support

### Which features need a Pro iPhone, TrueDepth, an iPad, or an Apple Watch?

- **Pro iPhone (LiDAR):** LiDAR/NDI depth modes, Point Cloud, Scene Build, Back Distance, AR Mesh.
- **TrueDepth front camera:** Face tracking (blendshapes, head transform, gaze) and Front Distance.
- **iPad + Apple Pencil:** the Apple Pencil operator (position, pressure, tilt, azimuth, barrel-tap).
- **Apple Watch (companion app):** heart rate, watch accel/rotation, Digital Crown.

Every operator's own doc page lists exactly what it requires.

### I have a non-LiDAR iPhone — what still works?

Almost everything. Motion (accel, gyro, gravity, attitude, magnetometer, barometer, activity), body and hand tracking, face tracking (if it has TrueDepth), AR device pose, AR planes, audio analysis, speech, sound ID, OCR, QR/barcode, rectangle/saliency detection, touch, proximity, NFC, remote/volume, AirPods, and Apple Watch all stream fine. You only miss the depth-camera modes that physically need the LiDAR scanner.

### Can I run multiple phones on one network?

Yes. For NDI, each phone has a **Depth Source name** so you can tell them apart in the NDI op's source-name dropdown — give each device a distinct name and select the one you want per operator. For OSC, point each phone at the computer and either separate them by port or address-prefix in your patch.

---

## TouchDesigner family

### How do I install the operator family?

Extract the release zip, keeping `TDLiDAR_family.tox` and the `operators/` folder **together**. In TouchDesigner, **drag `TDLiDAR_family.tox`** into your project — it self-installs on load and the **TDLiDAR** tab appears in the OP Create (TAB) menu. Save your `.toe`. Full steps are on the [Install]({{ '/install.html' | relative_url }}) page.

> Zero-config tip: extract `operators/` right next to your `.toe` file and the default relative path resolves automatically — nothing to configure.
{: .tip }

### The operators don't show up after I dropped the .tox. Now what?

Select the dropped **`TDLiDAR_fam`** component, set its **`Opfolder`** parameter to the package's **`operators/`** folder, pulse **`Ensuremanifests`**, then toggle **`Install`** On. Save the project. This forces the family to re-scan and register its operators.

### Where do the operators appear?

In the **TAB / OP Create menu**, under the blue **TDLiDAR** family. Each operator only wires to its own type, exactly like native TD — a CHOP op connects to CHOPs, a TOP op to TOPs, a POP op to POPs.

### Do I need Python installed?

No. The family is built on the dotsimulate **TDFam** runtime bundled inside `TDLiDAR_family.tox`, and it self-bootstraps. No external Python, no dependencies.

### What version of TouchDesigner do I need?

**TouchDesigner 2023.30000 or newer.** The 3D operators (Body, Hand, Point Cloud, etc.) use **POPs**, which only exist in that build and later.

---

## Pro & pricing

### What does Pro unlock?

Pro unlocks the advanced/heavier capabilities — **Extended LiDAR range**, **HD** streaming, **depth-mask alpha** (transparent-background keying), and the **advanced sensors**. The core sensor suite and basic streaming work without Pro; Pro removes the limits and adds the higher-fidelity outputs. See [Performance & Pro]({{ '/app-guide.html' | relative_url }}) in the App Guide.

### Does Pro work offline?

Yes. Once unlocked, Pro entitlements are cached on the device so it keeps working without a network connection. It reconciles with the App Store the next time you're online.

### How do I restore my purchase on a new phone?

Use **Restore Purchases** in the app (StoreKit). As long as you're signed into the same Apple ID, your entitlement is restored — Apple syncs it via your account, so there's nothing to re-buy.

---

## Recording & performance

### Can I record what the app is capturing?

Yes — the app can record and save clips straight to your **Photos** library. Recording runs alongside the live NDI/OSC streams without interrupting them.

### What happens if I background the app or get a phone call while recording?

Recording **finalizes cleanly** when the app is backgrounded — it closes out the file properly so you don't end up with a corrupted clip. That's why a recording survives an interruption instead of being lost.

### What is "Supercharge"?

**Supercharge** enlarges the internal frame buffer pool so the streaming pipeline has more headroom and is less likely to stall under load. It trades memory for smoothness. If the phone is thermally throttling, turning Supercharge **off** (and dropping resolution) is the first thing to try.

### What do "Keep Screen On" / "Keep Alive in Background" do?

**Keep Screen On** stops iOS from auto-locking mid-performance (the screen staying on keeps the capture pipeline fully active). **Keep Alive in Background** lets streaming continue when the app isn't frontmost. Use both for tripod-mounted, unattended live setups.

### My frame rate keeps dropping. Why?

Almost always **thermal throttling**. The frame-rate pill in LiDAR mode is your health check — if it sits well below your target, the phone is hot and is silently lowering capture rates. Drop the resolution, turn off Supercharge, and give the device airflow. OSC sensor rates also scale down under thermal load (the ~50 Hz motion rates assume a cool iPhone 15 Pro).

### What is NDI Wired Mode?

**NDI Wired Mode** sends the NDI video stream over a **USB cable** instead of Wi-Fi, giving you a rock-solid, low-latency, congestion-free link for the depth/RGB video — ideal for installations and live shows where Wi-Fi is unreliable. (OSC sensors still travel over the network.)

---

## Other receivers

### Can I use this with something other than TouchDesigner?

Yes. The **OSC wire format is completely open** — every operator is just an OSC In CHOP/DAT plus a Select; nothing is locked. Anything that speaks OSC can receive the data: **Max/MSP, Pure Data, Blender, Unity, openFrameworks, Processing, a browser** via a WebSocket-to-OSC bridge, and more. Point your receiver at UDP 9000 and read the `/tdlidar/…` addresses.

### Where's the full wire spec?

The complete address list — every sensor, type, range, and rate — is in the [OSC Reference]({{ '/osc-reference.html' | relative_url }}). For a friendlier walkthrough of how to consume each sensor (and the all-important CHOP-vs-DAT rule), see the [OSC Sensor Guide]({{ '/osc-sensor-guide.html' | relative_url }}).

> If you build against the raw OSC, remember the one rule that trips everyone up: **numbers → OSC In CHOP, strings → OSC In DAT.** Speech, OCR, QR payload, NFC, and Sound ID labels are strings.
{: .warning }
