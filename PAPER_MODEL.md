# Paper fold model

Working notes toward a formal model of the paper: point labels, which
face is which, and what a fold actually does to position + visible
face + layer order. Goal: stop re-deriving "which face shows here" by
trial and error against screenshots (see the flower-base peek-line saga
in the flower.html commit history for exactly the failure mode this is
meant to prevent), and have a model precise enough to check a fold
diagram against before touching Three.js code.

This is a companion model, not a replacement for flower.html's actual
3D math. flower.html already computes exact 3D positions through a
fold sequence (FLAT, COLLAPSED_UP, SETTLED, foldedCorner, etc.) and
that stays as-is. What's missing there — and what caused every round of
"still not it" on the peek line — is a way to reason about *which face
is exposed* and *what's layered underneath* independent of eyeballing
a render. That's what this model is for.

## 1. Point numbering

From the reference photos: label the flat square 1–9, where 1 is the
center and 2–9 go **clockwise around the perimeter starting at the
bottom-right corner**, alternating corner/edge-midpoint:

```
   6───────7───────8
   │               │
   │               │
   5       1       9
   │               │
   │               │
   4───────3───────2
```

Corners are the even numbers (2, 4, 6, 8); edge-midpoints are the odd
numbers other than 1 (3, 5, 7, 9). The rule "just count 2 through 9
clockwise from the bottom-right corner" is the whole convention — no
compass directions needed, which sidesteps the north/south confusion
below.

Square coordinates for reference (side 2, centered at origin, +y up on
the page as drawn):

| # | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|---|
| x | 0 | 1 | 0 | -1 | -1 | -1 | 0 | 1 | 1 |
| y | 0 | -1 | -1 | -1 | 0 | 1 | 1 | 1 | 0 |

## 2. Mapping to flower.html's current names

flower.html names points `Ctr`, corners `A/B/C/D`, edge-midpoints
`N/E/S/W`, matching `OUTER = ['A','N','B','E','C','S','D','W']` (a
cyclic ring — consecutive entries are adjacent). Matching `FLAT`
coordinates to the table above (its X,Z plane against the numbering's
x,y) gives:

| flower.html | Ctr | A | B | C | D | N | E | S | W |
|---|---|---|---|---|---|---|---|---|---|
| number | 1 | 4 | 2 | 8 | 6 | 3 | 9 | 7 | 5 |

Checked against `CREASE_NEIGHBORS` and `PRIMARY_NEIGHBOR` in the code
(each corner's two ring-neighbors, and which one survives collapse as
that corner's primary layer):

- `A:[N,W]` → 4's neighbors are 3, 5 ✓. Primary `N`→3.
- `B:[N,E]` → 2's neighbors are 3, 9 ✓. Primary `E`→9.
- `C:[E,S]` → 8's neighbors are 9, 7 ✓. Primary `S`→7.
- `D:[S,W]` → 6's neighbors are 7, 5 ✓. Primary `W`→5.

All four check out, so the table is self-consistent with the code as
it stands today. **Important:** flower.html's letters N/E/S/W do *not*
correspond to compass directions in the numbering's layout above (code
`N` lands at number 3, which is the *bottom* edge in the diagram) —
match points by this table, not by the letter names.

## 3. The two faces: Color and White

The paper has two faces. Call them:

- **Color** — the customizable, normally-visible side. This is what's
  numbered in the reference photos (the numbers are written directly
  on the colored paper), and what flower.html shows on every panel
  once the base has collapsed (via `aForce` forcing `uBack`).
- **White** — the plain side, normally hidden. This is flower.html's
  `uFront`.

**flower.html's `uFront`/`uBack` naming is backwards from how a reader
would guess** — `uFront` is the White face, `uBack` is the customizable
Color face that's normally *visible*. Worth a rename to something like
`uWhite`/`uColor` at some point; noting it here so it doesn't cause a
misreading the way it nearly did in this session.

Starting state: the flat square lies Color-face-up (numbers facing the
viewer), White-face-down.

## 4. The fold operation

A fold is:

```
Fold = (crease, movingRegion, angle, sense)
```

