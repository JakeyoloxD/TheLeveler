# 3D model to LEGO build: a pipeline spec

Paste this whole file to your AI, attach your 3D model, and say:

> Run this pipeline on the attached model. Target <N> studs on the long axis.

It works on any OBJ or STL. It has been run end to end and every constant
below is measured, not guessed.

---

## What you get

1. An OBJ with real LEGO colours baked in
2. A buildable model as an LDraw `.ldr`
3. A parts list as CSV with BrickLink item numbers you can order from
4. Building instructions, auto-generated, as a sectioned `.mpd`

---

## Stage 0: inspect before doing anything

Report before writing any code:

- vertex, face, UV counts
- bounding box on all three axes, and which axis is up
- group and material names
- whether an `.mtl` was supplied
- whether faces carry valid `vt` indices

**Two things go wrong here almost every time.**

Models downloaded from Sketchfab and similar sites often ship the OBJ with
no MTL. Separately, even when the MTL exists it usually holds two or three
materials while all the actual colour lives in a texture atlas addressed by
UVs. Either way, importing "with colours" gives a three-tone model.

If there are UVs and a texture, go to Stage 1. If there are neither, the
model is colourless and the user picks colours by hand.

---

## Stage 1: bake texture to LEGO colours

Per face:

1. Sample the texture at the face's UV centroid **plus five points at 55% of
   the way from centroid toward each vertex**. Take the median.
   Sampling the centroid alone catches the black outline strokes that most
   atlases draw around each island and drags everything toward black.
2. Flip V: `py = (1 - v) * height`. OBJ origin is bottom-left, images are
   top-left.
3. Convert sample and palette to **CIELab**, match nearest by Euclidean
   distance. RGB distance picks visibly wrong colours.
4. Write a new OBJ, `usemtl` per colour, plus a matching MTL with flat `Kd`.

Report the colour distribution weighted by **3D face area**, not face count.
Face count lies badly: a texture-dense region of tiny faces will dominate the
count while covering almost none of the model.

Palette to match against (BrickLink name, LDraw id, sRGB):

```
Black                0    #05131D     Dark Orange     484  #A95500
Dark Bluish Gray     72   #6C6E68     Orange           25  #FE8A18
Light Bluish Gray    71   #A0A5A9     Reddish Brown    70  #582A12
White                15   #FFFFFF     Dark Red        320  #720E0F
Red                  4    #C91A09     Dark Brown      308  #352100
Sand Blue            379  #6074A1     Light Aqua      323  #ADC3C0
Medium Nougat        84   #AA7D55     Yellow           14  #F2CD37
```

Expect four or five colours to carry 95%+ of the surface. Anything under
0.5% is texture detail smaller than a stud: tell the user to drop it and use
the neighbouring colour. Twenty extra BrickLink lots for invisible pieces is
pure shipping cost.

---

## Stage 2: choose a scale

The grid is **anisotropic** and which one you use depends on the build type:

| Build type | X, Z | Y |
|---|---|---|
| Bricks | 8 mm | 9.6 mm |
| Plates (3x the vertical detail) | 8 mm | 3.2 mm |
| Studless Technic | 8 mm | 8 mm |

Using a cubic grid gives a vertically stretched model. This is the single
most common mistake.

**Check the sub-assembly, not the whole object.** Measure the part the user
actually cares about. If half the length is an arm or a tail, the body is
only half the stated size, and the body is what has to read.

Rules of thumb:

- Below ~32 studs on the long axis, silhouette detail dies
- Working Technic tracks need a track bay at least 4 studs tall
- Report the resulting mm dimensions and part count before committing

---

## Stage 3: voxelise the surface

Point-sample each triangle into the grid. Sample density: at least two
samples per grid cell along the longest edge. Assign each cell the modal
colour of the samples landing in it.

Surface only. Do not solid-fill: most downloaded meshes are not watertight,
and a hollow shell is what you build anyway.

---

## Stage 4: pack cells into real parts

Per layer, per colour, greedy scan order, largest legal footprint first.

**Bricks** (LDraw part numbers):

```
1x1  3005    1x2  3004    1x3  3622    1x4  3010    1x6  3009
1x8  3008    1x10 6111    1x12 6112    1x16 2465
2x2  3003    2x3  3002    2x4  3001    2x6  2456    2x8  3007
2x10 3006    4x6  2356    4x10 6212    4x12 4202
```

**Plates**:

```
1x1  3024    1x2  3023    1x3  3623    1x4  3710    1x6  3666
1x8  3460    1x10 4477    1x12 60479
2x2  3022    2x3  3021    2x4  3020    2x6  3795    2x8  3034
2x10 3832    2x12 2445    2x16 4282
4x4  3031    4x6  3032    4x8  3035    4x10 3030    4x12 3029
6x6  3958    6x8  3036    6x10 3033    6x12 3028    6x16 3027
8x8  41539   8x16 92438
```

**There is no 4x4 brick.** A 4x4 plate exists, the brick does not. Never
emit a footprint that is not in the table above.

---

## Stage 5: emit LDraw

