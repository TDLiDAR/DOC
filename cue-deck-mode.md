---
title: Cue Deck Mode
layout: default
parent: App Guide
nav_order: 10
---

# Cue Deck Mode

Cue Deck turns the phone into a bank of large, physical-feeling trigger pads for firing cues into TouchDesigner by hand — no AR, no camera, no sensors. It's the app's answer to a TouchOSC control panel, but it reuses TDLiDAR's own always-on OSC connection instead of a second app or a separate config file.

## The pads

Cue Deck starts with **4 pads**, laid out edge-to-edge across the screen in a 2-column grid so each one is a large, easy target — deliberately not a dense 16-button grid. A **+** button adds more pads (extra pads add rows below the first four), and a gear icon opens the full settings editor.

Pressing a pad sends **1** to its OSC address; releasing sends **0** — a clean momentary gate, and a firm haptic tick fires on press so it feels like a real button, not a flat glass tap. Pressing again too soon after a release is silently ignored for that pad's configured lockout window, and the pad's border flashes red to show the tap was swallowed, so accidental double-fires from a nervous hand don't send extra triggers.

## Editing pads

Tap the gear to open Cue Deck Settings. Each pad in the list can be tapped to edit:

- **Name** — the label shown on the pad.
- **Address** — the OSC address it fires (defaults to `/tdlidar/cue/<n>`, but any address works).
- **Colour** — picked from a native colour wheel; purely cosmetic, shown only on the phone.
- **Time between taps** — the re-trigger lockout, adjustable from no lockout up to several seconds.

Pads can also be deleted from the same list, and the **+** button in Settings adds another with the next default name/address/colour.

## Network settings

The same Settings sheet has the **TouchDesigner OSC** section — target IP and port. Cue Deck defaults to **port 9000**, the same bus the Sensors mode sensors use, so it can share one OSC connection rather than needing a dedicated port.

## What you can do with it

Use it as a manual scene-advance button, a blackout switch, a "go" cue for a stage manager, or any trigger that needs to be reliable and reachable from across the room without looking at a laptop. Because every pad is just an OSC address firing 1/0, it drops straight into whatever trigger/logic chain you already have in TouchDesigner.

## Needs

Nothing beyond the app's normal network setup — Cue Deck works on any iPhone, with no LiDAR, TrueDepth or AR requirement.
