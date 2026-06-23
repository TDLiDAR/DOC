---
title: Install
layout: default
nav_order: 2
---

# TDLiDAR — TouchDesigner Operator Family · v1.0.0

Turn an iPhone into **~34 live sensors** inside TouchDesigner — motion, LiDAR depth, body / hand / face tracking, audio, camera vision, touch and more — over your local network, with one drag‑and‑drop.

This package is the **TouchDesigner half**. Get the **TDLiDAR** app on the iOS App Store for the data source.

---

## Install (one time)

1. **Extract** this zip. Keep `TDLiDAR_family.tox` and the `operators/` folder **together**.
2. In TouchDesigner, **drag `TDLiDAR_family.tox`** into your project. It self‑installs on load — the **TDLiDAR** tab appears in the OP Create (TAB) menu.
3. If the operators don't appear: select the dropped **`TDLiDAR_fam`** component, set its **`Opfolder`** parameter to this package's **`operators/`** folder, pulse **`Ensuremanifests`**, then toggle **`Install`** On.
4. **Save your `.toe` project.**

> Zero‑config tip: if you extract `operators/` **next to your `.toe` file**, the default relative path resolves automatically — nothing to set.

---

## Use

1. Open the **TDLiDAR** app on your iPhone → point OSC at your computer's IP, **port 9000** → start streaming.
2. **TAB → TDLiDAR →** drop any operator. Each listens on OSC port 9000 and previews its live output on the node tile.
3. Each operator only wires to its own type (a CHOP op → CHOPs, a TOP op → TOPs, etc.), like native TD.

Troubleshooting "no data": same Wi‑Fi/LAN, correct IP, port 9000, sensor enabled in the app, firewall allows TouchDesigner. String sensors (Speech, OCR, Sound ID, NFC, QR payload) use an OSC In **DAT**.

---

## What's inside

| Item | What it is |
|---|---|
| `TDLiDAR_family.tox` | The family component — **drag this into TouchDesigner**. Self‑bootstraps the TDFam runtime. |
| `operators/` | The 34 operator `.tox` files + `manifest.json` (per‑op connect‑type metadata). |
| `docs/` | Full documentation: `README.md`, `OSC-REFERENCE.md` (complete wire spec), and a page per operator in `docs/ops/`. |

---

## Requirements

- **TouchDesigner** 2023.30000+ (the 3D operators use POPs).
- The **TDLiDAR** iOS app (App Store) streaming on the same network.
- LiDAR / TrueDepth / iPad+Pencil / Apple Watch are needed only by the operators that use them (each op's doc page lists its requirement).

Built on the dotsimulate **TDFam** operator‑family runtime (bundled inside `TDLiDAR_family.tox`).
