---
title: "Show"
parent: Operators
---
# Show — `tdlidar_show`

> The one operator in the family that *sends* instead of receives — a ready-made control panel for remote-triggering the phone from TouchDesigner, no hand-built OSC Out CHOP required.

**Category:** Output & Utility · **Tier:** Free · **Needs:** the phone's [Show Control]({{ '/show-control-mode.html' | relative_url }}) listener enabled, same LAN

## What it does
Every other operator in this family reads data the phone is sending. `tdlidar_show` is the mirror image: point it at the phone's IP and Show Control port, and its parameters become a control panel — recall a saved look, switch the app's mode, start/stop recording, toggle the NDI stream, set its output resolution, flip the alpha mask, trigger a Mesh Cloud or Point Cloud capture step, or ride a live gamma/contrast/brightness/threshold/colormap fader — each one just an OSC Out DAT send under the hood, wrapped in named parameters so you don't have to remember the `/tdlidar/show/*` addresses.

## OSC out
| address | args | sent when |
|---|---|---|
| `/tdlidar/show/recall` | int | **Recall** pulsed — sends **Recall Index** |
| `/tdlidar/show/mode` | string | **Switch Mode** pulsed — sends **Mode** (LiDAR / Monocular Depth / Point Cloud only) |
| `/tdlidar/show/record` | int (0/1) | **Record** toggled |
| `/tdlidar/show/ndi` | int (0/1) | **NDI Enable** toggled |
| `/tdlidar/show/resolution` | int (0–3) | **NDI Resolution** changed — Low / Med / High / Max |
| `/tdlidar/show/alpha` | int (0/1) | **Alpha Mask** toggled |
| `/tdlidar/show/capture/<verb>` | — | **Send Capture** pulsed — verb from **Capture Verb** |
| `/tdlidar/show/reset` | — | **Reset Settings** pulsed — factory-reset every setting |
| `/tdlidar/show/screen` | int (0/1) | **Screen Off** toggled — 1 blacks out the phone's screen (streams keep running), 0 wakes it |
| `/tdlidar/show/param/<key>` | float 0–1 (≥0.5 for toggles) | any of the **142** mode-tab controls changed — see below |

**v9.11 — every setting is a parameter.** Three parameter pages (**LiDAR** · 45, **Monocular Depth** · 14, **Point Cloud** · 83) expose every user-facing control of each mode as a `0–1` fader (or a toggle), each sending the matching `/tdlidar/show/param/<key>` live on change. All faders are **normalized 0–1** and scaled to the control's real range on the phone (matching a MIDI CC's 0–127); toggles send `1`/`0`. The full key list lives in [Show Control → Every setting over OSC & MIDI]({{ '/show-control-mode.html#every-setting-over-osc--midi' | relative_url }}).

(Matches [Show Control]({{ '/show-control-mode.html' | relative_url }}) exactly — this op is a thin wrapper, not a second spec.)

## Outputs
`out1` (CHOP) — mirrors the current parameter values (`Record`, `Gamma`, `Contrast`, `Brightness`, `Threshold`, `Colormapindex`, `Recallindex`) as channels, for chaining or an on-screen readout of what was last sent. Nothing flows *into* this op from the phone — it's send-only.

## Parameters
| par | default | what it does |
|---|---|---|
| Address | (empty) | the phone's IP — required |
| Port | 9200 | Show Control's listen port (match the phone's Remote Control settings) |
| Recall Index | 0 | which Look Preset to recall |
| Recall | — | pulse to send the recall |
| Mode | LiDAR | which mode to switch to — LiDAR / Monocular Depth / Point Cloud only (the utility modes aren't remote-switchable) |
| Switch Mode | — | pulse to send the mode switch |
| Record | off | on/off sends 1/0 immediately |
| Capture Verb | Mesh Finish | finish / rescan / start / stop |
| Send Capture | — | pulse to send the capture verb |
| Reset Settings | — | pulse to factory-reset every setting (same as the in-app Reset All) |
| Screen Off | off | blackout the phone's screen while NDI/OSC/recording keep running (triple-tap the phone to wake) |
| NDI Enable | off | persistent-NDI master on/off — sends immediately on change |
| NDI Resolution | Med | Low / Med / High / Max — drives all three NDI outputs at once, on change |
| Alpha Mask | off | alpha mask on/off across all three modes, on change |
| **LiDAR** page — 45 controls | 0–1 / toggles | every LiDAR setting as a normalized fader or toggle — send `/tdlidar/show/param/<key>` live on change |
| **Monocular Depth** page — 14 controls | 0–1 / toggles | every Monocular Depth setting, same scheme |
| **Point Cloud** page — 83 controls | 0–1 / toggles | every Point Cloud setting (view, tilt, cleanup, motion FX, network), same scheme |

## Quick start (beginner)
1. On the phone: Main menu → gear → **Show Control** → Enabled on. Note the IP and Listen Port shown there.
2. Drop **Show** in TD. Set **Address** to that IP, **Port** to match (default 9200).
3. Set **Recall Index** to `0` and pulse **Recall** — if you've saved a Look Preset on the phone, it applies immediately.
4. Export a Slider CHOP onto **Gamma** for a live fader, or wire a Button COMP's pulse into **Send Capture** to drive Mesh Cloud's Finish/Rescan remotely.

## Advanced patterns
- **Lighting-desk fader → depth cutoff.** DMX In CHOP → Select → export onto **Depth Threshold** — a physical fader now drives the LiDAR far-clip distance live.
- **QLab / MSC → look recall.** A Select DAT watching incoming OSC (or MSC translated to OSC) → drives **Recall Index** → pulses **Recall** via a Logic/Trigger CHOP on the incoming cue number.
- **Companion buttons.** Companion talks to the phone directly over its own Generic OSC device — `tdlidar_show` isn't in that path, but its parameter list is the same address vocabulary, useful as a reference for what to configure on Companion's side.
- **Bluetooth/Network MIDI instead.** Show Control also takes MIDI directly (Note On = recall, CC = live param) — `tdlidar_show` and MIDI are two independent ways to reach the same phone, use whichever fits your rig.

## Gotchas
- **Nothing sends until Address is set.** The op has no default host — an empty Address silently sends nowhere (no error).
- **This is one-way.** `tdlidar_show` doesn't confirm the phone received anything. Turn on the phone's **Status Heartbeat** (Show Control settings) and read it back with a normal OSC In CHOP on port 9000 if you want confirmation the phone is alive.
- **Send-only op in a receive-only family.** Every other operator's wire-drag menu offers a CHOP/POP/TOP because it *outputs* sensor data; `tdlidar_show`'s only real output is the `out1` monitor CHOP — the actual "output" is the OSC packet leaving the network, not anything in the TD graph.
- **Mesh Cloud Send/Export aren't reachable.** `Capture Verb` only covers `finish`/`rescan`/`start`/`stop` — Send (stream to TD) and Export (PLY) live inside Mesh Cloud's own 3D viewer and aren't remote-triggerable yet.
