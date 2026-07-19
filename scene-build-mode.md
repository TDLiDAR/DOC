---
title: Scene Build Mode
layout: default
parent: App Guide
nav_order: 8
---

# Scene Build Mode

Scene Build runs Apple's RoomPlan to scan a room into a clean architectural model — walls, floor, doors, windows, openings and the major pieces of furniture — rather than a raw point cloud. It is the right mode when you want the shape of a space as tidy geometry instead of millions of points.

## How it works

RoomPlan uses the LiDAR and the camera together to recognize architectural surfaces and objects as you move the phone around the room. It doesn't capture every speck of detail; it fits clean planes and boxes to what it sees, so the result is a small, structured model you can actually work with — a wall is one rectangle with holes cut for its doors and windows, not a cloud of points.

## The live screen and its buttons

While scanning, a live mini-view shows the model building up so you can see which surfaces have been captured and which still need a pass. The main controls are **Scan**, which starts and continues the capture, **Reset**, which clears the current scan so you can start over, and **Finish**, which ends the scan. When you finish, the same view expands to Apple's own finished review model — correctly sized and fully rotatable with pinch-to-zoom — so you can inspect the result before doing anything with it.

A practical tip while scanning: move slowly and keep walls in frame from a slight angle. RoomPlan needs to see a surface from more than one position to lock it in, so a steady walk around the perimeter works far better than waving the phone.

## What you can do with it

The finished model can be reviewed on the phone and, depending on your workflow, exported or streamed to the Scene Build operator in TouchDesigner, which rebuilds the room as geometry with separate colours for walls, floor, doors, windows, openings and objects. Because the model is structured, you can style each category independently in your patch.

## Limits worth knowing

RoomPlan is built for rooms, not objects. It captures architectural scale — surfaces and large furniture — and ignores fine detail. It works best in a normally lit, normally sized room. Very large or very cluttered spaces, mirrors and glass, and extreme lighting all reduce its accuracy. For capturing a single object or fine detail, use Point Cloud mode instead.
