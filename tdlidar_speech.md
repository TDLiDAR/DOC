---
title: "Speech"
parent: Operators
---
# Speech — `tdlidar_speech`

> Live captions on a Text TOP, plus keyword triggers — the phone listens and TouchDesigner reads the words.

**Category:** Audio · **Tier:** Free · **Needs:** microphone + speech recognition on the phone

## What it does
Streams the phone's on-device speech-to-text into TouchDesigner as **text**. You get the rolling `partial` (what's being said right now, updating live), `final` (each completed utterance), and `paragraph` (the accumulated transcript). Use it for live subtitles on screen, or to fire events when a spoken keyword shows up.

Because this is a **string** payload, it must be read with an **OSC In DAT** — an OSC In CHOP cannot carry text. A callback on the DAT writes the latest words into a Text DAT (`out_label`).

## OSC in

| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/speech/partial` | string | live in-progress text | on change |
| `/tdlidar/speech/final` | string | completed utterance | per utterance |
| `/tdlidar/speech/paragraph` | string | accumulated transcript | on change |

> Speech transcript text can ride a second UDP port if you configure one in the app; point this op's **OSC Port** at whichever port the app is sending speech on.

## Outputs
- `out_label` (Text DAT) — the latest speech string, written by the OSC In DAT's callback. Wire it straight into a **Text TOP** for on-screen captions.

## Parameters

| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on — match the app's speech port (may differ from the main 9000 feed) |

## Quick start (beginner)
1. Enable Speech in the app and grant microphone + speech-recognition access.
2. Drop `tdlidar_speech`. Inside it is an **OSC In DAT** whose callback writes incoming text into the `out_label` Text DAT.
3. Speak near the phone — the Text DAT updates as you talk.
4. Wire `out_label` into a **Text TOP** and you have live captions on screen.

## Advanced patterns
- **Partial vs final captions:** drive the on-screen Text TOP from `partial` for buttery live-updating subtitles; log `final` lines into a Table DAT for a scrolling transcript that never rewrites itself.
- **Keyword triggers:** in the OSC In DAT callback (or a downstream **Select DAT** / DAT Execute), test the incoming string for a word — when it matches, pulse a trigger (e.g. set a Constant CHOP, fire a Chop Execute) to change scenes on a spoken cue.
- **Word/line count → motion:** an Evaluate or Convert DAT can turn the transcript length into a number you bring back into CHOP-land to push type animation as more is said.
- **Typewriter reveal:** feed `partial` through a DAT that exposes character count, then animate a Text TOP's visible-length for a per-character reveal synced to speech.

## Gotchas
- **String, so OSC In DAT only** — wiring `/tdlidar/speech/*` to an OSC In CHOP gives you nothing. This op already uses the DAT; keep it.
- `partial` rewrites itself constantly while you speak (it's the *current guess*); only `final` is settled. Caption from `partial`, log from `final`.
- Speech may arrive on a **different port** than the rest of the sensors (TD's OSC In DAT and OSC In CHOP can clash on one port). Set **OSC Port** to whatever the app uses for speech.
- Recognition is on-device and best-effort — expect occasional wrong words and a short latency; don't gate a hard cue on a single rare keyword.
