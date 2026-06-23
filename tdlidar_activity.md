---
title: "Motion Activity"
parent: Operators
---
# Motion Activity — `tdlidar_activity`

> Let the visuals know if the performer is standing still, walking, or running — and switch presets automatically.

**Category:** Motion · **Tier:** Free · **Needs:** any iPhone (no LiDAR required)

## What it does
Uses the phone's on-device activity classifier to tell you, in plain terms, what the person carrying it is doing right now: walking, running, cycling, in a vehicle (automotive), or stationary. Each is a 0/1 flag, and a separate confidence channel (0=low, 1=medium, 2=high) says how sure the phone is. It updates roughly once a second, making it ideal for high-level context switching rather than fast control.

## OSC in
| address | type | range | rate |
|---|---|---|---|
| `/tdlidar/activity/walking` | float | 0/1 | ~1 Hz |
| `/tdlidar/activity/running` | float | 0/1 | ~1 Hz |
| `/tdlidar/activity/cycling` | float | 0/1 | ~1 Hz |
| `/tdlidar/activity/automotive` | float | 0/1 | ~1 Hz |
| `/tdlidar/activity/stationary` | float | 0/1 | ~1 Hz |
| `/tdlidar/activity/confidence` | float | 0=low, 1=med, 2=high | ~1 Hz |

## Outputs
`out1` (CHOP) — six channels: `walking`, `running`, `cycling`, `automotive`, `stationary`, `confidence`. The node tile previews `out1`.

## Parameters
| par | default | what it does |
|---|---|---|
| OSC Port | 9000 | UDP port to listen on (match the app) |

## Quick start (beginner)
1. Enable **Motion Activity** in the app (grant the Motion & Fitness permission when asked).
2. Drop the **Motion Activity** op. Sit still: `stationary` reads 1, the rest 0.
3. Walk around with the phone and `walking` flips to 1 a second or two later.
4. Wire the five state channels into a **Switch TOP** (via a small Math/CHOP-to-index) so each activity shows a different scene — instant, screen-free preset changes.

## Advanced patterns
- **State → index → preset:** Multiply each flag by its index in a **Math CHOP** and sum (a *Combine → Add*) to collapse the five one-hot channels into a single 0–4 index, then feed a **Switch TOP/CHOP** or a **Select** to pick the active look.
- **Confidence gate:** Use a **Logic CHOP** — only act when `confidence` ≥ 2 (high) — so a flickery low-confidence guess doesn't change your show. AND it with the state flag before the Switch.
- **Debounce the ~1 Hz jitter:** A **Trigger CHOP** with a minimum *on* time, or a Filter CHOP threshold, stops rapid walking/stationary flapping at the edge of a movement.
- **Energy mapping:** Map stationary→running onto a tempo or intensity ramp so the whole show breathes with the performer's pace.

## Gotchas
- **~1 Hz and laggy.** The classifier needs a few seconds of motion to commit; it is for *context*, not beat-accurate control. Don't drive anything time-critical from it.
- States can briefly overlap or all read 0 during transitions — always combine with `confidence` and a debounce before switching scenes.
- Requires the user to grant **Motion & Fitness** access in the app; with no permission the channels stay at 0.
