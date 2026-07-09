---
title: OSC Reference
layout: default
nav_order: 5
---

# TDLiDAR — OSC Wire Reference

The canonical list of every OSC address the **TDLiDAR** iOS app emits, grouped by sensor. This is the source of truth the operator family is built against. All messages are sent over **UDP**, default **port 9000**.

## How the wire works

- Every address is prefixed `/tdlidar/…`.
- In TouchDesigner an **OSC In CHOP** turns each numeric address into a channel named by the address **without the leading slash** (`/tdlidar/motion/accel/x` → channel `tdlidar/motion/accel/x`).
- **String** payloads (labels, OCR text, NFC payloads, speech) must be read with an **OSC In DAT**, not a CHOP.
- A sensor only streams when it is **enabled in the app**. Several sensors share one address bucket (Body/Hands/Pinch/Gesture all live under `/tdlidar/body/…`); enabling more than one just adds channels.
- Rates below are typical on an iPhone 15 Pro; thermal throttling lowers them.

---

## Motion

| Sensor | Address | Args | Units / range | Rate |
|---|---|---|---|---|
| Acceleration | `/tdlidar/motion/accel/{x,y,z}` | float | G, gravity removed (userAcceleration) | ≤50 Hz |
| Gravity | `/tdlidar/motion/gravity/{x,y,z}` | float | unit vector, points to ground | ≤50 Hz |
| Gyro | `/tdlidar/motion/rotation/{x,y,z}` | float | rad/s (rotationRate) | ≤50 Hz |
| Quaternion / Euler | `/tdlidar/motion/{pitch,roll,yaw}` | float | radians | ≤50 Hz |
| Magnetometer | `/tdlidar/motion/magnetic/{x,y,z}` | float | µT (raw field, not heading) | ≤50 Hz |
| Barometer | `/tdlidar/motion/altitude/{relative,pressure}` | float | metres (relative), kPa | ~1 Hz |
| Motion Activity | `/tdlidar/activity/{walking,running,cycling,automotive,stationary}` | float | 0/1 each | ~1 Hz |
| Motion Activity | `/tdlidar/activity/confidence` | float | 0=low,1=med,2=high | ~1 Hz |

> Calibration note: pitch/roll/yaw and rotation are zeroed against a snapshot when the app's "calibrate" is pressed; gravity/magnetic stay raw.

## Device

| Sensor | Address | Args | Units / range | Rate |
|---|---|---|---|---|
| Battery | `/tdlidar/motion/battery/level` | float | 0–1 | on change + ~2 Hz |
| Battery | `/tdlidar/motion/battery/state` | float | 0 unknown,1 unplugged,2 charging,3 full | on change |
| Thermal State | `/tdlidar/device/thermal` | float | 0 nominal,1 fair,2 serious,3 critical | on change + 2 Hz |
| Low Power Mode | `/tdlidar/device/lowpower` | float | 0/1 | on change + 2 Hz |
| Screen Brightness | `/tdlidar/device/brightness` | float | 0–1 | 2 Hz |

## Camera & Vision — pose

### Body (Vision 3D, back camera)

| Address | Args | Meaning |
|---|---|---|
| `/tdlidar/body/detected` | float | 1 when a body is in frame, else 0 (skeleton channels go stale at 0) |
| `/tdlidar/body/skeleton` | 51 float | 17 joints × (x,y,z), **camera-relative metres** |
| `/tdlidar/body/skeleton/img` | 34 float | 17 joints × (x,y), normalized image coords, top-left origin |
| `/tdlidar/body/<joint>` | x,y,z,conf | per-joint, metres + confidence |
| `/tdlidar/body/<joint>/img` | x,y | per-joint normalized image |
| `/tdlidar/body/distance` | float | subject distance, metres |

Joint order (index → name): 0 root, 1 spine, 2 neck, 3 head, 4 head_top, 5 l_shoulder, 6 l_elbow, 7 l_wrist, 8 l_hip, 9 l_knee, 10 l_ankle, 11 r_shoulder, 12 r_elbow, 13 r_wrist, 14 r_hip, 15 r_knee, 16 r_ankle.
Bones: (0,1)(1,2)(2,3)(3,4)(2,5)(5,6)(6,7)(2,11)(11,12)(12,13)(0,8)(8,9)(9,10)(0,14)(14,15)(15,16)(5,11 shoulder-bar)(8,14 hip-bar).
> X/Y are accurate; Z (depth) is noisy on monocular Vision — the Body tox drives shape from `…/skeleton/img` and clamps Z per-bone.

### Hands (Vision, back camera — shares the Body pass)

| Address | Args | Meaning |
|---|---|---|
| `/tdlidar/body/hands/count` | int | number of hands (0–2) |
| `/tdlidar/body/hands/distance` | float | hand distance, metres |
| `/tdlidar/body/hands/frame` | int | per-frame heartbeat (increments every frame so TD knows data is live) |
| `/tdlidar/body/hands/<side>/<landmark>` | x,y,z + rx,ry,rz | 21 landmarks × (x,y normalized image, z metres) + unit bone direction |

21 landmark order: wrist, thumb_cmc, thumb_mcp, thumb_ip, thumb_tip, index_mcp, index_pip, index_dip, index_tip, middle_mcp, middle_pip, middle_dip, middle_tip, ring_mcp, ring_pip, ring_dip, ring_tip, pinky_mcp, pinky_pip, pinky_dip, pinky_tip.

### Pinch / Gesture (hand-derived, first hand, no side)

| Address | Args | Meaning |
|---|---|---|
| `/tdlidar/pinch` | float | thumbTip↔indexTip distance, palm-normalized (~0 closed) |
| `/tdlidar/gesture/{open,fist,peace,point}` | float | 0/1 each, one-hot-ish static gesture |

