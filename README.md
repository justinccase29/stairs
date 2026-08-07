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
`Restored your last design` on open.

If a browser blocks storage (private windows, some `file://` configurations), the header says so
**instead of failing silently**, so you know to use `Export JSON`. Storage is also flushed on page hide
and unload as a safety net.

One caveat worth knowing: `localStorage` is **per origin**. The published copy at
`justinccase29.github.io` and a local copy opened from disk keep *separate* saved designs — that is the
usual reason a design looks "lost". Use `Export JSON` / `Import` to move one between them.

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

The platform is modelled as a **free-standing (floating) frame built entirely from the platform frame
stock** — nothing is fastened to the house:

* the frame is that stock **on edge**, so its depth is the stock's width (a 2x8 gives 7¼");
* **legs cut from the same stock** make up the remaining height, on a grid no wider than that stock's
  span (a 27" platform on 2x8 → 12 legs at 19¾", 4 × 3 at 24" o.c.);
* legs taller than a few stock thicknesses raise a bracing warning (double into an L at the corners, or
  switch to 4x4 posts);
* if the stock is deeper than the whole platform, it says to rip it instead.

The elevation draws it that way — frame band on legs, not a solid block.

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

| Trade name | What it is | Top end |
|---|---|---|
| **Common** | reaches the platform, runs the whole flight | the connection type you picked (below) |
| **Jack** | the 45° miter cuts it short, so it starts partway down | plain plumb cut against the framing of the step above — needs a cleat, hanger or blocking there; the connection type does **not** apply to it |
| **Hip** | the diagonal at the outside corner, run `R√2` | same as a common |

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

## Stair connection at the top — do I need a ledger?

Three ways the stringer can meet the platform. They differ in one thing that matters structurally:
whether the stringer merely **hangs** on the platform face or runs back **under** it and **bears** on the
frame. The sawtooth, riser count, run and footprint are identical in all three — only the top of the
board changes.

| Connection | Bearing on the frame | Ledger | Board length (7/11, 3 notches) |
|---|---|---|---|
| **Top tread down** — tread sits on the stringer | none, butts the face | **required** — ledger, cleat, or stringer hangers (Simpson LSC-type) | 42-1/16" |
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

### Known limitation

In **Both** units mode a few dimension labels in the plan and elevation can run past the edge of the
canvas, because the gutters are sized for a single unit and `12-1/2" / 318 mm` is more than twice as
wide. Same for the level tags on an 8-riser stair in mm. Imperial and mm-only are clean across every
configuration swept. Not yet fixed.

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
