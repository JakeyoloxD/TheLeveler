# README — Turning a 3D model into a real LEGO build

Australian edition. Prices in AUD at 1 USD = 1.40.

You have a folder of files and no idea what to open them with. This tells
you what to download, in what order, and exactly what to do with each file.

Everything in the required list is **free**.

---

# 1. What to download

## Required

### Bricklink Studio 2.0

The main tool. Free, made by BrickLink (owned by LEGO). Windows and macOS.

- **Get it:** https://www.bricklink.com/v3/studio/download.page
- **Size:** about 2 GB installed (it bundles the whole parts library)
- **Account:** not needed to install, but make a free BrickLink account
  anyway — you need one to order parts later

Studio does four jobs for you:

1. Opens `.obj` files and turns them into LEGO bricks
2. Opens `.ldr` and `.mpd` LEGO models
3. Generates building instructions as a PDF
4. Exports a shopping list you can order from

**Install notes:** accept the default install path. Do not install it into a
folder with unusual characters in the name. It bundles its own copy of the
LDraw parts library, so you do not need to download that separately.

### A free Rebrickable account

A website, nothing to install.

- **Get it:** https://rebrickable.com

You need this for one thing, and it is the single biggest money saver in the
whole process: **Multi-Buy**, which works out the cheapest way to split your
order across BrickLink stores including postage.

### A free BrickLink account

- **Get it:** https://www.bricklink.com

This is where you actually buy the parts. It is a marketplace of thousands
of independent sellers, not one shop.

## Optional but worth it

### LPub3D

Free and open source. Makes much better instructions than Studio: proper
part callouts, a running bill of materials, a cover page, page numbers.

- **Get it:** https://trevorsandy.github.io/lpub3d/
- **Also on:** https://sourceforge.net/projects/lpub3d/
- Windows, macOS, Linux

<cite index="72-1">It ships with the LDraw part libraries already bundled and
integrated, and exports to OBJ, 3DS, STL, DAE, CSV and BrickLink XML.</cite>
So you do not need a separate LDraw install for it either.

More setup than Studio. Better output. Use Studio first, come back to this
if you want the instructions to look professional.

### LeoCAD

Free, open source, lightweight. Opens `.ldr` and `.mpd` much faster than
Studio if you just want a quick look.

- **Get it:** https://www.leocad.org

### A text editor

`.obj`, `.ldr` and `.mpd` are all plain text. Being able to open and search
them is genuinely useful when something looks wrong.

- Notepad++ (Windows): https://notepad-plus-plus.org
- VS Code (any OS): https://code.visualstudio.com
- Windows Notepad works, but chokes on big files

### Blender

Only if you want to edit the 3D mesh itself before converting it.

- **Get it:** https://www.blender.org

## Do NOT bother downloading

- **The LDraw All-In-One Installer** — Studio and LPub3D both bundle the
  parts library. Installing it separately just creates a second copy and
  path confusion.
- **LEGO Digital Designer** — discontinued, replaced by Studio.

---

# 2. Set up your folder

Make one folder and keep everything in it.

```
Leveler/
├── Leveler_LEGO.obj              <- 3D model with LEGO colours baked in
├── Leveler_LEGO.mtl              <- colours for the OBJ (must stay beside it)
├── Leveler_32studs.ldr           <- the buildable model
├── Leveler_32studs_technic.ldr   <- same, with Technic parts added
├── Leveler_32studs_sectioned.mpd <- same, split up for instructions
├── Leveler_32studs_parts.csv     <- the shopping list
└── view_side.png                 <- reference renders
```

**Two rules:**

1. **Never rename the `.mtl`.** The OBJ's first line points at it by name.
   Rename it and the colours silently disappear.
