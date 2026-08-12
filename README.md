# Corner Deck Stair Calculator

A single-file, offline calculator for a **cascading corner (pyramid) deck stair**: a top platform at the
house door with steps descending from **two adjacent faces**, wrapping the outside corner on a 45° miter,
landing on the existing deck surface.

Open [`stairs-calculator.html`](stairs-calculator.html) in any browser. No install, no server, no
internet. Add `?test=1` to the URL to run the built-in geometry self-tests.

---

## What it answers

1. **Is the geometry legal and comfortable?** Riser, run and tread depth checked against Québec/CNB
   private-stair limits, with every valid riser count listed for your measured rise.
2. **What exactly do I cut?** Framing-square marks for cut stringers (including the corner hip
   stringer), or piece-by-piece dimensions for stacked box frames.
3. **What is the finished footprint?** Scaled plan and elevation drawings so you can check it fits the
   deck and clears the door.
4. **Do I have to cut boards?** Whether tread depth and riser height come out to whole stock boards,
   or need a rip, a gap adjustment, or a splice — plus a buy list optimized against the stock lengths
   you can actually get.

---

## Assumed configuration

| | |
|---|---|
| Layout | Platform against the house wall; steps descend from the **front** face and **one side** face |
| Corner | Outside corner, 45° miter (both mitered treads and through-leg/butt treads supported) |
| Base | Platform is a low framed box **sitting on the existing deck surface** on sleepers/pads — no footings |
| Landings | The **deck** is the bottom landing, the **platform** is the top landing |
| Units | Inches internally; imperial / metric / both display toggle. Code limits stored in mm |

If your stair is not this shape — three-sided wrap, inside corner, or a stair to grade with footings —
the numbers will not apply.

---

## Running it

Open `stairs-calculator.html` in any browser — it is one self-contained file with no dependencies, no
build step and no network access, so it works straight off disk and offline.

**Published:** <https://justinccase29.github.io/stairs/> (GitHub Pages, `main` branch, root folder).
`index.html` redirects to the calculator so the bare URL works. Add `?test=1` to the URL to show the
self-test tab.

## The skirt

The stair wraps two faces of the platform and the house closes a third. The fourth — the **X=0 face** —
is the only exposed *end*, where you would otherwise see the cut ends of every box frame or stringer.
The **skirt** is the finish board that closes it, cut to the step profile.

Set `Far side face (X=0)` to `Against a wall` and the skirt disappears entirely — the field, the framing
section and the cut-list pieces. Set it to `Open` and you get two pieces: the raking board down the
stair (`hyp(total run, total rise)`) and one along the platform's rim. The tool also reports the
**minimum board width** the rake needs — `notch depth + 2"` — and flags the chosen stock if it's too
narrow to cover the profile.

## Your inputs are remembered

Every edit is written to `localStorage` immediately and read back when you reopen the page — including
the stock library and the code-limit table. The header shows `Saved 14:32` after each change, or
`Restored: 6 risers, box frames` on open — it **names** what came back, so a silent revert to the
defaults is obvious instead of looking like a normal restore.

### The bug that lost designs

The flush on unload was the problem. It fired whatever the page happened to be holding, so a visit that
**did not restore anything** — storage briefly unreadable, a payload the build could not parse, a fresh
profile — wrote its *defaults* straight over a good design on the way out. One bad load and the design
was gone for good. That is what "it forgets my options after an update" looked like.

Three things now stand in the way:

* **A session has to earn the right to write.** The unload flush only runs if this visit restored the
  saved design or actually changed something. An untouched page that started from defaults can no
  longer overwrite anything.
* **One step of undo.** The payload being replaced is kept, and a **Restore previous** button appears in
  the header whenever one exists. It swaps, so pressing it twice puts you back.
* **A version on the payload**, so a future rename migrates instead of silently falling back to the
  default for that field. A corrupt payload is *kept* and reported, not treated as "nothing saved".

The stock library is also **merged over the defaults** rather than replacing them, so a library saved
before an item existed no longer loses that item.

If a browser blocks storage (private windows, some `file://` configurations), the header says so
**instead of failing silently**, so you know to use `Export JSON`.

One caveat remains: `localStorage` is **per origin**. The published copy at
`justinccase29.github.io` and a local copy opened from disk keep *separate* saved designs. Use
`Export JSON` / `Import` to move one between them.

## Layout

The input column is **resizable** — drag the divider between it and the drawings, double-click it to
snap back to 340 px, or focus it and use ←/→ (Shift for bigger steps). Clamped to 240–900 px, saved
with the project, preserved across **Reset**, and hidden when printing.

## Run is nosing to nosing

`R` is the going, measured **nosing tip to nosing tip** — that is how the model has always computed it,
and the plan, elevation and terminology drawings now dimension it between the two nosing tips with
witness lines, rather than between frame faces (same length, but the frame is the wrong datum to show).

## Tread boards start at the nosing

Boards are laid out **from the nosing inwards**: board 1 is always a *full* board with its front edge on
the nosing tip, and any odd piece is ripped and placed at the **back**, where the riser board covers it.
Board numbering, the cut list, the plan view and the section all follow that order, so board 1 in the
table is the board you set first.

