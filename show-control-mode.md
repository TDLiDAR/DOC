---
title: Show Control
layout: default
nav_order: 8
---

# Show Control

Show Control is the reverse of everything else in this app: instead of the phone sending data out, it **listens** for commands and reacts. Turn it on and any system that already speaks OSC or MIDI in a show rig — a lighting desk, QLab, MIDI Show Control, Bitfocus Companion / Stream Deck, or a DAW's MIDI clock — can switch the phone's mode, recall a saved look, start/stop recording, drive a live parameter, or trigger a capture step, all without anyone touching the device.

It is not a mode you pick in the mode switcher. It runs independently of whichever mode is active, so a remote mode-switch command works even from the main menu.

## Turning it on

Open the **gear icon** on the main menu (the same "Network" sheet that sets the TouchDesigner/OSC destinations) and find the **Show Control** section:

- **Enabled** — starts/stops the listener.
- **Listen Port** — UDP port Show Control listens on. Default **9200**, separate from the sensor OSC bus (9000) so the two never collide.
- **Status Heartbeat** — when on, the phone sends its own state back out once a second (see [Status heartbeat](#status-heartbeat-out)).
- **MIDI** — shows the assembled MIDI timecode (if any is arriving) and a **Pair Bluetooth MIDI device…** button that opens Apple's own Bluetooth MIDI picker. Network MIDI (Wi-Fi) needs no pairing — any Network MIDI session on the same network connects automatically once Show Control is enabled.

## OSC in — `/tdlidar/show/*`

All UDP, to whatever port Show Control is listening on (default 9200).

| address | args | does |
|---|---|---|
| `/tdlidar/show/recall` | int | Recalls a saved [Look Preset](#look-presets) by its index in the list. |
| `/tdlidar/show/mode` | string or int | Switches the app's mode. String matches a mode's internal name (`ndi`, `osc`, `pointCloud`, `sceneBuild`, `monocularDepth`, `meshCloud`, `cueDeck`, `align`); int indexes the same list in that order. |
| `/tdlidar/show/record` | int/float/bool (0 or 1) | Starts or stops recording in LiDAR or Monocular Depth mode — whichever is currently active. No-op in other modes. |
| `/tdlidar/show/capture/finish` | — | Mesh Cloud: ends the current scan (same as tapping Finish). |
| `/tdlidar/show/capture/rescan` | — | Mesh Cloud: discards the finished scan and starts over. |
| `/tdlidar/show/capture/start` | — | Point Cloud: starts TCP streaming. |
| `/tdlidar/show/capture/stop` | — | Point Cloud: stops TCP streaming. |
| `/tdlidar/show/param/gamma` | float | Live gamma, both LiDAR and Monocular Depth. |
| `/tdlidar/show/param/contrast` | float | Live contrast, both modes. |
| `/tdlidar/show/param/brightness` | float | Live brightness, both modes. |
| `/tdlidar/show/param/threshold` | float, 0–10 | LiDAR far-clip distance in metres — the "lighting-desk fader drives a depth cutoff" pattern. |
| `/tdlidar/show/param/colorMapIndex` | int | Selects a colormap by its index in the app's colormap list, both modes. |

> Mesh Cloud's Send (stream to TD) and Export (save PLY) aren't remote-triggerable yet — they live as private state inside the mode's own 3D viewer, not reachable from outside it. Finish and Rescan are.
{: .note }

## From TouchDesigner — the `tdlidar_show` operator

You don't have to hand-build an OSC Out CHOP for every address above — **`tdlidar_show`** in the operator family is a ready-made control panel for exactly this. Drop it, set **Address** to the phone's IP and **Port** to its Show Control port, and every address in the table above is a parameter:

| parameter page | pars | sends |
|---|---|---|
| Network | Address, Port | (where everything below is sent) |
| Cues | Recall Index + **Recall** pulse | `/tdlidar/show/recall` |
| Cues | Mode + **Switch Mode** pulse | `/tdlidar/show/mode` |
| Cues | Record toggle | `/tdlidar/show/record`, on change |
| Cues | Capture Verb + **Send Capture** pulse | `/tdlidar/show/capture/<verb>` |
| Live | Gamma, Contrast, Brightness, Depth Threshold, Colormap Index | the matching `/tdlidar/show/param/*`, live on change |

`out1` mirrors the current parameter values as a CHOP, in case you want to chain or display what was last sent. It's a thin wrapper — internally just an OSC Out DAT and a Parameter Execute DAT — so the DMX/Art-Net/QLab/Companion patterns below can drive `tdlidar_show`'s own parameters (export a CHOP into **Recall Index**, pulse **Recall**) instead of building a raw OSC Out CHOP by hand, if you'd rather stay in parameter-land.

## Look Presets

A Look Preset is a saved snapshot of LiDAR or Monocular Depth's colormap, tone curve (gamma/contrast/brightness/invert) and mode-specific extras (LiDAR: depth mode + near/far clip; Monocular Depth: model + camera). Build the list in the same Show Control section: **Save LiDAR look** / **Save Monocular look** snapshots whatever that mode is currently set to; tap a saved preset to rename it or re-save it from the mode's current live settings.

Recall one from outside the app with `/tdlidar/show/recall <index>` (index = its position in the list, 0-based) or a MIDI Note On (see below) — the same recall path either way.

## MIDI in

**Network MIDI** (Wi-Fi, `MIDINetworkSession`) is always available once Show Control is enabled — connect to it the same way you'd connect to any Network MIDI session (macOS Audio MIDI Setup → MIDI Studio → Network, or any rtpMIDI-compatible app), no pairing step on the phone.

**Bluetooth MIDI** needs one-time pairing: tap **Pair Bluetooth MIDI device…** in the Show Control section, which opens Apple's standard Bluetooth MIDI picker. Once paired, the device behaves exactly like a Network MIDI source.

Either transport decodes the same two message types:

- **Note On** (velocity > 0) — recalls a Look Preset by note number, same as `/tdlidar/show/recall`.
- **Control Change** — drives a live param, mapped through an editable CC list. Ships with a default mapping so a generic MIDI controller works immediately:

| CC | drives |
|---|---|
| 1 | gamma |
| 2 | contrast |
| 3 | brightness |
| 4 | threshold |

**MIDI Time Code** (MTC) quarter-frames are assembled into a running `HH:MM:SS:FF` timecode, shown live in the Show Control section — useful for confirming the phone is receiving a show's clock. Show Control does not yet fire commands at specific timecodes (that's a real, separate cue-scheduler feature — a timecode-indexed cue list with its own editor — not built yet); today, MTC reception is for monitoring, and cueing is done via `/tdlidar/show/recall` or MIDI Note On.

## Status heartbeat out

With **Status Heartbeat** on, the phone sends its own state back out once a second, over the app's normal OSC destination (the same IP set under "OSC destination" in the Network sheet), on port 9000:

| address | args | meaning |
|---|---|---|
| `/tdlidar/show/status/battery` | float | battery level, 0–1 |
| `/tdlidar/show/status/thermal` | int | 0 nominal, 1 fair, 2 serious, 3 critical |
| `/tdlidar/show/status/depthMode` | string | LiDAR mode's current depth mode |

Useful for a Companion or QLab dashboard that wants to show the phone is alive and not overheating, without polling it.

## Bridging from a lighting desk, QLab, MIDI Show Control or Companion

Show Control speaks OSC and MIDI directly — none of the following need anything built into the phone, they're patterns for the *other* end.

**DMX / Art-Net from a lighting desk.** In TouchDesigner: a **DMX In CHOP** or **Art-Net In CHOP** reads the universe, a **Select CHOP** isolates the channel you want to use as a trigger, and an **OSC Out CHOP** sends `/tdlidar/show/recall` (or `/tdlidar/show/param/threshold`, for a fader) to the phone's IP and Show Control port. This is a TD patch pattern, not a `.tox` — build it in your own network next to whatever else reads that universe.

**QLab / MIDI Show Control cues.** QLab's OSC cues (or an MSC cue relayed through TD) can send `/tdlidar/show/recall <n>` directly to the phone — no TD in the path at all if QLab and the phone are on the same network. Point a QLab Network cue at the phone's IP and Show Control port.

**Bitfocus Companion / Stream Deck.** Add the phone as a **Generic OSC** device in Companion (Connections → Generic OSC): host = the phone's IP, port = the Show Control listen port. Then any button can send `/tdlidar/show/recall`, `/tdlidar/show/mode`, `/tdlidar/show/record` or `/tdlidar/show/capture/*` as its action, exactly like the addresses above. There is no dedicated TDLidar Companion module — the Generic OSC device covers everything Show Control exposes without one.

## Needs

Nothing beyond the app's normal network setup for OSC and Network MIDI. Bluetooth MIDI needs a one-time pairing via the button above. Works on any iPhone.
