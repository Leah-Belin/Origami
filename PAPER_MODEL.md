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

## 6. Open discrepancy to resolve: the corner-fold face flip

Running the same reasoning on flower.html's `precrease-corner` step
(fold corner A's loose tip down past the merge point — a single-hinge
180° fold of just the moving wedge) predicts the same thing: before
this fold the wedge shows Color (matching every other panel at that
stage), and a 180° fold should flip its outward face to White.

flower.html's code currently disagrees on purpose — `aForce` forces
every panel to `uBack` (Color) universally past collapse, *including*
whichever corner is actively folding, with a comment claiming this was
"confirmed against real paper." Given this session's peek-line
saga (three rounds of "not it" before landing on the right geometry
*and* the right visible side), I'd like an actual paper check before
trusting either side of this: fold one corner of the physical model
partway and look at the wedge that's moving mid-fold — does its
outward face read as still-colored, or does it show white partway
through the fold?

If it does flip to white, the current blanket `aForce=1` is a
simplification papering over a real color change (pun intended), and
letting `aForce` follow real front/back facing for the actively-folding
corner specifically (while keeping the rest of the base pinned, per
the existing comment about avoiding a wide wrong-colored-arc glitch
elsewhere) would make the render match the model above.

## 7. Next steps

- Confirm §6 against the physical model.
- Once confirmed, decide whether to extend this model far enough to
  formally re-derive the collapse step itself (four creases folding
  simultaneously, plus the turn-over the code's comments mention) —
  that's a harder, multi-crease layer-ordering problem, worth scoping
  separately rather than folding into this doc.
- If this model holds up, the natural next step is a small
  Three.js-independent module (pure functions over point labels) that
  can answer "what face shows at point X after fold sequence Y," which
  flower.html's rendering code — or a future fold-diagram importer —
  can be checked against instead of re-deriving it by hand each time.
