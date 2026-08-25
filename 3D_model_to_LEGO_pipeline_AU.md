# 3D model to LEGO build: a pipeline spec

Australian edition. All prices AUD, converted at 1 USD = 1.40 AUD
(rate as at Aug 2026 — check before you commit to a big order).

Paste this whole file to your AI, attach your 3D model, and say:

> Run this pipeline on the attached model. Target <N> studs on the long axis.

It works on any OBJ or STL. It has been run end to end and every constant
below is measured, not guessed.

---

# PART A — Using the files

You will end up with six file types. Here is what each one is and what to
open it with.

## `.obj` — the 3D model

Plain text geometry. Every vertex, every face, every UV coordinate, as
numbers. Open it in Notepad and you can read it.

| To do this | Use |
|---|---|
| Just look at it | Windows 3D Viewer (double click), macOS Preview |
| Turn it into LEGO | **Bricklink Studio** → File → Import → Import Model |
| Edit the mesh | Blender (free) |

**It must sit in the same folder as its `.mtl`.** Studio reads the MTL
automatically from alongside the OBJ. Move one without the other and you get
a grey model.

## `.mtl` — the colours

A companion text file listing colour definitions. You never open it
directly. Two rules:

1. Same folder as the OBJ
2. Do not rename either file

The OBJ's first line says `mtllib <name>.mtl`. If you rename the MTL, that
line stops matching and colours silently vanish.

## `.ldr` — the LEGO model

LDraw format. One line per brick: colour, position, rotation, part number.
This is the actual buildable model.

| To do this | Use |
|---|---|
| View and edit it | **Bricklink Studio** → File → Open |
| Make instructions | Studio's Instruction Maker, or **LPub3D** |
| Same thing, open source | **LeoCAD** |
| Get a parts list out | Studio → File → Export As → CSV |
| Get a shopping list out | Studio → File → Export As → BrickLink Wanted List |

Also plain text. Useful when something looks wrong: search the file for the
part number and check the coordinates.

## `.mpd` — the LEGO model, in sections

Same as `.ldr` but holds several sub-models in one file. Opens in exactly
the same programs. Studio shows the sections in a tree on the left and you
can click into each one.

**Always open the `.mpd` rather than the `.ldr` when making instructions.**
The `.ldr` is one giant object, so the instruction tool frames the whole
thing on every page and the picture ends up tiny and running off the edge.
The `.mpd` frames one section per page.

## `.csv` — the parts list

Columns: BrickLink item number, part name, colour, quantity.

| To do this | Use |
|---|---|
| Read it, budget it, tick things off | Excel, Numbers, Google Sheets |
| Print a picking list | Same |

**Do not try to upload the CSV to BrickLink.** BrickLink's Wanted List
uploader wants XML, not CSV. Two working routes to an actual order:

- **Rebrickable** (free account) → My Part Lists → Import → it accepts
  **LDraw `.ldr` / `.mpd` directly**. Skip the CSV entirely.
- **Studio** → open the `.ldr` → File → Export As → BrickLink Wanted List.

## `.png` — reference renders

Side and front views so you can sanity check the shape before spending
money. Any image viewer.

---

## The whole workflow, start to finish

```
 1. Get the .obj (and .mtl if it came with one)
 2. AI bakes texture colours          ->  *_LEGO.obj + .mtl
 3. AI voxelises and packs            ->  *.ldr  +  *_parts.csv
 4. Look at the .png renders. Shape wrong? Change scale, go back to 3.
 5. AI adds STEP markers and sections ->  *_sectioned.mpd
 6. Rebrickable: import the .ldr, run Multi-Buy   -> cheapest store split
 7. Order. Wait 2-6 weeks.
 8. Studio: open the .mpd, Instruction Maker, export PDF
 9. Build
```

Do step 6 before step 8. Nothing kills momentum like a half-built model
waiting on a part.

---

# PART B — The pipeline (for the AI)

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

## Stage 3: voxelise the surface

Point-sample each triangle into the grid. Sample density: at least two
samples per grid cell along the longest edge. Assign each cell the modal
colour of the samples landing in it.

Surface only. Do not solid-fill: most downloaded meshes are not watertight,
and a hollow shell is what you build anyway.

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

## Stage 7: instructions

Auto-generation only works if the file has structure. A flat parts list
produces either one step or one step per part.

**Insert `0 STEP` markers.** Bottom layer up, front to back within a layer,
about 10 to 20 parts per step. Group any Technic sub-assemblies together at
the end rather than scattering them through the body.

**Split into `.mpd` submodels** for anything over ~500 parts. Cut the model
into sections along the long axis, each a `0 FILE section_NN.ldr`, assembled
by the main file. Target roughly 16 studs per section.

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

# PART C — Money, in AUD

## Baseline part prices

| | AUD each |
|---|---|
| Common part, common colour, used | 0.07 to 0.21 |
| Common part, new | 0.15 to 0.35 |
| Rare colour (Dark Orange, Sand Blue, Dark Turquoise) | 3 to 5x the above |
| Technic pin, axle, bush | 0.06 to 0.15 |
| Technic liftarm | 0.10 to 0.35 |
| Tread link 3873 | 0.20 to 0.35 |

Per-store shipping within Australia: **AUD 6 to 12**. From overseas: 15 to
40 and three to six weeks.

## What a build actually costs

Real numbers from a model run through this pipeline (a 3D vehicle, ~84
units long):

