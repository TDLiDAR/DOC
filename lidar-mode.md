---
title: LiDAR Mode
layout: default
parent: App Guide
nav_order: 2
---

# LiDAR Mode

LiDAR mode turns the phone's depth camera into a live NDI video feed: a colour-mapped depth image (and, optionally, the plain RGB camera) that any NDI receiver on the network can pick up. It is the app's original mode and has the deepest set of controls, because depth from a sensor needs cleaning, ranging, toning and colouring before it looks good on screen.

## The live screen and its buttons

When you enter LiDAR mode you see the live depth preview filling the screen, with a thin strip of controls.

The **flip** button swaps between the rear LiDAR camera and the front TrueDepth camera. Flipping mid-stream is handled cleanly, and the app remembers a separate depth profile for the front camera so your face-tuned settings don't disturb your room-tuned settings.

The **record** button captures the live output to a clip. The clip is the same processed depth (or RGB) that is going out over NDI, and it is saved to your Photos library when you stop. If you flip the camera while recording, the app either asks first or, if you have turned that warning off, silently saves the current clip and switches.

The **NDI** indicator shows that the depth stream is being broadcast and, with the source dropdown, which name it is broadcasting under.

The **frame-rate pill** shows the real frames per second. Tap it to jump straight into Settings. Watch it as a health meter — a number well below your target means the phone is throttling.

The **gear** opens the full settings sheet, and a compact **slide-up dock** at the bottom edge reveals the main controls without covering the preview. A **double-tap** on the preview toggles the aspect framing between the tall and wide layouts.

The rest of this page is the settings sheet, control by control.

## Source and mode

**Depth Source** sets the NDI name the phone advertises. Tapping it cycles a small index, so if you run several phones on one network each broadcasts a unique source name instead of clashing — phone one, phone two, and so on.

**Mode** chooses the depth pipeline. **Environment** uses the rear LiDAR for room-scale depth out to several metres. **Face Detail** uses the front TrueDepth camera and is tuned for close, fine facial depth. **Raw** passes the sensor depth through with the least processing, for when you want to do your own work downstream.

## Range and clipping

**Max Range**, in Environment mode, sets how far out depth is mapped before everything beyond reads as empty. Higher keeps distant scene depth at the cost of resolution up close; lower concentrates all the detail in the near field. The Pro **Extended LiDAR Range** option pushes this picker out to ten and twelve metres for large spaces.

**Face Detail Range** is the depth window, in metres, kept around the median face distance in Face Detail mode. Tighten it to cut the face cleanly from the background; widen it to keep more of the surroundings.

**Near Clip** and **Far Clip** are hard cutoffs in metres. Anything nearer than the near clip or farther than the far clip is forced to black and dropped. They fence the depth to exactly the slab you care about, which is the cleanest way to remove a busy background or your own hands.

## Cleaning the depth

**Smart Hole Fill** fills the small black gaps the depth sensor leaves — most visible around the edges of objects in the front-camera modes — and keeps the rear LiDAR smooth.

**Disable Apple Depth Filtering** is the opposite switch and is off by default. Left off, the gaps are filled and the depth is smooth. Turned on, you get raw, unfiltered depth with crisper edges, but the black gaps return. Pick smoother-but-filled or sharper-but-holey depending on the look.

**Smoothing** averages depth across frames to calm shimmer. More is steadier but adds a little lag. There is a second, dedicated smoothing control on the advanced page specifically to dampen the rear LiDAR's occasional bright flash.

## Tone and definition

**Invert Depth** swaps which end is bright. By default near is bright and far is dark; invert to flip that.

**Brightness**, **Contrast** and **Gamma** are the output tone curve. Brightness shifts the whole image lighter or darker, contrast spreads or compresses the midtones, and gamma bends the response to lift shadows or crush highlights. They shape the colour-mapped image, not the underlying distances.

**Definition** is the one control most people use. It stacks contrast, near-range lift and edge enhancement into a single knob so small objects separate from each other without tuning three things by hand. Higher means more separation; zero gives the older, flatter look. When the rear LiDAR blows out the foreground, the advanced **Near Compression** and **Near Threshold** controls rein in the over-bright close range — most users never need them.

## Colour and style

The colour map chooses the palette that turns distance into an image. The **Colour-Map Effect** toggle layers a stylized look on top. **Topographic Bands** draws contour-like rings at set depths, and **Posterize Bands** quantizes the depth into stepped levels; each has its own density slider. A separate depth-edge **outline** control traces crisp lines along depth discontinuities in the NDI and recorded output, with zero meaning off — good for a clean line-art look over the colour.

## Frame rate

**Target FPS** sets the capture rate. The **60 fps** option is a deliberate trick: depth is captured at 30 fps but the NDI stream is advertised at 60 fps with each frame doubled, so receivers play it back smoothly at 60 Hz with no added judder. It costs nothing in capture load and makes downstream playback feel smoother.

## HD and masking (Pro)

For Pro users, **HD resolution** upscales the output with a high-quality Lanczos pass over the sensor-native frame. The on-device preview keeps showing the native frame for responsiveness, while the NDI stream and recordings go out at the chosen HD size. **Edge-preserving smoothing** is a gentle convolution over the upscaled frame that softens the stair-step edges upscaling introduces. **Depth mask alpha** keys the depth so the background goes transparent, which lets you composite the subject straight over other layers in TouchDesigner.

## Sending the colour camera

**Send RGB Camera** broadcasts the ordinary colour camera as its own NDI output alongside the depth. With both arriving in TouchDesigner you can composite the real image against the depth, key one with the other, or simply use the phone as a wireless camera.

## Receiving it in TouchDesigner

On the TouchDesigner side, an NDI In TOP picks the phone from its source dropdown — no IP or port. If you also turned on Send RGB Camera, a second NDI source appears for the colour feed. The family includes ready-made NDI and Depth operators that wrap this up for you.