Overshooting backwards is never offered — behind the tread's back edge sits the frame of the level
above, so the extra material has nowhere to go. With an 11" run, a 1" riser and a 1" nosing the layout
is `5½" + 5½" + 1¾" rip`, the rip at the back.

## Floating platform

The platform is modelled as a **free-standing (floating) frame** — nothing is fastened to the house:

* the frame is the platform frame stock **on edge**, so its depth is that stock's width (a 2x8 gives 7¼");
* **legs have their own stock**, defaulting to **4x4**. A square post takes the load in compression on
  its own; frame stock on edge is a plank and wants doubling into an L at the corners plus a diagonal
  brace once it gets tall, so that warning only fires for the plank case and names the fix;
* leg **spacing** still comes from the *frame*, because that is the member spanning between the legs
  (a 27" platform on 2x8 → 12 legs at 19¾", 4 × 3 at 24" o.c.);
* if the frame stock is deeper than the whole platform, it says to rip it instead.

The elevation draws it that way — frame band on real legs at the leg stock's width, not a solid block.

## Nosing is measured from the riser face

A **captured** riser board sits on the tread below it and tucks under the tread above, so *its face* —
not the frame — is the last vertical surface. The nosing is measured from there, which means the tread
board is one riser thickness deeper than the going:

```
board depth   = run + riser thickness + nosing      ← what you cut
walking depth = run + nosing                        ← what code measures
```

With an 11" run, a 1" riser and a 1" nosing, the tread board is **13"** and the walking depth is 12".
Set the nosing to 0 and the tread still laps the riser to 12", finishing flush with its face — it never
leaves the riser's top edge exposed. Open risers (or `riserFit: laps the tread edge`) add nothing, so
the depth returns to `run + nosing`. The bottom footprint and the platform's finished size grow by the
riser thickness too, and the Summary reports both depths.

## Step numbering

Steps are numbered **from the platform down**: `L1` is the first step below the platform, `L(N−1)` is
the last one before the deck. The platform is the top landing and the deck the bottom landing — neither
is numbered. Drawings tag each tread `L1 (top)`, `L2`, `L3 (bottom)` with its height, the step schedule
adds a plain-language "which step" column and rows for both landings, and every table, sheet and cut
list uses the same two helpers so the numbering can't disagree between views.

## Label placement

Every drawing runs through a **declutter pass** before it is returned: overlapping labels are nudged
apart and any leader line that pointed at a moved label is dragged with it. Pairwise relaxation was
tried first and oscillated — a label pushed off one neighbour landed on another and got pushed back —
so it uses a single ordered placement pass that never revisits a settled label and always terminates.
Dimension lines and their ticks are deliberately excluded from the leader-dragging, since moving those
would falsify the measurement. A test renders the tightest layouts and asserts zero overlaps.

## Language

**EN / FR toggle** in the header, next to Units. It re-labels everything — inputs, hints, tables,
warnings, the cut list, and every label inside the drawings — and sets `<html lang>`. The choice is
saved with the project and survives **Reset** (it's a UI preference, not a stair input).

In French, decimals display with a comma (`9,333"`, `711 mm`) and a comma-decimal entry like `9,5`
parses correctly. Imperial fractions, feet and inch marks are unchanged, since that's how the lumber
is sold and how a tape reads.

Translations are keyed by the **English source string**, so anything untranslated falls back to English
rather than showing a blank or a raw key. Three self-tests guard the layer: no translation may lose a
`{n}` placeholder (that would silently swallow a number), switching language must not change a single
computed dimension or validation outcome, and a `MISSING_T` set records any string asked for in French
that has no entry — currently empty across every input combination.

To add or fix wording, edit the `FR` object near the top of the file: `"English source": "French"`.

## Two inputs people mix up

**`span` (stock library) vs `Stock lengths you can buy` — unrelated.**

| | What it is | What it drives |
|---|---|---|
| **`span`** | *Structural.* The largest clear distance allowed between supports when that board is walked on. 5/4x6 = 16", 2x6 = 24". | How **many** joists / stringers you need, and their spacing. Note it is the **tread** material's span that spaces the stringers, not the stringer's own — 5/4x6 treads force 16" o.c. even though the 2x12 under them is good for 24" |
| **`Stock lengths`** | *Commercial.* The lengths the yard sells — 96, 120, 144, 192 in (8/10/12/16 ft). | The **buy list** and how pieces are cut from each board |

## Pricing follows the length

A 16' 2x12 is not four times a 4' one, so price is keyed to **(stock, length)**, not to stock alone.
Three forms in the `Prices` field, most specific wins:

| Form | Means |
|---|---|
| `2x6@96:12.50` | that stock **at that length** |
| `2x12:2.10/ft` | by the **foot** — `length ÷ 12 × rate` |
| `5/4x6 PT:11.25` | one **flat** price whatever the length |

A bare `@length` is **inches**, like the `Stock lengths` field it pairs with — write `@2438mm` if you
mean metric. (Reading it in the *display* unit meant switching to mm silently made every price miss.)

The buy table shows the unit price, the line total, and **where each price came from** — `your price`,
`2.10/ft`, `flat, any length`, or `estimated from your 96" price` when a length you did not list is
scaled from the nearest one you did. An unpriced stock reads `no price given` and contributes nothing,
rather than silently costing zero and making the total look complete.

### Seeded prices

The field ships filled in from **Home Depot Canada, GREENFIELD PARK #7152** (500 Auguste, Greenfield
Park QC), read once on **2026-08-07** and pasted in as plain text. **The tool never fetches anything** —
it is a single offline file. Check them before you buy; lumber moves.

| Stock | 8' (96") | 10' (120") | 12' (144") | 16' (192") |
|---|---|---|---|---|
| 5/4x6 PT | 7.09 | 8.89 | 10.69 | 12.49 |
| 2x4 | 7.19 | 8.89 | 10.79 | — |
| 2x6 | 10.29 | 12.89 | 15.49 | 20.69 |
| 2x8 | 17.49 | 21.89 | 26.29 | 35.19 |
| 2x10 | 23.79 | 29.79 | 34.79 | 46.49 |
| 2x12 | 32.13 | — | 48.19 | — |
| 4x4 | 15.69 | 20.29 | 23.59 | — |
| 1x6 | 5.30 | — | — | — |
| 1x8 | 10.37 | — | — | — |

Blank cells are lengths that store does not stock. These numbers are also the argument for the
per-length model: **5/4x6 PT runs 88.6¢/ft at 8' but only 78.1¢/ft at 16'**, while 2x6 goes the other
way (1.286 → 1.293). Costing a 16-footer as two 8-footers is wrong in both directions.

`5/4x6 composite` is deliberately unpriced — it is a different product line, not a lumber SKU. A save
made before this existed picks the list up automatically **only if its price field was empty**; a price
you typed yourself is never overwritten.

**Total rise is measured finished surface → finished surface:** from the top of the platform's
**decking** down to the top of the existing deck **boards**. Not to the framing, and not to the
platform's frame.

**Where the framing bears is a separate question** (`Structure bears on`):

* **The deck surface** — the framing sits on top of the existing decking.
* **The deck structure** — the decking is cut away or notched so the framing lands on the joists.

This **never changes the stair geometry**. Riser height, tread heights and total rise are identical in
both modes, because rise is always finished-to-finished. What changes is only the framing that has to
reach down to bearing, one deck-board thickness lower:

| | Surface | Structure (1" decking) |
|---|---|---|
| Riser height `r`, tread heights | 7" / 21, 14, 7 | **identical** |
| Stringer board length (3 notches) | 42.07" | 42.60" (`+ deckT·r/Ls`) |
| Bottom box rim height | 5½" | 6½" (other boxes unchanged) |
| Platform frame height | 27" | 28" |
| Inner-rim blocking | to the surface | 1" longer |

The bottom box and the platform frame get deeper, so the tool re-checks whether they still come out of
your chosen stock and says so if they don't.

## Coordinate system and naming

```
        house wall  (y = 0)
   ┌──────────────────────────────┐
   │                              │
   │          PLATFORM            │  platY  (depth out from the wall)
   │                              │
   └──────────────────────────────┘ ── front face, steps descend in +y
   │            platX             │
   x = 0                      corner (platX, platY)
   (far side face)                 side face, steps descend in +x
```

* **`platX`** — platform width measured along the house wall.
* **`platY`** — platform depth measured out from the wall.
* **Front leg** — the flight descending in +y; its boards run along x; width base `Wb = platX`.
* **Side leg** — the flight descending in +x; its boards run along y; width base `Wb = platY`.
* `R` = unit run (going), `n` = nosing, `H` = total rise, `N` = riser count, `r = H/N`.

---

## Geometry (what the tool actually computes)

**Levels.** `N` risers, `N − 1` step treads. Tread `k` (k = 1 is the top step, k = N−1 the bottom step)
has surface height `z_k = H − k·r`, so the bottom tread sits at `r` above the deck. Every level is
derived from exact `r` — the tool never rounds a riser and then accumulates, so the last riser cannot
drift out of tolerance.

**Tread ring `k`.** An L-shaped ring: inner rectangle `(platX + (k−1)R) × (platY + (k−1)R)`, outer
`(platX + kR + n) × (platY + kR + n)`. The **miter line** is `y = x + (platY − platX)`, running from the
inner corner to the outer corner. Because that line is at 45°, each leg's tread area is a **trapezoid** —
which is why every board on a tread is a different length.

**Tread boards.** Tread depth `T = R + n`. Boards run parallel to their leg's face. For board `j` at
offset `o_j` from the tread's inner edge (offsets accumulate board width + gap):

* *Mitered corner* — long point `= Wb + (k−1)R + o_j + w_j`, short point `= Wb + (k−1)R + o_j`, one 45°
  end cut. Consecutive boards therefore differ by exactly one board pitch (width + gap).
* *Through leg* — the through leg's boards are all `Wb + kR + n`; the butting leg's are all
  `Wb + (k−1)R`. Every cut square.

**Risers.** `N` risers; riser 1 is the platform's own face, riser `N` the outermost. Riser `k` lies at
plane offset `(k−1)R`, so its mitered long point is `platX + (k−1)R` (front) or `platY + (k−1)R` (side),
with the short point one board thickness less. Riser board height is `r − tread thickness` when captured
between treads, or the full `r` when it laps the tread edge.

**Cut stringers.** Stringers sit at spacing ≤ the tread material's span. A stringer at width position `p`
on a leg only supports levels `k ≥ k_min`, where `k_min = max(1, ⌈(p − Wb)/R⌉ + 1)` — **stringers nearer
the corner carry fewer notches**, which is what makes the pyramid work. Three kinds come out of that:

| Trade name | ID | What it is | Top end |
|---|---|---|---|
| **Common** | `F1…`, `S1…` | reaches the platform, runs the whole flight | the connection type you picked (below) |
| **Jack** | `F6`, `S3`… | the 45° miter cuts it short, so it starts partway down | plain plumb cut against the framing of the step above — needs a cleat, hanger or blocking there; the connection type does **not** apply to it |
| **Hip** | `H` | the diagonal at a **square** corner, run `R√2` | same as a common |
| **Corner hip** | `H1`, `H2` | a **tapered** corner's two joint lines, run `hyp(R−dc, R)`. Each is one board from the platform corner to the deck | same as a common |
| **Corner face** | `C1…` | perpendicular to a tapered corner's face, run `g`, breaking up the span between the two hips | same as a common |

IDs are numbered **after** the ones that carry nothing are dropped, so they always read `1…n` with no
gaps — a `C2` with no `C1` is just confusing.

**Every stringer is identified.** `F1…Fn` on the front leg, `S1…Sn` on the side leg, numbered outward
from the house wall, `H` for the hip. The same ID is tagged on the **plan view**, heads its **sheet and
cut schedule**, and names its piece in the **cut list** (`F6 — jack stringer, 1 notch @ 12-1/4"`), so a
board on the pile traces back to a line on the drawing. Sheets covering a run of identical boards
collapse to `F1–F5`.

**The cut schedule.** Every distinct board gets its own sheet — the cut nested on the stock you buy, the
finished profile, and then the numbers, all generated from one function (`stringerBoardLayout`) so a
number can never contradict a drawing. Two coordinate systems are in play:

* **profile** — `x` out along the stair, `y` up. The nosing line is `y = x·r/R`.
* **board** — `x'` along the board from the bottom end, `y'` down from the **top edge**, which *is* the
  nosing line: `x' = (x·R + y·r)/Ls`, `y' = (x·r − y·R)/Ls`.

In board coordinates every notch is the **same right triangle sitting on the top edge**: hypotenuse
`Ls = √(r² + R²)` along the edge, legs `r` and `R`, apex `r·R/Ls` deep and set back `r²/Ls` from its
mark. So the sheet gives you, per board:

* framing square: rise `r` on the tongue, run `R` on the blade; pitch `atan(r/R)`
* **marks measured from one datum** — the bottom end of the board — at `startOff + i·Ls`, not stepped off
  one from the last, which accumulates the square's own thickness
* which level each seat carries, and that tread's height above the deck
* the cuts in the order you make them, each with its size and why: bottom riser face (the "drop"),
  bottom seat, the `c` tread cuts, the `c` riser cuts, and the top end
* throat, minimum board width for the pitch, cut length, material past the last mark, board to buy

The **bottom** is a plumb face `drop = r − tread thickness (+ base)` below the lowest seat, then a level
cut running out to the board's lower edge — not a line to the corner of the board. The **hip** has the
same rise `r` but run `R√2`, so it bites deepest and keeps the least throat; its seats are square across
the face and it is the tread boards landing on it that are cut at 45°.

### The top notch has no riser face

Without a tab, the **topmost triangle taken off the edge has its vertical leg in the very plane the
board is cut off in**. Approach the apex `(c·R, c·r)` from inside the board and there is nothing there —
the material above the top seat belongs to the notch. So the board's top corner is the top **seat's**
back corner `(c·R, (c−1)·r)`, and one plumb cut runs from it down to the lower edge:

* board length is `c·Ls − r²/Ls + drop·r/Ls`, i.e. **`r²/Ls` shorter** than the naive `c·Ls` (≈4" at 7/11)
* the top plumb cut is `bw·Ls/R − r` tall, one rise shorter than the full plumb depth of the board
* the last mark, `mark c`, falls on the **offcut** side of that cut — you still need it to lay the cut
  out, but it is past the end of the finished board, and the schedule says by how much

Carrying the outline up to the apex and back down the same line drew a **zero-width spike** above the
board, which is the stray thin vertical line that used to appear on jack stringers and on every
*top tread down* board. A flush tab is different: there the last riser face is real — it is the front of
the tab — so the apex is material and the board is `tab·R/Ls + r²/Ls` longer than the tabless one.

### Boards that do not match the stair

Two mismatches used to make the outline cross itself, and both are now reported instead:

* **too short** — a one-notch jack out of a 2x12 is deeper than its own run, so the lower edge dives
  past the bearing plane before the top cut. The level cut stops *at* the plumb cut.
* **too narrow** — the lower edge never reaches the bearing plane, so there is no bottom level cut at
  all and the board has to be packed up. The schedule gives the packing.
* **un-cuttable** — when the notch is deeper than the board (`r·R/Ls ≥ board width`) it cuts clean
  through. The sheet refuses to draw it and says why, rather than rendering an impossible shape.

**Box frames.** One L-shaped box per level, rim height `r − tread thickness`, outer rim mitered at the
corner, joists spanning `R` at the tread material's spacing, plus a 45° corner block. Load path: box `k`'s
outer rim bears on box `k+1`'s tread directly over box `k+1`'s **inner** rim (keep that rim continuous or
doubled); box `k`'s inner rim needs support blocks down to the deck, except the top box (fastens to the
platform frame) and the bottom box (sits on the deck).

**Board fitting.** For both tread depth and riser height the tool reports every option: exact fit,
`n` full boards plus a **rip** of the remainder, `n+1` boards **overshooting** by X (absorb it into the
nosing), a **tightened gap** that fills the depth exactly with no rip, or an adjusted gap if that lands
in a sane range. Riser 1 is reported separately when it differs — it tucks under the platform decking
rather than under a tread board, so a different decking thickness makes it a different height.

**The platform is the top tread.** Its decking overhangs the two faces that have steps below them by
the tread nosing, exactly like every other step. `Platform width/depth` are **frame** dimensions; the
finished size is `frame + nosing` on those two faces, and the decking boards are cut to match.

## Tapered corner steps

`Corner step shape` switches the corner between **square** (the 90° corner, carried on a hip stringer or
mitered rims) and **tapered** — a 45° clip that *widens* as the stair descends. The platform is
untouched either way: its corner stays a square point, and it is the apex the taper opens out from.

The whole thing is **one parameter**, `dc` — how much more the clip takes off each step. Two numbers
fall out of it and nothing else needs a special case:

```
joint slope     m = (R − dc)/R        dx per dy along the tread/rim joint
diagonal going  g = (2R − dc)/√2      depth of the corner step, face to face
```

`dc = 0` is the square corner: `c_k ≡ 0`, `m = 1`, 45° miters, no diagonal face at all. So every
formula in the tool is written **once**, with the square corner as `dc = 0` — there is no parallel code
path to drift out of step, and a test asserts the tapered code reproduces the old square numbers exactly.

### Why "match the legs' going" is the setting worth having

| `dc` | Corner face, 3 steps at R = 11" | Going across it |
|---|---|---|
| 0 — constant clip | 17", 17", 17" | 15-9/16" (395 mm — **over** the 355 mm max) |
| `R(2−√2)` ≈ 0.586R | 9-1/8", 18-1/4", 27-5/16" | **11"** — same as the legs |
| R | grows with the ring only | 7-3/4" (shallower than the legs) |

At `dc = R(2−√2)` four things line up at once, and none of them are coincidences:

* the corner step is **exactly as deep** as the legs, so it is not a different depth underfoot;
* the corner treads use the **same board layout** as the legs (`TbDiag = Tb`), same rip, same widths;
* the joint lands on the **true bisector** of the 135° corner, so every corner cut is a single
  **22.5°** setting — the octagon detent on any mitre saw, not a compound guess;
* the corner stringers become the **same board as a common leg stringer** — same notch, same throat.

That last one is the real prize. The square corner's hip needs a run of `R√2` = 15-9/16", which bites so
deep it often will not come out of a 2x12 at all — the tool currently errors on it at steeper pitches.
A tapered corner has no 90° point to hip, so the hip simply **does not exist**.

### What it changes elsewhere

* **Three faces per ring**, so a third riser board (mitered *both* ends) and a third band of tread
  boards, running parallel to the diagonal. They are trapezoids, because the two joint lines diverge.
* **Ring 1's corner is a triangle** — its inner edge is the platform's square point, so the innermost
  piece tapers to nothing and is flagged as a triangle rather than listed as a board.
* **Both legs get shorter.** A leg's outer end now advances `R − dc` per step, not `R`, which also means
  a leg stringer at a given position starts one level later than it used to.
* **Two hips** (`H1`, `H2`) on the joint lines — see below — plus **face stringers** (`C1…`) running
  perpendicular to the face to break up the span between them. Interior only: the face's edges *are* the
  hips. The tool reports the worst clear span the corner treads cross and warns past the tread stock's.
* Code is re-checked on the diagonal: `g` against the run min/max, and `g + nosing` against the tread
  depth minimum. It also warns if the clip has eaten so far into the legs that little straight run is
  left, and errors if it has eaten them entirely.
* `Corner tread joint` (miter / through / compare) is hidden — a tapered corner has no square butt to
  run a board through, so the joint is always the mitered one, at the taper's own angle.

### Three runs at a corner, one ring growth

A corner has three different "runs" and they are measured along three different **directions**. All
three fall out of the same ring growth — the ring advances `R` square to the leg face, and `R − dc`
*along* it once the clip is eating in:

| | Formula | At 7/11 | ×R | Measured |
|---|---|---|---|---|
| Leg step | `R` | 11" (279 mm) | ×1.000 | square to the leg face, nosing to nosing |
| Corner hips `H1 H2` | `hyp(R − dc, R)` | 11-15/16" (302 mm) | ×1.082 | along the joint line |
| Corner **face** | `(2R − dc) ÷ √2` | 11" (279 mm) | ×1.000 | square to the 45° face |
| *Square corner's hip* | `R × √2` | *15-9/16" (395 mm)* | *×1.414* | *along the diagonal* |

The face one comes from the face lines themselves: consecutive faces are `x + y = S_k` with
`S_k = Wx + Wy + 2kR − c_k`, so consecutive `S` differ by `2R − dc`, and a gap of `ΔS` between two lines
of slope −1 is `ΔS/√2`. A test checks the reported number against the gap **measured off those lines**,
not just the formula restated.

**Set `dc = 0` and all of it collapses:** `hyp(R,R) = 2R/√2 = R√2`. The square corner's single hip is the
same two formulas at zero clip growth — which is the whole argument for the taper, because `R√2` is
1.414× the leg run and 395 mm at 7/11, over the 355 mm code max.

Notch depth for any of them is `r × run ÷ hyp(r, run)`, which is why a longer run bites deeper and is
what decides whether the board comes out of a 2x12. The Framing tab shows this table with your numbers.

### Plan cuts on an angled stringer — and a top view

A leg stringer is square to its face, so **every** cut on it is square in plan. A corner or hip stringer
sits at an angle, and exactly one thing about it stops being square — and only for one connection type.

**The notches stay square.** A seat is a horizontal cut and a riser face is a vertical one, both straight
across the board. It makes no difference that the tread landing on the seat arrives at 45°. Same for the
bottom level cut: it bears flat on the deck, square across. That is the part people expect to be
complicated, so the tool says it outright.

What is not square is the **top end**, and only when it butts:

| Connection | Top end | Why |
|---|---|---|
| **Flush half / full** | **90°, square** | the tab rides *over* the rim, like a rafter over a plate, and ends in free air inside the platform — nothing butts anything |
| **Top tread down** | a **90° vee** | the end butts the platform's outside corner from outside, so the vee's two faces *are* the two platform faces |

For the vee, the apex lands on the platform's corner point at mid-thickness, and the two saw settings
come out as **φ and 90 − φ**, where φ is the board's plan angle:

| Member | φ | Saw settings | Horns past the apex |
|---|---|---|---|
| Square corner hip `H` | 45° | 45° / 45° — symmetric | 3/4" / 3/4" |
| Taper face stringers `C…` | 45° | 45° / 45° — symmetric | 3/4" / 3/4" |
| Taper hips `H1 H2` | 22.5° | **22.5° / 67.5°** — lopsided | 5/16" / 1-13/16" |

67.5° is past a mitre saw's range, so the tool says to cut the 22.5° face on the saw and the other by
hand, or notch the end — it only has to clear the rim.

Each corner and hip stringer sheet carries a **plan-cut schedule** (what, where, what angle, why) and a
**top view**: the board seen from above so an angled end shows as a line across the thickness, full
length with the layout marks, plus the top end enlarged against the platform corner it has to fit.

The cut list's angle column now reflects the connection type too — it used to claim a "45° plan bevel"
on the hip regardless, which was wrong for two of the three connections.

### Long point vs short point — the joint angle is in the length

Every length the tool reports is the **long point**, the longest corner of the board, and that is what
the cut list buys. The short point is the other corner of the same mitered end, and the gap between them
is set by the joint **angle**, not by the length:

```
long − short = board width × m          m = (R − dc)/R = the joint slope
```

| Joint | `m` | On a 5-1/2" board | On a 1" riser |
|---|---|---|---|
| 45° (square corner) | 1 | **5-1/2"** — the full board width, the classic rule | 1" |
| 22.5° (matched taper) | 0.4142 | **2-1/4"** | 7/16" |

The **corner-face** boards are mitered at *both* ends, so they lose twice — `2 × w × 0.4142` = 4-9/16"
on a 5-1/2" board. Riser boards use their **thickness** rather than their width, so they lose far less.
Tests pin all four of those relations, including that the 45° case still gives exactly the board width.

Ring `k`'s joint line — where the leg tread boards meet the corner ones — runs from
`(Wx+(k−1)R−c_{k−1}, Wy+(k−1)R)` to `(Wx+kR−c_k, Wy+kR)`. Its direction is `(R−dc, R)` for **every** `k`,
and ring 1's starts at the platform corner itself. So all of them are segments of **one straight line**
from that corner down to the deck. There are two, mirrored.

That makes them hips, and they get a real notched stringer rather than blocking — a member on the hip
carries the mitered ends of the leg boards *and* of the corner boards at once, which is exactly what
those miters need. `H1` and `H2`:

| | Taper's two hips | Square corner's one hip |
|---|---|---|
| Run per step | `hyp(R−dc, R)` = **11-15/16"** | `R√2` = 15-9/16" |
| Notch depth | 6-1/16" | 6-3/4" |
| Throat left in a 2x12 | 5-3/16" | 4-1/2" |
| Minimum board | **11-1/16"** — a 2x12 works | 11-3/8" — a 2x12 does **not** |
| Plan bevel | 22.5° each side of the 45° line | 45° |

`Rj < R√2` always, so a taper hip always needs less board than a square corner's hip. At 7/11 out of a
2x12 that difference is pass versus fail, and a test pins exactly that.

The corner **face** between the hips is then broken up by the `C` stringers — one on the centreline in
the common case — so the tread boards never cross more than `faceMax ÷ segments`. On a 4-riser stair
that is 13-11/16", inside what a 5/4x6 can span.

> An earlier version framed the joints with two nailers per step instead. That was a workaround for a
> member that should have been there: the nailers had nothing to bear on at either end, because the
> joint line crosses neither a leg stringer nor the centre corner stringer. Replaced by the hips.

## Stair connection at the top — do I need a ledger?

Three ways the stringer can meet the platform. They differ in one thing that matters structurally:
whether the stringer merely **hangs** on the platform face or runs back **under** it and **bears** on the
frame. The sawtooth, riser count, run and footprint are identical in all three — only the top of the
board changes.

| Connection | Bearing on the frame | Ledger | Board length (7/11, 3 notches) |
|---|---|---|---|
| **Top tread down** — tread sits on the stringer | none, butts the face | **required** — ledger, cleat, or stringer hangers (Simpson LSC-type) | 38-5/16" |
| **Flush, half tread** | half a run (5½") | not needed for support | 46-11/16" |
| **Flush, full tread** | a full run (11") | not needed for support | 51-3/8" |

With *top tread down* nothing carries the stringer — the entire load is in the fasteners, in shear —
so a ledger or proper hangers is not optional. The flush options put the stringer on top of the frame,
so fasteners only need to stop it sliding. The Framing tab states which case you are in and why, and
the elevation highlights the bearing surface. Only the stringers that actually reach the platform get
the tab; jack stringers nearer the corner start lower down and are unaffected.

### The tab is a level cut — the last riser stays plumb

The bearing tab is **not** the sloped nosing line carried on past the platform face. It has to slide
under the platform decking and sit flat on the frame, so its top edge is **horizontal**. The nosing line
stops at the face, the last riser face stays **plumb** through a full rise `r`, and only then does the
top edge turn level and run back `tab`. Two consequences:

* **Length.** The far corner of a level cut projects `tab·R/Ls` on the slope, not `tab·Ls/R`. At 7/11
  a half-tread tab adds **4-5/8"**, not 6-1/2" — the sloped version buys ~2" of board per stringer that
  you do not need, and on a marginal board it buys the next stock length up.
* **Two thicknesses set its height.** The tab can sit no higher than the material that is there — the
  nosing line at the face, one tread thickness below the platform surface — and no higher than the
  underside of the platform decking. So it is cut `lift = max(0, platform decking − tread)` below the
  top notch corner. When the treads are the *thicker* stock instead, the tab lands
  `gap = tread − platform decking` **below** the frame top and has to be packed out; the tool says so
  with the number, rather than drawing a joint that does not close.

A self-test walks the actual board outline for both flush types and asserts the last riser is one full
rise at a constant `x`, that the tab leaves that corner level, that nothing sits above the nosing line,
and that the tab measures exactly the advertised bearing.

## Will it come out of a standard 2x12?

For cut stringers the tool reports, for the legs and for the hip separately:

* **notch depth** = `r · cos θ` = `r·R/√(r²+R²)`
* **throat** = board width − notch depth, checked against your minimum
* **minimum board width** = notch depth + throat minimum — the number that tells you whether a 2x12
  (11¼") is actually enough
* longest stringer vs. the longest stock length you buy

At 7" / 11" the **legs** are fine — notch 5-29/32", throat 5-11/32", needs a 10-29/32" board. The **hip**
is not: its run is `R·√2` = 15-9/16", so the notch bites 6-3/8" deep and leaves only 4-7/8" of throat,
needing an 11-3/8" board. A 2x12 is ⅛" too narrow, so the tool raises it as an error with the options
(double the hip, wider stock/LVL there, accept a smaller throat, or shorten the run). This is inherent
to a mitered corner, and it is the main reason box frames are the easier build for this stair.

## Drawings

All drawings are to scale, with a scale bar, and show real material sizes rather than single lines:

* **Plan view** — every tread board at its true width, with your real gap and its 45° mitered (or
  square-cut) end. The platform decking is drawn too. Framing hidden below the treads — box rims and
  joists, or stringers plus the diagonal hip — is shown dashed, hidden-line style.
* **Elevation** — a true section through the front leg: tread boards in cross-section (so you see the
  gap between them), riser boards, box rims with their real height, blocking down to the deck, the
  platform frame and its decking, and the existing deck surface. In stringer mode the actual sawtooth
  stringer outline is drawn, including its lower edge, top plumb cut and bottom level cut. The platform
  is drawn cut short with a break line so the steps stay legible.
* **Framing only** — the plan view again, same scale and orientation, with every tread board and the
  platform decking taken away and the framing drawn as the real stock it is rather than dashed hidden
  lines. Rims, joists, stringers, blocking, nailers, the platform frame and its legs, all at true
  thickness, with the level tags and stringer IDs on top. This is the sheet you build from.
* **Box frames — one drawing per level** (box mode). The plan view shows the *finished* stair, so the
  boxes under it are hidden lines and you cannot see what to build. This draws each box on its own, as
  the boards it is made of: outer rim, inner rim, the joists between them, the corner block, and the 45°
  miters where the two legs meet at the **outside** corner `(Wx+kR, Wy+kR)` and the **inside** one a run
  back. Each rim is therefore a board with a mitered end, not a rectangle butted at the corner: the
  front outer rim's long point is at `Xo` and its short point one thickness back. Every rim, the joist
  count/length/spacing and the rim height are called out, and the step above is shaded for context.
  It uses the **same orientation as the plan** — house wall along `y = 0` at the top, steps descending
  toward the bottom (+Y), side leg off to the right (+X), miter running top-left to bottom-right. A
  drawing meant to explain the plan cannot be mirrored against it, so a test asserts both put the wall
  in the top third and run the miter the same way.
* **Terminology** — a labelled schematic of two steps at your actual rise and run, defining total rise,
  unit rise, unit run/going, tread depth, nosing, tread board, riser board, box rim/stringer, platform
  decking and frame, and the bearing point where each rim lands over the rim below.
The stringer is drawn at its **true board width**. The board's top edge is the *nosing line* — the line
through the **front** corner of every seat, `(Wy + kR, H − kr − tt)` — which extrapolated back to the
platform face sits at `H − tread thickness`, one riser above the first seat's back corner. Offsetting
the lower edge from that back corner (the notch apex) instead draws a 2x12 half again too deep, since
the apex line lies `r·cos θ` below the real edge. The lower edge is `bw` perpendicular below the nosing
line, i.e. `bw·Ls/R` down on a vertical cut, which puts every notch apex exactly one *throat* above it.

With *top tread down* the stringer's **top end is a plumb cut in the plane of the platform face**,
bearing against the rim — not a diagonal back to the corner of the board. That plumb face is
`board width × Ls / run` tall, which is *taller* than the board is wide (13-3/8" for a 2x12 at 7/11),
because a vertical cut across a sloped board is longer than the board's width. Same detail as a
standard cut stringer hung off a rim. With a **flush** connection the last riser face is still plumb
through a full rise; the top edge then turns **level** and runs back `tab`, and the plumb cut moves to
the back of the tab, where it is `bw·Ls/R − lift − tab·r/R` tall. See *the tab is a level cut* above.

* **Stringer stock layout** — the board you *buy*, laid flat, with the cut nested inside it and the
  offcut shaded at the end. Profile coordinates are rotated into board coordinates, where the nosing
  line **is** the board's top edge, so every notch becomes a triangle on that edge one `Ls` apart and
  the framing-square marks read straight off the board. It also nests as many stringers per board as
  actually fit, so the sheet agrees with the cut list instead of implying one board per stringer.
* **Stringer sheets** — one sheet per *distinct* board in the job (the ones nearer the corner carry
  fewer notches, so they are not all the same board), plus the hip at its `R·√2` run. Each shows the
  real board outline: notches, nosing line, a layout tick every `√(r²+R²)` along the edge, the top plumb
  cut, and the bottom **level cut** — horizontal, because it bears flat — set `drop` below the last
  seat, where `drop = r − tread thickness (+ decking thickness when bearing on the structure)`. Below
  the two drawings, that board's **cut schedule** — square settings, marks from one datum, every cut in
  order with its size and reason, and what each seat carries.

A colour legend above the drawings maps each fill to the stock you selected.

## Navigating

The **tab bar is pinned** under the header, so you can jump between Summary / Plan & elevation / Step
schedule / Framing / Per-step sheets / Cut list / Warnings from anywhere in a long sheet without
scrolling back up. Switching tabs lands you at the top of the new panel; clicking one while you are
already at the top doesn't move anything. The input column is pinned the same way. Both offset by a
`--hh` CSS variable that a `ResizeObserver` keeps equal to the real header height — the header wraps to
two or three rows on a narrow window, and a hardcoded offset would let content slide under it. Printing
hides the bar, as before.

### Known limitations

Two label-placement issues survive, both pre-dating the current work and both confined to
non-default settings. Everything else is clean across 576 swept configurations (64,752 labels).

* **Both units mode** — a few dimension labels in the plan and elevation run past the edge of the
  canvas, because the gutters are sized for a single unit and `12-1/2" / 318 mm` is more than twice as
  wide. Same for the level tags on an 8-riser stair in mm.
* **2x10 stringer stock** — in the elevation the connection note can land on a level tag, and on a
  9-riser stair two level tags can collide. A 2x12 (the default) is clean.

---

## Code limits — read this

The riser/run/tread limits are the CNB / *Code de construction du Québec* values for stairs serving a
single dwelling unit. **They are editable in the sidebar on purpose.** Verify them against the current
code text and your municipality before cutting anything — requirements vary, and an exterior stair on a
deck may also trigger guard and handrail requirements that this tool only *flags*, never sizes.

This calculator is a layout and quantity aid, not an engineering document. It does not check the existing
deck's capacity to carry the new platform and stair loads.

---

## Files

* `stairs-calculator.html` — the whole tool (markup, styles, logic, drawings, self-tests).
* `README.md` — this file.
