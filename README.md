# Phenakistoscope 3D CD Project

**by Sean Donny**


\---


## What is this?


So this is a project that combines 2D and 3D animation techniques to produce a procedural Phenakistoscope — basically a 21st century version of a 19th century optical toy, built in Blender.


A **Phenakistoscope** is one of the earliest animation devices ever made. It was invented in **1832** by Joseph Plateau — the idea being that if you spin a disc with sequential drawings on it fast enough and strobe the light (or in the original design, view it through slits in the disc), your brain stitches the frames together into the illusion of motion.


My inspiration was the CD of ***PXE***, the Ecco2k album, designed by Canadian illustrator **Freddy Carrasco**.


The whole thing is built using **2D hand-drawn animation** for the frames, and **Blender** to make it all procedural and 3D. Blender is completely free and open source — grab it at [blender.org](https://www.blender.org/) if you don't already have it. I used **Blender 5.1** for this project specifically.


\---


## The Full Breakdown — How to Actually Make This


I'm going to walk you through this from start to finish. Don't panic if you're newer to Blender — I'll flag the important concepts as we go and explain the maths wherever it shows up. Intermediate Blender users should be able to follow this pretty comfortably. If something mentions a topic that's a bit deeper than we can cover here, I'll 
link resources at the end.


---



### Part 1 — The 2D Animation


Firstly, you need your animation frames...



#### How many frames?



I used **12 frames**. This is a pretty classic choice for a looping animation — it's enough frames to read as fluid motion without being an overwhelming amount of work to draw.

It also divides cleanly into a full circle, and it's a standard number in traditional animation timing.

You can technically use any number of frames — 8, 16, 24 — just keep in mind the complexity and work involved.



#### What software do I use?



I drew mine in **Photoshop**, using the Timeline panel to build and scrub through frames. But honestly, use whatever you're comfortable with for frame-by-frame animation. Procreate, Krita, Clip Studio Paint, TVPaint — all of these can export individual frames. The end goal is just to have your frames as individual image files you can line up in order.



**Tip**: Whatever you use, make sure you can export each frame as a **separate PNG file** with a transparent background (unless your design specifically needs a background baked in).


#### Animating well — principles that actually matter here



Because this is a **looping** animation on a spinning disc, a few things are especially important:

* **It should loop seamlessly.** Frame 12 should flow back into Frame 1 without a jump. While you're drawing, keep checking that transition constantly.
* **Use reference.** I recorded myself doing the movement I wanted to animate, and used that as a guide. It sounds basic but it makes a huge difference, and kills the need for guesswork.
* **Squash and stretch** gives your animation weight and life. Things that move fast get stretched in the direction of motion; things that land or hit compress. Without it, animation feels stiff and dead.
* **Anticipation and overshoot** — before a movement, there's a small wind-up in the opposite direction (anticipation). After the movement lands, it goes slightly past the target before settling (overshoot).

**Smear frames** — on fast movements between two positions, rather than just cutting hard, you draw a distorted smear frame that suggests the blur of motion.



\---



### Part 2 — Building the Image Atlas



Once your 12 frames are drawn and exported as individual PNGs, you need to combine them into a single image called an **image atlas** (also sometimes called a sprite sheet).



#### What is an image atlas?



Instead of 12 separate image files, you have **one long horizontal strip** containing all your frames laid out side by side, left to right, in order. Frame 1 on the far left, Frame 12 on the far right — one continuous image.

This is important because Blender's material system works with a single texture at a time per material slot. Rather than juggling 12 separate textures and 12 separate materials, we use one texture and use maths to tell Blender *which section* of that texture to show on each frame plane. Much cleaner, much more procedural.

#### 

#### How to construct it



Open all 12 frames in Photoshop (or any image editor that supports canvas resizing) and create a new document that is **\[frame width × 12] wide** and **\[frame height] tall**. Then paste each frame into its correct position — Frame 1 at X=0, Frame 2 at X=(frame width), Frame 3 at X=(frame width × 2), and so on.



For example, if each frame is **512×512px**, your atlas will be **6144×512px**:

```
512 × 12 = 6144px wide
```

The result is one wide strip that looks like this:

```
\[Frame 1]\[Frame 2]\[Frame 3]...\[Frame 12]
|--------|--------|--------|    |--------|
0       512     1024    1536   5632    6144  (pixel positions)
```



**Tip**: Make sure the frames are lined up precisely — no gaps, no overlaps. If any Frame is even a few pixels off, you'll see a sliver of the wrong frame bleed through in Blender. Use snap-to-pixel-grid if your software supports it.

Export the final atlas as a single PNG. Name it something clear like `phenakistoscope\_atlas.png`. You'll be importing this into Blender in the next step.



\---



### Part 3 — Modelling \& Texturing the CD



Before the Geometry Nodes magic, you need an actual CD model. If you already have one or want to use a different object like a vinyl, feel free to model it, use the one in the file, or get one off online marketplaces like Sketchfab.



\---



### Part 4 — Geometry Nodes: Placing the Frames Procedurally



This is where the project gets really interesting. **Geometry Nodes** is Blender's node-based system for procedurally generating and manipulating geometry.



Open your **Geometry Nodes editor** and add a Geometry Nodes modifier to a new object (it can be a cube or any other mesh) (I'd recommend keeping the CD as it's own object at the centre of the scene, then adding this new object at the same centre, just for organisation and for when it is time to animate them).



#### Step 1 — The Cylinder as a Point Distribution



The core idea here: we need 12 evenly spaced points arranged in a circle, one for each animation frame. The easiest way to get 12 perfectly evenly spaced points in a circle in Blender? A **cylinder with 12 vertices**.



Add a **Mesh Primitive → Cylinder** node to your node tree. Set **Vertices** to **12** (matching your frame count). Set the **Depth** to **0** — we only want the ring of top vertices, not a tall tube.



**Why a cylinder?** A cylinder in Blender is just a circle of vertices with caps. By setting depth to 0, we flatten it so it's just a ring of 12 points. The maths that distributes them evenly is simple: \*\*360° ÷ 12 = 30°\*\* between each point. The cylinder does this for you automatically — you just have to ask for 12 vertices.



Feed the cylinder into a **Mesh to Points** node. This converts the mesh vertices into a point cloud you can instance things onto.



#### Step 2 — Creating the Frame Planes (Instances)



Now we have 12 points. We want to place a flat rectangular plane at each one — this is the surface each animation frame will be displayed on.



Add a **Mesh Primitive → Grid** node. Size it to match the proportions of a single frame from your atlas. This grid is the geometry that will be instanced 12 times.



Add an **Instance on Points** node. Plug your 12-point cloud into the **Points** input, and plug your Grid geometry into the **Instance** input. You should now see 12 planes arranged in a circle around the origin.



**Tip**: At this stage they'll all be flat and facing the same direction. That's fine — we fix rotation next.



#### Step 3 — Rotating the Frames to Face Outward



Each frame plane needs to rotate so its face points outward from the disc's centre, with its top edge pointing away from the centre — like the hour markers on a clock face.



After your **Instance on Points** node, add a **Rotate Instances** node. To rotate each instance by a different amount based on its position in the sequence, you'll use the **Index** node — this outputs each instance's number (0, 1, 2... 11).



Multiply the Index by `(2 \* pi / 12)` using a **Math** node set to **Multiply**, then feed the result into the **Rotation Z** input of the Rotate Instances node.



> **Why `2 \* pi / 12`?** This is the angle in \*\*radians\*\* between each of the 12 equally spaced positions around a full circle.
>
> - A full circle = \*\*360°\*\* = \*\*2π radians\*\* (≈ 6.2832)
> - One step out of 12 = \*\*2π ÷ 12 ≈ 0.5236 radians\*\* = \*\*30°\*\*
>
> Blender's maths nodes use radians internally, so we express it this way. The result: Instance 0 rotates 0°, Instance 1 rotates 30°, Instance 2 rotates 60°, and so on around the full circle.



#### Step 4 — Storing the Frame Index as a Named Attribute



Here's the real procedural sauce. We need each frame plane to "know" which frame of animation it should display. We do this by storing each instance's index number as a **named attribute** on the geometry, so the material shader can read it later.



Add a **Store Named Attribute** node. Set the **Name** to something like `frame\_id`. Set the **Data Type** to **Integer**. Plug the **Index** node into the **Value** input.



> **What is this for?** The named attribute is the bridge between Geometry Nodes and the Material shader. Geometry Nodes sets a per-instance value (`frame\_id = 0` on the first plane, `frame\_id = 1` on the second, etc.), and the shader reads that value to calculate which section of the image atlas to display. Without this, every plane would show the same thing.



#### Step 5 — Z-axis Offset to Prevent Clipping



One last small but important thing: nudge each frame plane very slightly upward along the Z axis so they don't sit at exactly the same depth as the CD surface.



When two surfaces occupy the same depth, you get **Z-fighting** — that horrible flickering/striped artifact where the renderer can't decide which surface is in front. A tiny Z offset fixes it.



Add a **Set Position** node and add a small Z value — somewhere between `0.001` and `0.005` Blender units is usually enough.



> **Tip**: Don't go too large with the offset or the planes will visibly float above the disc. You want it close — just enough to give the renderer a clear winner.



\---



### Part 5 — The Material: Using the Image Atlas



Now we set up the material that reads from the atlas and shows the correct frame on each plane. Open the **Shader Editor** with your frame geometry object selected and create a new material.



#### How the UV maths works on a horizontal strip atlas



Your atlas is one long horizontal strip. Each of the 12 frames occupies exactly `1/12` of the total width:

```
Full atlas UV width = 1.0 (in UV space, 0.0 to 1.0)
One frame's UV width = 1.0 / 12 ≈ 0.0833
```

So the UV offset for each frame is:

* Frame 0: U starts at `0 / 12 = 0.0`
* Frame 1: U starts at `1 / 12 ≈ 0.0833`
* Frame 2: U starts at `2 / 12 ≈ 0.1666`
* Frame N: U starts at `N / 12`

In the shader, we need to do two things to the UV coordinates:

1. **Scale** the U axis by `12` — so that the plane only "sees" one frame's worth of the atlas at a time (zooming in to 1/12th of the width)
2. **Offset** the U axis by `frame\_id / 12` — to slide the window to the correct frame


#### Node setup in the Shader Editor



**1.** Add an **Attribute** node. Set the **Name** field to `frame\_id` (exactly matching what you named it in Geometry Nodes). This pulls the per-instance frame index into the material. Use the **Factor** output.

**2.** Add a **Math** node set to **Divide**. Plug `frame\_id` into the first input and set the second value to `12`. This gives you a value between 0.0 and \~0.917 — the UV offset for each frame.

**3.** Add a **Texture Coordinate** node and connect the **UV** output to a **Mapping** node.

* In the **Mapping** node's **Scale X**, type `12`. This zooms the U axis so only one frame is visible at a time.
* Plug the result from Step 2 into the **Location X** of the Mapping node. This slides the UV window to the right frame.

**4.** Connect the Mapping node's output into an **Image Texture** node. Load your `phenakistoscope\_atlas.png`.

**5.** Connect the Image Texture's **Color** output into your **Principled BSDF** (or an **Emission** shader if you want the frames to be self-lit. Plug the **Alpha** output from the Image Texture into the **Alpha** of the BSDF. In the material settings panel, set the **Blend Mode** to **Alpha Clip** or **Alpha Blend**.

> **If the frames look shifted by one** (first plane shows Frame 2, etc.): your index is likely starting at 1 instead of 0 somewhere. Check whether your Attribute node is returning 0-indexed values and subtract 1 from the frame\_id before dividing if needed.

> **If you see the full atlas stretched across a plane** instead of a single frame: your Scale X isn't set to `12`. Every plane is showing the full atlas squished down — scale zooms you into the right portion.

> **Why scale first, then offset?** If you offset first, the scaling would amplify your offset and shift everything wrong. The order matters: scale the UV space to frame-size, then slide to the correct position.



\---

### Part 6 — 3D Animation \& Making the Phenakistoscope Work



You've got a CD, 12 frame planes sitting on it, each showing the right drawing. Now you need to make it *spin* — and not just spin freely, but spin in sync with the animation frames so the optical illusion actually fires.



#### Setting up the parent object



Add an **Empty** object (*Shift+A → Empty → Plain Axes*) and position it at the exact centre of your disc. Select your CD and frame geometry, then Shift-select the Empty last, and hit *Ctrl+P → Object* to parent them to it.



> **Why?** Parenting everything to a single Empty means you can move, rotate, and animate the whole assembly by just touching the Empty — without accidentally breaking any of the procedural setups. Good practice for any complex Blender scene.



#### Making sure all origins are centred



Go through each object — the CD, the frame instances object — and confirm their **origins are at the centre of the disc**. If anything's off: place the 3D cursor at the disc centre, select the object, and go *Object → Set Origin → Origin to 3D Cursor*.



This is critical. If an origin is off-centre, that object will orbit around the wrong point when it rotates, and the whole illusion falls apart.



#### The rotation driver — the magic trick



Now for the centrepiece of the whole setup. We're going to use a **Driver** to link the Z-rotation of the disc to the timeline frame number — but in a specific stepped way, so the disc snaps between frame positions rather than spinning smoothly.



Select your Empty. In the properties panel ("N" on your keyboard), find the **Z Rotation** field. Right-click it and choose **Add Driver**. The Driver Editor will open.



Delete any default variables Blender adds (`var`, etc.) — we don't need them. In the driver expression field, enter this:

```
-(floor((frame - 1) / 2) \* (2 \* pi / 12))
```



#### Breaking down the maths in this expression



Let's go through it piece by piece — understanding this means you can customise it for any frame count or timing.

**`frame`** — Blender's built-in variable for the current timeline frame number. Updates automatically as you play or scrub.



**`frame - 1`** — Subtract 1 so that at timeline Frame 1, the expression equals 0 (no rotation yet). Without this, the disc starts pre-rotated by one step.



**`/ 2`** — The timing divisor. Dividing by 2 means: "advance one animation frame for every 2 timeline frames." This is **animation on 2's** — a classic traditional animation technique where each drawing is held for 2 frames. At 24fps timeline: 12 drawings × 2 frames each = a 24-frame loop. It looks natural and fluid.



> **Want to change the timing?**
> - **On 1's** (every timeline frame is a new drawing — snappier): change `/2` to `/1`
> - **On 3's** (slower, more held): change `/2` to `/3`
> - **Different frame count?** Keep the `/2` but change both instances of `12` to match your frame count.



**`floor(...)`** — Rounds *down* to the nearest whole number. This is what makes the rotation **step** rather than glide. Without `floor()`, the disc would rotate continuously and you'd just see motion blur. With `floor()`, it snaps — one position, next position, next position — exactly replicating the slitted disc of the original Phenakistoscope.



> **The maths of why floor works**: At timeline frame 1: `floor((1-1)/2) = floor(0) = 0` steps. At frame 2: `floor((2-1)/2) = floor(0.5) = 0` steps (still). At frame 3: `floor((3-1)/2) = floor(1) = 1` step. So the disc holds for 2 frames, then snaps — exactly on 2's.

**`2 \* pi / 12`** — The angle per step in radians. `2 \* pi` is a full 360° circle. Divided by 12, each step = **30°** (≈ 0.5236 radians). The disc advances one frame position at a time.



**The `-` (negative sign)** at the front reverses the direction so the animation plays forward. If your animation plays in reverse, remove the negative sign.



**Full expression in plain English:** *"At any given timeline frame, count how many animation frames have elapsed (stepping every 2 timeline frames), and rotate the disc by that many 30° increments, clockwise."*



#### Keyframing and watching it work



With the driver in place, hit **Space** to play your timeline. Watch the disc spin in stepped increments — and watch your hand-drawn animation come alive on a spinning 3D CD.



If you want the disc to also travel through 3D space (floating toward the camera, tilting, etc.), animate the **parent Empty** for that separately. You can also just add another Empty and parent the existing one to the new one (Name them properly so you don't confuse yourself), then animate the new one for the movement, this is known as separation of concerns.



\---



## Quick Troubleshooting



**Frames showing the wrong drawings (shifted by one):**
Your index might be 1-based somewhere. Subtract 1 from the `frame\_id` before dividing in the shader, or check how your Attribute node outputs the value.



**Slivers of adjacent frames bleeding in:**
Either your atlas frames aren't pixel-perfectly aligned, or your UV Scale X isn't exactly `12`. Double-check both.



**Animation plays backwards:**
Remove the `-` from the front of the driver expression.



**Z-fighting / flickering on the frame planes:**
Increase the Z offset in your Set Position node slightly.



**Disc wobbles when spinning:**
An origin is off-centre. Go through every object in the assembly and reset origins to the disc centre.



**Driver expression throws an error:**
Make sure you've deleted all default variables Blender added. The expression only uses Blender's built-in `frame` and `pi` — no external variables needed.



\---

## 

## Resources \& Credits

Big shoutout to **Ducky3D**, **Xan 3D**, **Intranet Girl**, and **CGMatter** — a huge chunk of the Geometry Nodes and Blender knowledge that made this possible came from their channels. Genuinely some of the best Blender content out there, go check them out.



### 2D Animation

|Topic|Link|
|-|-|
|Photoshop frame-by-frame animation — quick intro|[Watch](https://youtu.be/IO1yDUJl8qw?si=mJBP0TV7jIzZXHQG)|
|Using live action reference to learn animation \& improve drawing|[Watch](https://youtu.be/-Y6Kkrlvzgs?si=UZrYot9-bLVGA0Qf)|
|Disney's 12 Principles of Animation — full series|[Watch](https://youtu.be/uDqjIdI4bF4?si=mqcLkuAMYCdwxGDk)|

### 

### 3D Animation

|Topic|Link|
|-|-|
|How to Use Store Named Attribute|[Watch](https://youtu.be/vM9m8HAT6kI?si=ueGr5R-DbcA7uKRh)|
|Using Geometry Nodes for Animation|[Watch](https://youtu.be/-aJahfHZpzg?si=gakAbMofbMhiqTOU)|
|Phenakistoscope Planning|[Watch](https://youtu.be/i-aBtrgjWkE?si=QEvPV4kuwrKnr4PW)|
|Phenakistoscope Inspiration|[Watch](https://youtu.be/ofqs7vwg3Wc?si=vYEpZGTmBwzd482X)|
|Phenakistoscope Development|[Watch](https://youtu.be/zJqJL3DG908?si=IwmqdpsXTFC6r-uB)|
|Phenakistoscope IRL Setup|[Watch](https://youtu.be/98W0WBfTbms?si=R_mypY5-50HUP0wy)|
|Realistic CD Modelling|[Watch](https://youtu.be/fZ7fYmyVHak?si=Xs-93oxfAGMkYCoO)|
|CD Surface imperfections|[Watch](https://youtu.be/KmRl-pYInpY?si=xEx31wDmYIrCA6l-)|



\---



## Licence

Open source — use it, build on it, make it your own. Just give credit where it's due.