| Scale | Parts | Cost |
|---|---|---|
| 24 studs, bricks | 273 | build from the tub, order nothing |
| 32 studs, bricks | 490 | AUD 50 to 80 |
| 32 studs + working Technic | 599 | AUD 80 to 120 |
| 48 studs, bricks | 1,058 | AUD 150 to 250 |
| 64 studs, bricks | 1,789 | AUD 260 to 420 |
| 96 studs, plates, motorised | ~11,000 | AUD 2,500 to 3,500 |

Powered Up, Australian retail, roughly:

- Technic Hub 88012: **AUD 130 to 150**
- Large Motor 88013: **AUD 55 to 70** each
- XL Motor 88014: **AUD 70 to 85** each

Check lego.com.au. Australian RRP does not track the USD conversion.

## The GST trap

Imported orders **under AUD 1,000** have GST collected at checkout. Simple,
no delay.

Orders **over AUD 1,000** get stopped at the border. You pay duty, GST and a
customs processing charge, and it adds one to three weeks. If a single
overseas order is creeping toward AUD 1,000, split it into two.

---

# PART D — Building it cheaper

Ranked by how much they actually save.

## 1. Use Rebrickable Multi-Buy. Biggest single saving.

Rebrickable imports LDraw `.ldr` and `.mpd` files directly. Once your list
is in, Multi-Buy works out the cheapest split across BrickLink stores
including their shipping.

A naive order picks the cheapest price per part and lands you in 60 stores.
Multi-Buy will often get the same parts from 12 and save you 40% on
shipping alone. On a big build that is hundreds of dollars.

Set it to prefer Australian stores. Do this before you order anything.

## 2. Scale down. Cost falls faster than size.

Part count tracks surface area, so it scales with the **square** of length.
Halving the size quarters the price.

```
24 studs   273 parts     32 studs   490 parts
48 studs 1,058 parts     64 studs 1,789 parts
```

Going from 48 to 32 studs is a 54% cut. The model is still 26 cm long.

## 3. Bricks, not plates.

Plates give three times the vertical detail and cost roughly three times as
much for the same object. Only go to plates when the shape genuinely
demands it.

## 4. Cut to three colours.

Every extra colour is extra lots, extra stores, extra shipping. Ask the AI
to re-quantise to three colours and see the render. Most voxel models look
fine, and some look better, because the blockiness reads as deliberate.

A single-colour sculpture is cheapest of all, and nobody thinks it is a
compromise. It looks like a design decision.

## 5. Substitute the expensive colour.

Find the rare colour driving cost and swap it. Dark Orange → Reddish Brown
or plain Orange. Sand Blue → Medium Blue. Dark Turquoise → anything.

Ask the AI which colour is the cost driver by **lot count**, not by part
count. A colour spread across 30 different part shapes is worse than one
with 500 pieces of a single shape.

## 6. Buy bulk used LEGO by the kilo.

Gumtree, Facebook Marketplace and eBay AU. Expect **AUD 25 to 45 per kg**
used and mixed. One kilo is roughly 700 to 1,000 small parts.

At those prices a 500-part build is AUD 20 to 30 of bulk instead of 80 to
120 of picked lots. You get whatever colours turn up, which is exactly what
tip 4 is about.

Search "lego bulk kg", "lego mixed lot", "lego bulk box".

## 7. Ask for cost-aware packing.

1x1 parts are the worst value per volume in the entire system. Tell the AI:

> Penalise 1x1 and 1x2 footprints in the packing. Prefer 1x4 and larger even
> where it means a small colour error.

On the model above, 1x1s were about 30% of the count. Pushing the packer
toward bigger footprints cuts both part count and lot count.

## 8. Delete the underside.

Nobody sees the bottom. Tell the AI:

> Remove all cells on the lowest layer and any cell whose only exposed face
> points downward.

Typically 15 to 25% off, for zero visible difference.

## 9. Skip the Technic on version one.

Tracks, gears, motors and frame are around 30% of the cost at small scale
and 100% of the fiddliness. Build the static model, live with it, add the
moving parts later if you still want them. The brick body does not change.

## 10. Buy a donor set instead of loose parts.

For anything you need dozens of — tread links especially — a cheap
incomplete used set on eBay AU is often less than the same parts bought
individually, and it arrives in one box with one postage charge.

74 tread links bought loose is AUD 15 to 25 plus shipping from two or three
stores. A junk Technic set with tracks can be AUD 20 delivered.

## 11. LEGO Pick a Brick for the high-quantity commons.

lego.com.au sells loose parts direct. For 400 Black 1x2 plates it is often
cheaper per piece than BrickLink and it is **one** postage charge, brand
new, no waiting on a hobbyist to pack it.

Bad for rare colours and odd parts. Excellent for bulk commons.

## 12. Check what you already own.

Rebrickable will compare your parts list against sets you own and tell you
what you are missing. If Jim has a tub, this can knock out half the list
before he spends anything.

---

## The genuinely cheapest good build

```
24 to 32 studs
bricks, not plates
three colours, from a bulk kilo lot off Marketplace
underside deleted, cost-aware packing
no Technic
instructions auto-generated in Studio, free
```

**AUD 25 to 45, all in.** About 300 parts, 30 to 40 steps, a couple of
afternoons.

Spending 100x that gets you something 8x longer that took a year. It does
not get you 100x the fun, and it is not the version that teaches a kid
anything.

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
[ ] stated the shell caveat and the cost driver by lot count
[ ] ran Rebrickable Multi-Buy before ordering anything
```
