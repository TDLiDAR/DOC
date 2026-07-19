---
title: Sensors Mode
layout: default
parent: App Guide
nav_order: 9
---

# Sensors Mode

Sensors mode is the heart of the TouchDesigner integration. Instead of video or geometry, it streams the phone's individual sensors — motion, body, hands, face, audio, touch, device state and more — as OSC messages on port 9000. Each sensor you enable adds its addresses to the stream, and the operator family on the other side turns each one into a native TouchDesigner operator.

## The dashboard and its buttons

The main screen is a dashboard of every sensor the phone can stream, grouped into Motion, Device, Camera and Vision, Audio, Touch and Input, and External. Each sensor is a card with a toggle.

**Start** begins streaming; the app opens the network connection and the enabled sensors go live. **Settings** opens the panel where you tune individual sensors and set the OSC target. Toggling a sensor card on adds that sensor to the stream immediately, and several can run at once — turn on motion, body and audio together and all three flow down the same connection.

Some camera-based sensors cannot run at the same time, because two different camera engines can't share the camera. When you enable one, the app greys out the ones that conflict with it. The rule of thumb: the back-camera Vision sensors (body, hands, pinch, gesture) are one group; the AR-world sensors (device pose, planes, mesh, ambient light, back distance) are another; the front face camera, the ARKit body, the front-distance depth, and the image-detection sensors each hold the camera exclusively. Non-camera sensors — motion, audio, touch, device state — have no such limit and stack freely.

At the top of the panel are small **OSC** and **NDI** tabs. OSC is where the sensor stream is configured; NDI appears because a couple of camera sensors can also send their preview as NDI.

Several camera sensors that draw an on-screen overlay have small action buttons in their card. **Reset** and **Reset Plane** re-zero a tracker's reference. **Scan** drives the mesh capture. **Recenter AirPods** zeroes the AirPods head pose to your current head position. The back-distance sensor shows a movable cross on the live view that you drag to aim the measurement, with a **Reset square to centre** action to put it back.

## Setting the OSC target

The OSC target is the same network setup as everywhere else: open Settings, use the discovery dropdown to pick your computer by name, or type its IP. Data goes to UDP port 9000. If you change the port here, change the OSC Port parameter on each operator in TouchDesigner to match.

## Per-sensor tuning

Most sensors have their own controls, reached by expanding the sensor in the panel.

The motion sensors — the **accelerometer** and the **rotation rate** — each give you per-axis **Sensitivity X, Y and Z** to scale each axis independently, plus a noise gate to discard small jitter before it goes out. A **calibrate** action snapshots the current orientation as the new zero for pitch, roll, yaw and rotation rate; gravity and the magnetic field always pass through raw so they stay world-true.

The **Quaternion / Euler** attitude sensor has a sensitivity control and a **Reference Frame** picker — Arbitrary, Corrected, Magnetic North or True North — which decides what the orientation is measured against. Magnetic and true north anchor the heading to the world; arbitrary and corrected are relative to where you started.

The **magnetometer** has a sensitivity control. The **barometer** has separate **Altitude** and **Pressure** sensitivity controls.

The **body** and **hands** sensors share the back-camera pipeline and offer several options. **Output rate** switches the stream between 30 Hz and 60 Hz. **Max hands** chooses one or two. **Hand side** filters the output to Both, Left or Right. **LiDAR depth fusion** replaces the noisy single-camera depth estimate at each landmark with real LiDAR depth when a LiDAR camera is present, which firms up the otherwise weak Z axis.

The **detection** sensors carry their own knobs. A **confidence** slider gates weak detections so only solid hits go out. A **smoothing** control steadies a jittery result. A **QR codes only** toggle ignores other barcode symbologies. A **maximum rectangles** control caps how many quads are reported at once.

The **Apple Watch** sensor needs the companion Watch app and streams acceleration, rotation and heart rate. It has a **Heart Sensitivity** control, a **Reset Watch Defaults** action, and shows whether a watch is currently connected.

The **microphone** sensors offer an **Input Source** picker. Auto prefers a connected USB audio interface and falls back to the built-in mic; you can also force a specific route such as a Bluetooth headset, line-in, a wired headset or AirPlay.

**Proximity** deserves a note. It reads the earpiece sensor at the very top of the phone, not the screen. Hold a hand or object within a few centimetres of the top edge to read near. iOS blanks the screen while near — that is normal and expected — which makes proximity an excellent hands-free blackout or cue trigger.

## The overlay

The on-screen overlay that visualizes tracking — skeleton lines, joint dots, detection boxes — has its own appearance controls: **overlay opacity**, **line thickness** and **point size**. These change only what you see on the phone; they do not change the data sent to TouchDesigner.

## What goes on the wire

Every enabled sensor broadcasts under a `/tdlidar/...` address. The exact addresses, value types, units and rates are listed in the OSC Reference. The short version: numbers arrive as OSC floats and are read with an OSC-In CHOP; text — speech, OCR strings, sound labels, NFC and QR payloads — arrives as OSC strings and must be read with an OSC-In DAT.
