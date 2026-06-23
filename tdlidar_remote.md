---
title: "Volume"
parent: Operators
---
# Volume — `tdlidar_remote`

> Drive transport and step presets hands-free with the phone's volume buttons and any headset/media remote.

**Category:** Touch & Input · **Tier:** Free · **Needs:** any iPhone (optionally wired/Bluetooth headset with media buttons)

## What it does
Turns the iPhone's hardware volume buttons and the play/pause/next/previous controls on a headset or media remote into OSC events. The five button channels are **momentary** — they flash 1 then snap back to 0 on each press — while `volume` is the current system level 0–1. The clever bit: the app keeps re-centring the volume after each press, so the up/down buttons never hit the end-stop and keep firing forever. It's a free wireless clicker for running a TD show without touching the screen.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/remote/playpause` | float | momentary 1→0 | on press |
| `/tdlidar/remote/next` | float | momentary 1→0 | on press |
| `/tdlidar/remote/previous` | float | momentary 1→0 | on press |
| `/tdlidar/remote/volup` | float | momentary 1→0 | on press |
| `/tdlidar/remote/voldown` | float | momentary 1→0 | on press |
| `/tdlidar/remote/volume` | float | 0–1 (re-centres so buttons keep firing) | on change |

## Outputs
`out1` (CHOP) — six channels named after the addresses without the leading slash: `tdlidar/remote/playpause`, `…/next`, `…/previous`, `…/volup`, `…/voldown`, and `tdlidar/remote/volume`. The node tile previews `out1`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. In the TDLiDAR app, enable **Remote Control** (Volume).
2. Drop the **Volume** op. Press the phone's volume-up button — `tdlidar/remote/volup` blips to 1 and back.
3. Wire `tdlidar/remote/playpause` into a **Trigger CHOP** to start/stop your transport (a Timer, a Movie File In play toggle, a Beat reset).
4. Plug in earbuds with inline buttons and you've got a wireless remote — `next`/`previous` step your set from your pocket.

## Advanced patterns
- **Step a Switch / preset bank:** **Count CHOP** on `next` (and another, counting down, on `previous`) → feed the count into a **Switch TOP/SOP** *index* to walk through presets, slides or scenes hands-free. Set the Count's *Limit Max* + *wrap* to loop the bank.
- **Volume as a fader:** because `volume` re-centres, don't read it as an absolute level — instead take its **Slope CHOP** (or compare against its lagged self) so each up/down nudges a value by a delta. Great for a relative "trim" knob that never saturates.
- **Toggle from a momentary:** **Trigger CHOP** → **Count CHOP** (*Limit Max 1, wrap*) turns `playpause` into a latched on/off you can drive a Switch with.
- **De-bounce / hold:** a long press can repeat — a Trigger *Re-Trigger Delay* gives one event per intentional press.

## Gotchas
- The five buttons are **momentary 1→0** — a blink, not a held state. Always go through a Trigger/Count; you can't read them as a level.
- `volume` **deliberately re-centres** after each press so the buttons never hit min/max. That means its absolute value is meaningless — use it for *change* (deltas), not as a 0–1 fader position.
- iOS may also change the actual ringer volume as you press; that's harmless to the show but expected.
