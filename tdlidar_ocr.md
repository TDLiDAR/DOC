---
title: "OCR"
parent: Operators
---
# OCR — `tdlidar_ocr`

> Read signage, labels or handwriting through the camera and pour the live text straight into TouchDesigner as triggers and on-screen type.

**Category:** Camera & Vision · **Tier:** Free · **Needs:** any iPhone (rear camera)

## What it does
Runs on-device text recognition on the camera feed and sends every word it reads into TD as a string. The op keeps a running table of recognized words, renders them onto a "paper" Text TOP you can show on screen, and reports how many strings are currently in view. Point the phone at a poster, a price tag, a name badge or a page and the words appear in TouchDesigner — ready to match against keywords and fire scenes.

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/detect/text/count` | float | number of recognized strings | camera rate |
| `/tdlidar/detect/text/string` | string | the recognized text (read with an **OSC In DAT**) | on read |

## Outputs
- `out1` (CHOP) — one channel `tdlidar/detect/text/count` (how many strings are visible right now).
- `out_words` (DAT) — the running words table, newest reads appended. This is the table the "paper" Text TOP draws from.
- the **paper Text TOP** — recognized words composited onto a paper background for direct display.
- a **Clear Text** pulse parameter — empties `out_words` and the paper TOP.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |
| Clear Text | (pulse) | wipes the words table and the paper Text TOP |

## Quick start (beginner)
1. In the app, enable **OCR / Text** (Camera & Vision).
2. Drop the **OCR** op and point the rear camera at some clear printed text.
3. Watch `out_words` fill in and the paper Text TOP show the words; `out1` reports how many strings are in frame.
4. Press **Clear Text** to reset between takes. Composite the paper Text TOP into your output to put live captured type on screen.

## Advanced patterns
- **Keyword triggers:** use a small Python/Expression CHOP or a Match DAT against `out_words` to fire a bang when a specific word appears (e.g. the word "START" launches a cue). Pair with `count` to know when *any* text is present.
- **Live lower-thirds:** route `out_words` (or the latest row) into your own Text TOP with custom font/colour instead of the built-in paper look, for branded on-screen captions.
- **Gate on text presence:** **Logic CHOP** on `out1` (≥ 1) enables a reveal animation only while text is being read.
- **Throttle re-reads:** because reads are de-duped (see Gotchas), each distinct phrase lands once — good for treating an incoming word as a discrete event rather than a per-frame stream.

## Gotchas
- The string lives on `/tdlidar/detect/text/string` and **must be read with an OSC In DAT** — an OSC In CHOP will silently ignore it.
- The op **de-dupes repeated reads**: holding the camera on the same sign won't spam the same word every frame. If you need a re-fire, move the camera or press **Clear Text** to reset state.
- `out1` is a count, not the text itself — use it as a gate, read actual words from `out_words` / the DAT.
- OCR wants steady, well-lit, reasonably large text. Motion blur and tiny fonts drop the read rate.
