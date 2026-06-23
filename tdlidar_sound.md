---
title: "Sound ID"
parent: Operators
---
# Sound ID — `tdlidar_sound`

> Semantic audio triggers — react to a *clap*, *music*, *applause* or *speech* by name, going beyond a raw beat.

**Category:** Audio · **Tier:** Free · **Needs:** microphone access on the phone

## What it does
Classifies what the phone's microphone is hearing into one of 300+ everyday sound classes and streams the winning **label** (e.g. "clapping", "music", "speech", "dog") plus a **confidence** score. Where the Audio op only knows *how loud* and *what frequency*, Sound ID knows *what the sound is*, so you can fire visuals on meaning — applause launches confetti, music switches to a beat-reactive scene, silence falls back to ambient.

The label is a **string**, so it rides an **OSC In DAT**; the confidence is a number on a small CHOP.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/sound/label` | string | one of 300+ class names | on change |
| `/tdlidar/sound/confidence` | float | 0–1 | on change |

## Outputs
- `out_label` (Text DAT) — the current sound class name, written by the OSC In DAT's callback.
- `out1` (CHOP) — one channel: `tdlidar/sound/confidence` (0–1), how sure the classifier is.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable Sound ID in the app and grant microphone access.
2. Drop `tdlidar_sound`. The **OSC In DAT** callback writes the label into `out_label`; `out1` carries the confidence.
3. Clap, play music, or talk near the phone — `out_label` changes to name it and `out1` shows how confident it is.
4. Wire `out_label` into a **Text TOP** to display the live sound name.

## Advanced patterns
- **Semantic trigger (the point):** in the OSC In DAT callback (or a downstream **Select DAT** filtering by label), test for a class — `clapping`, `music`, `cheering` — and pulse a trigger when it matches. Gate it on `out1` (confidence) with a **Logic CHOP** *greater than ~0.5* so a faint mistaken guess doesn't fire your show.
- **Confidence as intensity:** map `out1` through a Math/Range CHOP and use it as the *strength* of the effect the label chose — a confident "applause" lands bigger than a hesitant one.
- **Debounce flapping:** the label can wobble between similar classes. Hold the last confident label with a DAT Execute (only overwrite when confidence clears the threshold), or require the same label for N updates before acting.
- **Scene routing:** build a small Lookup/Select DAT mapping class names → scene indices, convert to a CHOP value, and drive a Switch TOP so the room's sound picks the look.

## Gotchas
- **Two different operators:** the label is a **string on an OSC In DAT**; the confidence is a **float on a CHOP**. Don't try to read the label from a CHOP — it won't appear.
- Always gate on `out1` confidence. The classifier always emits *some* label; a low-confidence guess is noise, not a cue.
- Labels change on the fly and can chatter between near-neighbours (e.g. "speech" ↔ "conversation"); debounce before firing anything destructive like a scene cut.
- It's a microphone in a real room — overlapping sounds and bleed will confuse it; design cues around clear, dominant events (a big clap, sustained music), not subtle ones.
