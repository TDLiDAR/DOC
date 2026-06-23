---
title: "QR / Barcode"
parent: Operators
---
# QR / Barcode — `tdlidar_qr`

> Point the phone at a QR code and have TouchDesigner react — trigger a scene, or open the decoded URL straight into a Web Render TOP.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** any camera (back recommended)

## What it does
This op watches the camera for QR codes and barcodes. When one is found it streams the decoded text (the **payload** — a URL, an ID, any string) plus the four screen corners of the code so you can frame or track it. The payload is text, so it arrives as a string. The shipped tox can route that payload straight into a **Web Render TOP**, so scanning a QR that holds a link opens that page live inside TD.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/detect/qr/payload` | string | decoded text (URL / id / any string) | on detect |
| `/tdlidar/detect/qr/*` (corners) | float | corner coordinates of the code | on detect |

> The **payload is a string** — it must be read with an **OSC In DAT**. The numeric corner channels come in on the **OSC In CHOP**.

## Outputs
- `out_label` (DAT) — the decoded `tdlidar/detect/qr/payload` string (latest scan).
- `out1` (CHOP) — the `tdlidar/detect/qr/*` corner coordinates.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the app, enable **QR / Barcode** and start streaming. Aim the camera at a QR code.
2. Drop the **QR / Barcode** op. Confirm `OSC Port` is 9000.
3. Open `out_label` (the DAT) — the decoded text appears the moment a code is read.
4. To act on it, feed the DAT into a **Web Render TOP** (set its URL from the payload cell) — scan a link, see the page render in TD.

## Advanced patterns
- **Scan-to-website.** Pipe `out_label` into a **Web Render TOP** URL parameter (via a DAT Execute or an expression). Each new QR swaps the page — instant interactive signage.
- **Scan-to-trigger.** Use a **DAT Execute** (or compare the payload cell in a Text DAT) to match known codes and pulse a Logic/Trigger — e.g. QR "SCENE_2" jumps your timeline.
- **Frame/track the code.** The corner floats on `out1` let you draw an outline (SOP/POP from the four corners) or position content over the physical code.
- **Debounce repeats.** The same code re-reads every frame it's visible; gate on payload *changes* (compare to last value in a DAT Execute) so you trigger once per new code, not continuously.

## Gotchas
- The **payload needs an OSC In DAT**, not a CHOP — strings never appear as channels. The corners are the only numeric part.
- The payload **persists/repeats** while the code stays in frame — debounce on change if you want a single trigger.
- A **Web Render TOP** will navigate to whatever the QR contains; in a public show, validate/whitelist payloads before opening them.
- Detection is **on-detect**, only while a readable code is actually in view — no code, no update (the last value lingers).
