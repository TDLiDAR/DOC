---
title: OSC Sensor Guide
layout: default
nav_order: 6
---

# TDLiDAR — OSC Sensor Guide

A plain-language reference to the raw OSC the TDLiDAR iPhone app broadcasts. This is the wire format itself — what travels over the network — independent of any TouchDesigner operator or patch. Use it to build your own receivers in TouchDesigner, Max, Pure Data, Blender, Unity, openFrameworks, a browser, or anything that speaks OSC.

## How the wire works

Everything is sent as standard OSC over UDP. The default destination port is 9000. Point the app at your machine's local IP address and it streams.

Every address begins with /tdlidar/. Numeric values are sent as OSC floats unless noted. Text values — recognized speech, OCR strings, sound labels, NFC payloads, QR payloads — are sent as OSC strings; receive those with an OSC-In DAT (or your platform's equivalent), not a CHOP, because a numeric channel can't hold text.

Several sensors share one address space. Body, Hands, Pinch and Gesture all live under /tdlidar/body, so turning on more than one simply adds more addresses on the same stream. A sensor only broadcasts while it is enabled in the app. Rates below are typical for an iPhone 15 Pro and drop when the phone thermally throttles.

A calibration note: pitch, roll, yaw and rotation rate are zeroed against a snapshot taken when you press calibrate in the app. Gravity and the magnetic field stay raw.

## Motion

Acceleration streams /tdlidar/motion/accel/x, /y and /z in G with gravity already removed, up to about 50 Hz. Good for shake, impact and throw detection.

Gravity streams /tdlidar/motion/gravity/x, /y and /z as a unit vector pointing toward the ground — useful for knowing which way is up.

Gyro streams the rotation rate as /tdlidar/motion/rotation/x, /y and /z in radians per second.

Quaternion / Euler attitude streams /tdlidar/motion/pitch, /roll and /yaw in radians. This is the cleanest way to drive a 3D rotation.

Magnetometer streams the raw magnetic field as /tdlidar/motion/magnetic/x, /y and /z in microtesla. Note this is the raw field, not a true compass heading.

Barometer streams /tdlidar/motion/altitude/relative in metres and /tdlidar/motion/altitude/pressure in kilopascals, roughly once a second.

Motion Activity streams a set of flags — /tdlidar/activity/walking, /running, /cycling, /automotive and /stationary — each zero or one, plus /tdlidar/activity/confidence from zero (low) to two (high), around once a second.

## Device

Battery streams /tdlidar/motion/battery/level from zero to one, and /tdlidar/motion/battery/state where zero is unknown, one is unplugged, two is charging and three is full.

Thermal State streams /tdlidar/device/thermal from zero (nominal) through one and two up to three (critical) — handy for degrading your render quality before the phone throttles itself.

Low Power Mode streams /tdlidar/device/lowpower as zero or one.

Screen Brightness streams /tdlidar/device/brightness from zero to one, so the phone's own brightness slider becomes a spare fader.

## Body and vision

Body is Apple's Vision 3D body pose from the back camera. It sends /tdlidar/body/detected as one or zero — when it is zero, no one is in frame and the skeleton values go stale. The skeleton arrives two ways: /tdlidar/body/skeleton carries fifty-one floats, which is seventeen joints each as x, y, z in camera-relative metres; and /tdlidar/body/skeleton/img carries thirty-four floats, the same seventeen joints as normalized x, y image coordinates with a top-left origin. Each joint also has its own address such as /tdlidar/body/head and /tdlidar/body/l_wrist carrying x, y, z and a confidence value, plus an image-space version at the same address with /img appended. Subject distance arrives at /tdlidar/body/distance in metres. The seventeen joints, in order, are root, spine, neck, head, head top, left shoulder, left elbow, left wrist, left hip, left knee, left ankle, right shoulder, right elbow, right wrist, right hip, right knee, right ankle. The image coordinates are accurate; the depth axis is noisy because it is estimated from a single camera.

Hands sends /tdlidar/body/hands/count for the number of hands seen (zero to two), /tdlidar/body/hands/distance in metres, and a per-frame heartbeat at /tdlidar/body/hands/frame that increments every frame so a receiver can tell the stream is live even when a hand is still. Each landmark is sent under /tdlidar/body/hands/left/ or /right/ with x and y as normalized image coordinates, z as depth in metres, and a unit direction for orienting bone geometry. There are twenty-one landmarks per hand in MediaPipe order from the wrist out through each finger. The side (left or right) is decided by where the wrist sits on screen, which stays stable as the hand turns.

Pinch sends a single float at /tdlidar/pinch — the thumb-tip to index-tip distance, normalized to palm size, near zero when pinched closed. It is the expressive air-fader.

Gesture sends four flags — /tdlidar/gesture/open, /fist, /peace and /point — each zero or one, for discrete triggers.

Face is ARKit from the front TrueDepth camera and sends fifty-two facial blendshapes, each zero to one, under /tdlidar/face along with the head transform and gaze direction.

6DoF Device Pose tracks the phone in the room and sends position at /tdlidar/arpose/tx, /ty and /tz in metres, an orientation quaternion at /tdlidar/arpose/quat, the same orientation as /tdlidar/arpose/euler/pitch, /roll and /yaw in radians, and a tracking-quality integer at /tdlidar/arpose/tracking. This is what drives a matching virtual camera or projection mapping.

ARKit Body is the true world-space, metric skeleton — far cleaner than the Vision body — with ninety-one joints at sixty hertz under /tdlidar/arbody, including an orientation quaternion and the joint hierarchy. It cannot run at the same time as the LiDAR depth session.

## Scene and detection

Camera Exposure is a light proxy (there is no public lux reading) and sends /tdlidar/cam/iso, /shutter_sec, /ev_offset, /focus, /wb_kelvin and /wb_tint.

Ambient Light sends room brightness at /tdlidar/light/ambient_lumens and colour temperature at /tdlidar/light/kelvin, so visuals can match the room.

AR Planes sends /tdlidar/scene/plane_count and /tdlidar/scene/plane_area_m2, a running measure of how much of the room has been mapped.

AR Mesh sends /tdlidar/scene/mesh_anchors and /tdlidar/scene/mesh_faces, a scan-completeness meter from the LiDAR mesh.

QR / Barcode sends the decoded payload as a string at /tdlidar/detect/qr/payload plus the detected corner coordinates.

Rectangle detection sends /tdlidar/detect/rect/count and the centre at /tdlidar/detect/rect/cx and /cy.

Text (OCR) sends /tdlidar/detect/text/count and the recognized text as a string at /tdlidar/detect/text/string.

Saliency sends the attention box — where the eye is drawn — as /tdlidar/saliency/x, /y, /w, /h and /conf, all normalized.

Front Camera Distance sends the nearest distance in front of the TrueDepth camera as a single normalized value at /tdlidar/frontdist/norm, zero to one, with the range adjustable in the app. It turns your hand into an air-fader.

Back Camera Distance sends the LiDAR distance at a movable on-screen cross at /tdlidar/backdist/meters and /tdlidar/backdist/norm, out to roughly twenty metres.

Animal tracking sends cat and dog keypoints — nose, neck, the four paws and the tail — each as x and y only (no depth), under /tdlidar/animal, plus /tdlidar/animal/distance when a depth camera is active.

## Audio

Mic Level sends a single microphone volume reading at /tdlidar/audio/rms.

Audio sends the full analysis: band levels at /tdlidar/audio/low, /mid and /high; a twenty-band FFT spectrum; transient triggers at /tdlidar/audio/beat and /onset; and pitch, note, MIDI number, spectral centroid, flux, rolloff and loudness channels under /tdlidar/audio.

Speech sends on-device transcription as strings at /tdlidar/speech/partial, /final and /paragraph.

Sound ID sends a label as a string at /tdlidar/sound/label naming what it hears — one of roughly three hundred classes such as clap, speech, music or an instrument — alongside a confidence float at /tdlidar/sound/confidence. When nothing is recognized the label reads as a placeholder.

## Touch and input

Touch sends the on-screen touch as /tdlidar/touch/x, /y, /radius and /force, with each channel toggleable in the app.

Apple Pencil sends position, pressure, tilt, azimuth and barrel-tap under /tdlidar/pencil (iPad with a Pencil only).

Proximity sends /tdlidar/proximity as zero (far) or one (near). It is a free physical trigger, though triggering it blanks the phone screen.

NFC sends a trigger float at /tdlidar/nfc/trigger and the tag's payload as a string at /tdlidar/nfc/payload.

Remote Control turns headset, media-button and hardware volume presses into triggers: /tdlidar/remote/playpause, /next, /previous, /volup and /voldown each fire momentarily (one then back to zero), and /tdlidar/remote/volume reports the level from zero to one, re-centring after each press so the buttons keep firing.

## External

AirPods sends head pose under /tdlidar/airpods as pitch, roll and yaw in radians, the same in degrees with a _deg suffix, plus a quaternion, acceleration, gravity and tilt.

Apple Watch sends heart rate, acceleration, rotation and the Digital Crown under /tdlidar/motion/watch, via the companion Watch app.
