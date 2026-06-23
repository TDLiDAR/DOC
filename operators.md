---
title: Operators
layout: default
nav_order: 4
has_children: true
---

# Operators

The TDLiDAR family is thirty-four operators, one per sensor or output. Drop any of them and it listens on OSC port 9000 (or its own transport for the video and point-cloud ops), previews its live data on the node tile, and only wires to its own operator type — a CHOP op to CHOPs, a TOP op to TOPs, a POP op to POPs — exactly like a native TouchDesigner operator.

Each operator has its own detailed page under [`ops/`](operators.html) with its OSC input, parameters, outputs, a beginner quick-start and advanced patterns. This page is the overview.

## Motion

Acceleration, Gravity, Gyro, Quaternion / Euler, Magnetometer, Barometer and Motion Activity. These are the phone's inertial and environmental sensors, mostly three-channel CHOP outputs. Acceleration gives you shake and impact; the quaternion is the cleanest way to drive a 3D rotation; gravity tells you which way is down; the barometer reads relative altitude and pressure; motion activity reports walking, running, cycling, driving or standing still.

## Device

Battery, Thermal State, Low Power and Screen Brightness. Lightweight readings of the phone's own state. Thermal is a zero-to-three scale you can use to degrade your render before the phone throttles; battery, low-power and brightness are simple values, with the binary ones rendered as a black-to-white colour fill you can recolour and blend.

## Body and vision

Body and Hand reconstruct full skeletons from the back camera. Pinch is an expressive air-fader from the thumb-to-index distance, and Gesture reports open hand, fist, peace and point. Face delivers fifty-two ARKit blendshapes plus head pose and gaze from the front camera. AR Body is Apple's true ninety-one-joint, world-space skeleton. Device Pose tracks the phone in the room for driving a matching virtual camera. Animal tracks cats and dogs as two-dimensional keypoints.

## Scene and detection

Camera Exposure exposes ISO, shutter, white balance and focus as a light proxy. Ambient Light reads room brightness and colour temperature. AR Planes and AR Mesh report scan progress and geometry. QR / Barcode and OCR read codes and text and route them through the patch. Rectangle Detect finds quads. Saliency points at where the eye is drawn. Front Distance and Back Distance turn the depth sensors into distance fields.

## Audio

Mic Level is a single volume reading. Audio is the full reactive suite — band levels, FFT spectrum, beat and onset, and pitch and timbre channels. Speech is on-device speech-to-text rendered as live text. Sound ID names what it hears from a three-hundred-class classifier.

## Touch and input

Touch turns the phone screen into an XY pad with pressure. Apple Pencil adds pressure, tilt and azimuth on iPad. Proximity is a free physical trigger from the earpiece sensor. NFC fires a cue with a tag's payload. Volume (Remote Control) turns headset and media buttons into hands-free triggers.

## External

AirPods stream head pose for nods, shakes and head-look aiming. Apple Watch streams heart rate, motion and the Digital Crown through the companion Watch app.

## Output and utility

NDI receives the phone's depth or camera video. Point Cloud receives the lossless TCP point cloud as a POP. Scene Build reviews a RoomPlan scan. Depth brings in the colour-mapped depth modes as video. Rectangle is a simple meter whose size follows any chosen channel.

## How they connect

Each operator's connect-type is restricted to what it actually outputs, so the wire-drag menu offers only sensible targets — a single-output sensor connects to one family, a multi-output operator like Body offers CHOP, POP and TOP. See the [OSC Reference](TDLiDAR-Docs/OSC-Sensor-Guide.md) for the exact addresses each operator reads, and the [App Guide](TDLiDAR-Docs/App-Guide.md) for enabling the matching sensors on the phone.