```
1 stud     = 20 LDU        brick course = 24 LDU
plate      =  8 LDU        -Y is up
```

Part origin sits at the centre of the footprint on the **bottom** face, so a
part on layer `k` goes at `Y = -24*k` (bricks) or `-8*k` (plates).

Line format:

```
1 <colour> <x> <y> <z> <a b c d e f g h i> <part>.dat
```

Rotation 90 degrees about Y, for parts whose long axis must run along Z:

```
0 0 1 0 1 0 -1 0 0
```

---

## Stage 6: Technic, if the user wants moving parts

Technic is skeleton and skin, not volume. There is no Technic equivalent of
"fill this cell". Do not try to convert a voxel grid into Technic. Use
Technic **only where something has to move**, and keep bricks elsewhere.

**Thick liftarms come in odd lengths only**, plus 1x2:

```
1x2  43857   1x3  32523   1x5  32316   1x7  32524
1x9  40490   1x11 32525   1x13 41239   1x15 32278
```

There is no thick 1x4, 1x6 or 1x8. Watch out for 32063: that is Liftarm
**Thin** 1x6, half the width, and it will not sit in a thick frame.

**Never guess a Technic part's orientation or size.** Fetch the real LDraw
file and measure its bounding box:

```
https://raw.githubusercontent.com/gkjohnson/ldraw-parts-library/master/complete/ldraw/parts/<part>.dat
```

Measured facts worth keeping:

| Part | Fact |
|---|---|
| Gear 24T `3648b` | lies in XY, axis along **Z**, outer radius 27.04 LDU |
| Link Tread `3873` | 52 LDU wide, pitch **16 LDU**, chains along Z |
| Thick liftarms | long axis **Z**, origin at centre |
| Technic bricks `3700/3701/3894` | long axis **X**, hole axis Z, origin at bottom |
| Axles `3705`, `4519` | axis along **X** |

Note the tread pitch. It is 16 LDU, not one stud. Track link count is
`perimeter / 16`, and perimeter is `2 * straight + 2 * pi * R`.

Powered Up, if motorising: Technic Hub **88012** has four ports, so four
functions. Large Motor **88013**, XL Motor **88014**.

Sanity check at large scale: LEGO's biggest gear is about 3.4 studs across.
If the loop needs a sprocket bigger than that, it has to be built from beams,
not ordered as a part.

---

## Stage 7: instructions

Auto-generation only works if the file has structure. A flat parts list
produces either one step or one step per part.

**Insert `0 STEP` markers.** Bottom layer up, front to back within a layer,
about 10 to 20 parts per step. Group any Technic sub-assemblies together at
the end rather than scattering them through the body.

**Split into `.mpd` submodels** for anything over ~500 parts. Cut the model
into sections along the long axis, each a `0 FILE section_NN.ldr`, assembled
by the main file. Without this, the instruction tool frames the entire model
on every page, so the step image is tiny, diagonal, and runs off the edge.

Target roughly 16 studs per section.

Then, in **Bricklink Studio** (free):

1. Open the `.mpd`
2. Instruction Maker
3. Unlock the page, use **Change Layout** to move the parts callout out of
   the centre, set landscape
4. Apply the layout to all pages **before** generating, or you will drag
   several hundred boxes by hand
5. Export PDF

**LPub3D** (free, open source) gives better output: proper callouts, running
bill of materials, cover page. More setup.

---

## Stage 8: tell the user the truth

Always report, unprompted:

- **It is a hollow shell.** One part thick, no internal structure, some parts
  floating. Add 30 to 50% for internal support before treating counts as
  final.
- **Which colour is the cost driver.** Rare colours in small elements cost
  several times common ones and force orders across dozens of stores.
- **What voxelising cannot do.** It produces a blocky proxy, not a designed
  model. Compound curves, SNOT, and anything mechanical are hand design.
  The output is a shape and a shopping list, and that is a real head start,
  but it is not a finished MOC.

---

## Cost model

Common parts run roughly USD 0.05 to 0.15 each used. Rare colours run three
to five times that. Add USD 4 to 8 per BrickLink store, and a big build
spans 40 or more stores, so shipping is often 20% of the total.

Rough bands, from a build actually run through this pipeline:

| Scale | Parts | Ballpark |
|---|---|---|
| 24 studs | ~270 | build from the tub, order nothing |
| 32 studs + working Technic | ~510 | USD 40 to 60 |
| 96 studs, plates, motorised | ~11,000 | USD 1,800 to 2,500 |

---

## Checklist

```
[ ] reported bbox, groups, materials, UV validity before coding
[ ] handled the missing or useless MTL
[ ] baked colour from the texture, not from materials
[ ] used CIELab, not RGB, for palette matching
[ ] reported colour split by area, not face count
[ ] used the correct anisotropic grid for the build type
[ ] checked the sub-assembly size, not just overall length
[ ] emitted only footprints that exist as real parts
[ ] fetched and measured any Technic part before placing it
[ ] inserted STEP markers in a sensible build order
[ ] split to MPD sections if over ~500 parts
[ ] stated the shell caveat and the cost driver
```