### Face (ARKit, front TrueDepth)

| Address | Args | Meaning |
|---|---|---|
| `/tdlidar/face/*` | float | 52 ARKit blendshapes (0–1), head transform, gaze/look-at |

### 6DoF Device Pose (ARKit world)

| Address | Args | Meaning |
|---|---|---|
| `/tdlidar/arpose/{tx,ty,tz}` | float | world position, metres |
| `/tdlidar/arpose/quat` | 4 float | orientation quaternion (x,y,z,w) |
| `/tdlidar/arpose/euler/{pitch,roll,yaw}` | float | radians |
| `/tdlidar/arpose/tracking` | int | ARKit tracking state (0 not available,1 limited,2 normal) |

### ARKit Body (back, exclusive)

| Address | Args | Meaning |
|---|---|---|
| `/tdlidar/arbody/skeleton…` | float | true 91-joint, world-space, metric skeleton, 60 Hz |
| `/tdlidar/arbody/orientation` | 4 float | body orientation quaternion |
| `/tdlidar/arbody/hierarchy` | — | joint parent indices for the rig |

## Camera & Vision — scene / detect

| Sensor | Address | Args | Meaning |
|---|---|---|---|
| Camera Exposure | `/tdlidar/cam/{iso,shutter_sec,ev_offset,focus,wb_kelvin,wb_tint}` | float | light proxy: ISO, shutter seconds, EV, focus 0–1, white-balance K + tint |
| Ambient Light | `/tdlidar/light/{ambient_lumens,kelvin}` | float | room brightness (lumens) + colour temperature (K) |
| AR Planes | `/tdlidar/scene/{plane_count,plane_area_m2}` | float | detected plane count + total area |
| AR Mesh | `/tdlidar/scene/{mesh_anchors,mesh_faces}` | float | LiDAR mesh totals (scan completeness) |
| QR / Barcode | `/tdlidar/detect/qr/payload` (string) + `/tdlidar/detect/qr/*` corners | str/float | decoded payload + corner coords |
| Rectangle (detect) | `/tdlidar/detect/rect/{count,cx,cy}` | float | detected quad count + centre |
| Text (OCR) | `/tdlidar/detect/text/count` (float) + `/tdlidar/detect/text/string` (string) | float/str | recognized strings |
| Saliency | `/tdlidar/saliency/{x,y,w,h,conf}` | float | attention box (normalized, TD flips Y) |
| Front Distance | `/tdlidar/frontdist/norm` | float | 0–1 normalized nearest distance (front TrueDepth) |
| Back Distance | `/tdlidar/backdist/{meters,norm}` | float | LiDAR distance at the movable cross |
| Animal | `/tdlidar/animal/{nose,neck,frontpaw/left,frontpaw/right,backpaw/left,backpaw/right,tail/…}` | x,y | 2D keypoints (no Z) |
| Animal | `/tdlidar/animal/distance` | float | distance when a depth camera is active |

> Scene Classification (`/tdlidar/scene/label` + `/confidence`) exists in the app but its tox is **deferred** (the prototype TOP chain crashed TD on load; rebuild with a Text TOP, no Movie File I/O).

## Audio

| Sensor | Address | Args | Meaning |
|---|---|---|---|
| Mic Level | `/tdlidar/audio/rms` | float | mic RMS volume |
| Audio | `/tdlidar/audio/{low,mid,high}` | float | band levels |
| Audio | `/tdlidar/audio/fft…` | float | 20-band FFT spectrum |
| Audio | `/tdlidar/audio/{beat,onset}` | float | transient triggers |
| Audio | `/tdlidar/audio/{pitch,note,midi,centroid,flux,rolloff,loudness}` | float | pitch/timbre channels |
| Speech | `/tdlidar/speech/{partial,final,paragraph}` | string | live STT (read with OSC In DAT) |
| Sound ID | `/tdlidar/sound/label` (string) + `/tdlidar/sound/confidence` (float) | str/float | 300+ class sound classification |

> The legacy `/tdlidar/audio/drums/{low,mid,high}` channels were removed (they never changed).

## Touch & Input

| Sensor | Address | Args | Meaning |
|---|---|---|---|
| Touch | `/tdlidar/touch/{x,y,radius,force}` | float | on-screen touch |
| Apple Pencil | `/tdlidar/pencil/{x,y,pressure,tilt,azimuth,barreltap}` | float | iPad + Pencil only |
| Proximity | `/tdlidar/proximity` | float | 0 far / 1 near (blanks the screen when near) |
| NFC | `/tdlidar/nfc/trigger` (float) + `/tdlidar/nfc/payload` (string) | float/str | tag read |
| Remote Control | `/tdlidar/remote/{playpause,next,previous,volup,voldown}` | float | momentary 1→0 |
| Remote Control | `/tdlidar/remote/volume` | float | 0–1 (re-centres so buttons keep firing) |

## External

| Sensor | Address | Args | Meaning |
|---|---|---|---|
| AirPods | `/tdlidar/airpods/{pitch,roll,yaw}` + `…_deg` | float | head pose (rad + degrees) |
| AirPods | `/tdlidar/airpods/{quat,accel,gravity,tilt}` | float | quaternion + motion |
| Apple Watch | `/tdlidar/motion/watch/*` | float | heart rate, accel, rotation, Digital Crown (needs the Watch app) |

## Output / utility toxes (not OSC sensors)

| Tox | Transport | Meaning |
|---|---|---|
| NDI | NDI video | receive the phone's NDI depth/RGB stream → TOP (source-name dropdown) |
| Point Cloud | TCP | 50k bit-exact XYZ+RGB points → POP |
| Scene Build | — | RoomPlan scan review |
| Rectangle | OSC-driven | a TOP + POP whose size follows any chosen channel (demo / meter) |