2. **Avoid spaces and odd characters in the folder path.** Some LDraw tools
   still break on them. `C:\LEGO\Leveler\` is safe.
   `C:\Users\Jim\My Stuff\LEGO (new)\` is asking for trouble.

---

# 3. Every file type, and what to do with it

## `.obj` — the 3D model

Plain text geometry: every vertex, face and texture coordinate as numbers.
This is the raw shape before it becomes LEGO.

**Just look at it:**
Double click. Windows opens it in 3D Viewer, macOS in Preview. Drag to
rotate. That is all these do — you cannot change anything.

**Turn it into LEGO bricks:**

1. Open Studio
2. File → Import → Import Model
3. Pick the `.obj`
4. A dialog appears asking for size. **Enter the size in studs on the
   longest axis.** Start at 32.
5. Studio builds the brick version. This takes a minute.
6. If it looks too coarse, undo and try a bigger number. If your part count
   is frightening, try a smaller one.

**If it imports grey:** the `.mtl` is missing, renamed, or in a different
folder. Fix that and re-import.

## `.mtl` — the colour definitions

A short text file listing colours. You never open it directly and you never
edit it.

It only has to do two things: sit in the same folder as the `.obj`, and keep
its original filename.

## `.ldr` — the LEGO model

LDraw format. One line per brick, listing colour, position, rotation and
part number. This is the actual buildable model — no more mesh, no more
triangles, just bricks.

**Open it:** Studio → File → Open. Or LeoCAD, or LPub3D.

**Rotate and inspect:** right-drag to orbit, scroll to zoom. Walk around it
before you spend money.

**Get a parts list out:**
File → Export As → CSV. Opens in Excel.

**Get a shopping list out:**
File → Export As → BrickLink Wanted List. This produces an XML file you
upload to BrickLink.

**Check for problems:**
Studio highlights parts that are not properly connected. On a voxel model
there will be plenty. That is expected — see section 6.

## `.mpd` — the LEGO model, in sections

Multi-Part Document. The same thing as `.ldr`, but holding several
sub-models in one file, plus a main file that assembles them.

Opens in exactly the same programs. Studio shows the sections as a tree down
the left side and you can click into each one.

**Use the `.mpd`, not the `.ldr`, whenever you are making instructions.**

Why it matters: the `.ldr` is one enormous object. The instruction tool
frames the entire model on every single page, so the picture is tiny, sits
diagonally, and runs off the edge of the page. The `.mpd` frames one
section at a time and the pictures come out the right size.

## `.csv` — the parts list

A spreadsheet. Columns: BrickLink item number, part name, colour, quantity.

**Open it:** Excel, Numbers, Google Sheets. Or double click.

**Use it for:** budgeting, printing a picking list, checking against what
you already own.

**Do NOT upload it to BrickLink.** BrickLink's Wanted List uploader wants
XML, not CSV. Use one of the two routes in section 4 instead.

## `.png` — reference renders

Side and front views of the model. Any image viewer.

Look at these **before** you order anything. If the shape does not read at
this size, no amount of correct part numbers will fix it.

## `.zip`

Right click → Extract All (Windows) or double click (macOS). Extract it
into your project folder before doing anything else. Do not try to open
files from inside a zip.

---

# 4. The full workflow

## Step 1 — Look at the renders

Open the `.png` files. Does it look like the thing? Is it recognisable?

If not, stop here and change the scale. Everything downstream is wasted
effort otherwise.

## Step 2 — Open the model and walk around it

Studio → File → Open → the `.ldr`.

Right-drag to orbit. Look at it from every angle. This is your last chance
to change your mind cheaply.

## Step 3 — Work out what it costs

Studio → File → Export As → CSV. Open in Excel.

Sort by quantity. The top ten lines are most of your money. Look at the
colours — if there is one unusual colour with a big count, that is your cost
driver and section 5 tells you what to do about it.

## Step 4 — Find the cheapest way to buy it

**This is the step that saves the most money. Do not skip it.**

1. Go to rebrickable.com, sign in
2. My Part Lists → Create a new list
3. Import → upload your `.ldr` or `.mpd` **directly** (Rebrickable reads
   LDraw files natively — the CSV is not needed)
4. Once imported, use **Multi-Buy**
5. Set it to prefer Australian sellers
6. It returns a shopping plan: which stores, which parts, total including
   postage

Ordering naively by cheapest-price-per-part puts you in fifty or sixty
stores at $6 to $12 postage each. Multi-Buy will typically get the same
parts from a dozen. On a 500-part build that is $150 saved. On a big one it
is over a thousand.

## Step 5 — Order

Follow Multi-Buy's plan. Place each store order separately on BrickLink.

**Set condition to Used** unless you have a reason not to. Used is 40 to 60%
cheaper and, for a model that will sit on a shelf, indistinguishable.

**Watch the $1,000 line.** Imported orders under AUD 1,000 have GST taken at
checkout and arrive normally. Over AUD 1,000 they stop at the border for
duty, GST and a processing charge, adding one to three weeks. If one
overseas order is creeping toward $1,000, split it in two.

Expect two to six weeks. Australian sellers are much faster.

## Step 6 — Make the instructions

While you wait for parts.

1. Studio → File → Open → the **`.mpd`**
2. Switch to the **Instruction Maker** tab
3. **Set the page to landscape** before anything else
4. Click **Lock Page** to unlock the page elements
5. Click **Change Layout** and pick a preset with the parts callout in a
   corner or a top strip, not the centre
6. Drag the callout box clear of the model image
7. **Apply the layout to all pages now** — Preferences → Instruction Maker.
   If you skip this you will drag several hundred boxes by hand.
8. Export → PDF

If you want better output, open the same `.mpd` in LPub3D instead. It gives
you a cover page, part callouts and a bill of materials, at the cost of more
setup.

## Step 7 — Sort and build

Parts arrive in dozens of small bags from different sellers.

Sort by **part and colour**, not by bag. Egg cartons, ice cube trays, a
tackle box, whatever. An hour sorting saves three hours hunting.

Then work through the PDF.

---

# 5. Making it cheaper

Ranked by how much they save.

| # | What | Saves |
|---|---|---|
| 1 | Rebrickable Multi-Buy before ordering | up to 40% of total |
| 2 | Scale down — cost tracks the **square** of length | 48→32 studs is 54% off |
| 3 | Bulk used LEGO by the kilo, Gumtree / Marketplace / eBay AU, $25-45/kg | 60-70% |
| 4 | Buy Used, not New, on BrickLink | 40-60% |
| 5 | Bricks instead of plates | about 65% |
| 6 | Cut to three colours | fewer lots, fewer stores |
| 7 | Substitute the one expensive colour for a common one | varies, often 20% |
| 8 | Delete the underside (nobody sees it) | 15-25% |
| 9 | Skip the Technic on version one | about 30% |
| 10 | Buy a junk donor set for bulk parts like tread links | 30-50% on those parts |
| 11 | LEGO Pick a Brick (lego.com.au) for high-quantity commons | one postage charge |
| 12 | Rebrickable checks your list against sets you already own | free parts |

Two things to ask the AI for, which cut cost at the file level:

> Penalise 1x1 and 1x2 footprints in the packing. Prefer 1x4 and larger.

1x1 parts are the worst value per volume in the system and often 30% of the
count.

> Remove all cells on the lowest layer and any cell whose only exposed face
> points downward.

Nobody sees the bottom.

**The cheapest genuinely good build:** 24 to 32 studs, bricks not plates,
three colours from a bulk kilo lot, underside deleted, no Technic. About
**AUD 25 to 45 all in**, roughly 300 parts, 30 to 40 steps.

---

# 6. Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| OBJ imports grey | `.mtl` missing, renamed, or in another folder | Put it beside the OBJ with its original name |
| Model looks vertically stretched | Cubic voxel grid was used | Bricks need 8 / 9.6 / 8 mm, not 8 / 8 / 8 |
| Instruction pictures tiny and cut off | You opened the `.ldr` | Open the `.mpd` instead |
| Parts callout sits on top of the model | Default centred layout | Unlock page, Change Layout, apply to all |
| Hundreds of "not connected" warnings | It is a hollow one-brick shell | Expected. See below. |
| A step asks for a brick with nothing under it | Same reason | Expected. See below. |
| BrickLink rejects the CSV | It wants XML | Use Rebrickable, or export a Wanted List from Studio |
| Order held at customs | Over AUD 1,000 | Split into two orders next time |
| A part number does not exist on BrickLink | LDraw and BrickLink number some parts differently | e.g. gear `3648b` in LDraw is `3648` on BrickLink |
| Studio very slow | Model over ~5,000 parts | Use the sectioned `.mpd` and work one section at a time |

## About the floating bricks

The model is a **hollow shell, one brick thick**. It has no internal
structure, so some pieces have nothing beneath them.

This is not a bug in the files. It is what surface voxelisation produces,
and it is why the part count is as low as it is.

Working out how to hold those pieces up — adding a plate underneath, running
a longer brick across a gap, bracing from inside — is the only part of this
build where you have to actually think. It is also the part worth doing with
a kid, because it is real problem solving rather than following a diagram.

Budget 30 to 50% extra parts if you want the finished thing to be sturdy
rather than fragile.

---

# 7. Glossary

| Term | Meaning |
|---|---|
| **Stud** | The bump on top of a brick. Also the unit of size: 1 stud = 8 mm |
| **Plate** | A thin brick. Three plates stacked = one brick tall |
| **LDU** | LDraw Unit. 1 stud = 20 LDU, 1 brick tall = 24 LDU, 1 plate = 8 LDU |
| **LDraw** | The open file format for LEGO models. `.ldr` and `.mpd` |
| **MOC** | My Own Creation. A model that is not an official set |
| **BOM** | Bill of Materials. The parts list |
| **Lot** | One part in one colour from one seller. 20 lots = 20 line items |
| **Part out** | Splitting a set into individual parts |
| **Voxelise** | Convert a smooth 3D shape into a grid of cubes, then into bricks |
| **SNOT** | Studs Not On Top. Building sideways for smoother surfaces |
| **Technic** | The beam-and-pin system. Frames and mechanisms, not solid shapes |
| **Powered Up** | LEGO's current motor and hub system, app controlled |
| **Wanted List** | A BrickLink shopping list you can match against sellers |
| **Multi-Buy** | Rebrickable's tool for finding the cheapest store split |

---

# 8. Quick reference

```
DOWNLOAD
  Bricklink Studio   https://www.bricklink.com/v3/studio/download.page
  Rebrickable        https://rebrickable.com          (account only)
  BrickLink          https://www.bricklink.com        (account only)
  LPub3D  optional   https://trevorsandy.github.io/lpub3d/
  LeoCAD  optional   https://www.leocad.org

OPEN WITH
  .obj  .mtl   Studio (File > Import > Import Model)
  .ldr         Studio (File > Open), LeoCAD, LPub3D
  .mpd         same — use this one for instructions
  .csv         Excel / Sheets
  .png         any image viewer

ORDER
  Rebrickable > Import the .ldr > Multi-Buy > prefer AU sellers > BrickLink

INSTRUCTIONS
  Studio > open the .mpd > Instruction Maker > landscape >
  unlock page > Change Layout > apply to all > Export PDF
```
