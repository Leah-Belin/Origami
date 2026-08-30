# Fold Along

An app that animates origami folding instructions step by step, in real 3D.

The near-term goal is: given a described sequence of folds, render each step
as an accurate, physically-grounded animation. The longer-term goal is to let
people upload their own folding instructions and have them rendered
automatically — this repo is the first step toward that.

## What's here now

Two dependency-light pages (Three.js loaded from a CDN, no build step),
sharing one engine pattern:

- `index.html` — a traditional **waterbomb base** from a square sheet: 8
  precrease fold/unfold steps, then the collapse. The collapse can't be
  built from a chain of ordinary hinge folds (that was tried, and proven
  wrong — it drags points together that need to stay apart); instead it's
  modeled as a single 3D target pose, solved from the constraint that every
  triangular panel keeps its original side lengths.
- `flower.html` — a traditional **flower base**, in progress. Same
  precrease as the waterbomb base; the collapse is the same single-vertex
  mechanism with the opposite 4 points converging (edge midpoints meet,
  corners spread as the rim — the standard fact that the waterbomb and
  preliminary bases are the same crease pattern with reversed mountain/
  valley assignment), then a precrease for the squash fold, done on all 4
  flaps at once (fold each loose corner down to the bottom point, then
  unfold). The squash fold itself — opening a flap and pressing it flat
  along that crease to form a petal — is still to come. This was
  originally built (and briefly shipped) as a crane page; a crane's bird
  base needs a different squash — two flaps folded together as one paired
  kite, not one flap at a time — which this page doesn't do, so it's named
  for what it actually produces. A real crane page, with the correct
  paired-flap squash, is still future work; see Roadmap.

Front/back paper coloring on both pages comes from the GPU's own
per-pixel "which side of this triangle is facing the camera" test
(`gl_FrontFacing`), so a real front and back exist even on the untouched
flat sheet, and color can never drift out of sync with the shape or
depend on tracked fold history.

Open either file directly in a browser, or serve the repo and visit it —
no build step required.

## Roadmap

- [x] Waterbomb base — precrease + collapse, verified against real folded paper.
- [~] Flower base — preliminary base done (precrease + collapse, same
      mechanism as the waterbomb base, verified); the squash-fold's own
      precrease is done on all 4 flaps at once (fold each corner to the
      bottom point, then unfold — the engine's first panel-splitting fold,
      one split per flap). Still needs the actual squash: opening one flap
      and pressing it flat along that crease to form a petal, repeated for
      all 4 (a real squash fold needs a flap's two layers to move
      independently, which the engine doesn't support yet).
- [ ] Traditional crane — needs its own page: same preliminary base, but a
      different squash (two flaps folded together as one paired kite, not
      one at a time, per real bird-base instructions), plus reverse-fold
      primitives for the head and tail — reverse folds need rotation about
      an arbitrary axis (not just one through the center), which the
      engine doesn't support yet.
- [ ] Generalize the engine so a new model is a data file, not new code —
      currently the geometry, panels, and fold steps are hand-coded per
      page.
- [ ] Let people upload their own folding instructions (starting with a
      structured description; parsing arbitrary diagram images is a further
      step past that) and have them rendered automatically.