- **crease** — the fixed hinge line (two or more point-labels that
  don't move).
- **movingRegion** — the flap on one side of the crease; everything
  else, crease included, stays put for this operation.
- **angle** — rotation applied to `movingRegion` about `crease`, 0° to
  180° (180° = fully folded).
- **sense** — `valley` (flap rotates toward the viewer, lands *above*
  whatever paper is already at its destination) or `mountain` (rotates
  away, lands *below*).

Consequences of folding by the full 180°:

1. **Position**: for a flap starting coplanar with the rest of the
   sheet, a 180° hinge fold about an in-plane crease reflects the
   flap's points across the crease line, within that original plane
   (it's a mirror image, not a general 3D placement) — matches
   flower.html's own `foldedCorner`, which reflects the corner's ray
   across the crease's bisector direction.
2. **Face orientation flips**: rotating 180° about an axis that lies
   *in* the flap's own surface flips its outward normal. Whatever face
   was exposed (Color) is now hidden against whatever it landed on,
   and the *other* face (White) is what a fixed external viewer now
   sees on that flap. This holds regardless of valley vs. mountain —
   sense only decides which side of the existing stack the flap tucks
   into, not whether its visible face flips.
3. **Layer order**: at any point where the moved flap now overlaps
   existing paper, insert it above (valley) or below (mountain) the
   existing stack at that point.

**Caveat that matters a lot in practice**: rule 2 is stated for a
single ply of paper. Once earlier folds have already stacked multiple
plies on top of each other, a later fold that moves the *whole stack*
as one rigid unit (crease through all plies at once, no separating
them first) flips the stack's two *exterior* faces, not "the" face of
some single ply inside it — see §6, where this distinction is the
whole answer.

## 5. Worked example: a single diagonal precrease fold

Fold corner 8 down onto corner 4 (a standard preliminary-base
precrease step): crease is the *other* diagonal, `6–1–2` (reflecting
across that line maps (1,1)→(-1,-1), i.e. 8→4, by direct reflection
across `y=-x`). `movingRegion` = the triangle `6-7-8-9-2`. Valley
sense (folding the corner toward the viewer, the usual convention).

Before: whole sheet Color-up.
After: points 7, 9 land on 5, 3 respectively; 8 lands on 4. The moved
triangle's visible face is now **White** (it flipped 180°), sitting
*above* the stationary half (valley). This matches ordinary experience
folding colored kami paper — a valley-folded flap shows its white
crease on top.

## 6. Resolved: why the corner-fold wedge stays colored

Checked against the physical model: folding a corner's loose tip down
partway, the moving wedge **stays colored** — it does not show white.
flower.html's `aForce` behavior (forcing `uBack`/Color universally,
including on the actively-folding corner) was right; §4's naive
single-ply flip rule was the thing that was wrong here, for the reason
the caveat in §4 flags.

The resolution is the standard preliminary-base construction itself.
Per flower.html's own comments, the sheet is turned over between the
horizontal/vertical creases and the diagonal ones specifically so
every crease is an easy valley fold instead of forcing two of the four
to be the harder-to-see mountain kind — and the well-known side effect
of that turn-over trick is that **every flap of the collapsed base
ends up 2 plies thick, White-to-White, Color-out on both exterior
faces.** Nothing patchwork about it: the whole base reads as one
uniform color from outside no matter which flap or which of its 2
layers you're looking at, on purpose, by construction.

The `precrease-corner` step folds a flap's loose *tip*, without
separating its 2 plies first — a crease straight through both layers
at once, the whole flap moving as one rigid 2-ply unit (the "actually
opening and pressing each flap flat" move that *does* separate the
plies is later, the squash fold itself). Per the §4 caveat, what flips
is the *stack's* two exterior faces — and since this stack already
reads Color on both exterior faces before the fold (by the paragraph
above), flipping it exposes the *other* exterior face, which is also
Color. Same color both before and after, not because nothing moved,
but because a symmetric stack's mirror image is still symmetric.

This is a real, general modeling point, not a one-off fix: **whether
a fold changes the visible color depends on the full ply structure at
that point, not just "did something rotate 180°."** A model that only
tracks single-ply orientation (§4's naive version) gets this specific,
very common case backwards. Concretely: it's the difference between
folding a *flap's tip* (plies move together, color unchanged here) and
*opening and pressing a flap flat* (plies separate, and that's where
this model would predict white legitimately showing — worth checking
against real paper specifically when that squash-fold step is built).

## 7. Next steps

- Extend §4/§6's ply-stack accounting into a proper primitive: a point
  should carry a *stack* of plies (each with its own Color/White
  orientation), not just one face. A fold either moves an entire local
  stack together (§6 case) or explicitly separates plies within it (a
  squash/petal fold) — worth modeling both as distinct operations
  before flower.html's actual squash fold gets built, and checking the
  prediction against real paper before writing any render code for it.
- Decide whether to extend this model far enough to formally re-derive
  the collapse step itself (four creases folding simultaneously, plus
  the turn-over) — a harder, multi-crease layer-ordering problem,
  worth scoping separately rather than folding into this doc.
- If this model holds up, the natural next step is a small
  Three.js-independent module (pure functions over point labels) that
  can answer "what face shows at point X after fold sequence Y," which
  flower.html's rendering code — or a future fold-diagram importer —
  can be checked against instead of re-deriving it by hand each time.
