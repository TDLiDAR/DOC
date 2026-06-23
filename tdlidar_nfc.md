---
title: "NFC"
parent: Operators
---
# NFC — `tdlidar_nfc`

> Tap a physical sticker or token to the phone to fire a named cue in TouchDesigner — props that recall scenes.

**Category:** Touch & Input · **Tier:** Free · **Needs:** any iPhone with NFC reading (iPhone 7+), an NFC tag

## What it does
Reads NFC tags (the same chips in transit cards and "tap to pay" stickers) and turns each tap into two things: a pulse on `trigger`, and the tag's stored text on `payload`. Pre-write a word like `verse` or `chorus` onto a cheap sticker, tap it to the phone, and TD both fires an event *and* receives the word — so you can recall a specific scene from a specific physical token. Great for letting a performer or audience trigger named looks with objects instead of a laptop.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/nfc/trigger` | float | momentary pulse (1 on tap) | on tap |
| `/tdlidar/nfc/payload` | **string** | the tag's stored text | on tap |

> `payload` is a **string** — it must be read with an **OSC In DAT**, not the CHOP.

## Outputs
- `out1` (CHOP) — one channel `tdlidar/nfc/trigger`, a brief pulse on each tap.
- `out_label` (Text DAT) — the latest tag `payload` string, fed from an internal **OSC In DAT**.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app, both CHOP and DAT) |

## Quick start (beginner)
1. Write a short word (e.g. `scene1`) to an NFC tag using any NFC writer app, and enable **NFC** in the TDLiDAR app.
2. Drop the **NFC** op. Tap the tag to the top-back of the phone.
3. `out1` fires a pulse; `out_label` now shows `scene1`. You've read the tag.
4. Wire `out1` into a **Trigger CHOP** to bang an event, and reference `out_label`'s text wherever you want the cue name (a Text TOP, a Select).

## Advanced patterns
- **Token → scene recall:** drive a **Switch TOP/SOP** by matching `out_label` against a table. A **DAT Execute** (or a Python `op('switch1').par.index = lookup[op('out_label').text]`) maps each payload string to a Switch index, so each physical tag jumps to its own look.
- **Cue list with payload + fire:** treat `trigger` as "go" and `payload` as "which" — fire transport on the pulse, set the target from the string, so one tap both selects and launches.
- **De-bounce:** a fast double-tap can fire twice. A **Trigger CHOP** with a short *Re-Trigger Delay*, or a **Timer**, swallows the second pulse.
- **Logging:** append each tap to a **Table DAT** (`payload` + `absTime.seconds`) to build a tap history for a generative or audience-driven set.

## Gotchas
- **Two transports:** the pulse is a CHOP channel but the payload is a **string** — you need the **OSC In DAT** (`out_label`) for the text. Reading payload off the CHOP gives you nothing.
- `trigger` is **momentary** — it pulses to 1 and back. Route it through a Trigger/Count; don't expect a held level.
- The payload is only ever **what you wrote to the tag** — blank tags read an empty string. Pre-program your tokens, and keep payload strings short and unique so your lookup table is clean.
