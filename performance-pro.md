---
title: Performance, Pro & Recording
layout: default
parent: App Guide
nav_order: 13
---

# Performance, Pro and Recording

These settings cut across every mode: how to keep a long session stable, what a Pro purchase unlocks, and how recording works.

## Performance and device

**Supercharge** doubles the size of the internal NDI buffer pools, using roughly two hundred megabytes more memory, so a long streaming session doesn't drop frames when the system gets busy. It takes effect after you restart the app. Turn it on for installations and long performances; leave it off on older phones or when memory is tight.

**Keep Screen On** stops the phone from sleeping while it is streaming, so a tripod-mounted phone doesn't doze off mid-show. **Keep Alive in Background** keeps the streams running when the app is moved to the background, so a notification or a glance at another app doesn't kill the feed.

There is also an option to **skip the camera-switch warning**. Normally, flipping the camera mid-recording asks you to confirm so you don't lose the clip by accident; with this on it silently saves the current clip and switches without the prompt, which is handy once you trust the workflow.

A word on heat. The phone's frame rate is the honest signal — if the frame-rate pill in LiDAR mode sits below your target, the phone is thermally throttling. The cures, in order: drop the output resolution, turn off Supercharge if you don't need it, lower the target frame rate, and give the phone air. A phone in a sealed mount in a warm room will throttle no matter what the software does.

## Pro features

A Pro purchase unlocks the heavier features: **Extended LiDAR Range** (the ten and twelve metre options on the depth range), **HD output resolution** with edge-preserving smoothing, the **depth mask alpha** key that makes the background transparent, and a number of the more advanced sensors. Pro state is cached on the device and reconciled with the App Store each time the app launches, so once you have unlocked it the features keep working even offline, and the app quietly corrects itself the next time it can reach the store.

## Recording

Any stream can be recorded to the phone and saved to your Photos library. A recording captures the same processed output that is going out live — the colour-mapped depth, or the RGB camera — not the raw sensor data, so what you record is what your audience saw. If the app is sent to the background in the middle of a recording it finalizes the clip cleanly rather than leaving a corrupted file, so an interruption costs you the rest of the take but never the part you already captured.
