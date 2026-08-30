# Fold Along

An app that animates origami folding instructions step by step, in real 3D.

The near-term goal is: given a described sequence of folds, render each step
as an accurate, physically-grounded animation. The longer-term goal is to let
people upload their own folding instructions and have them rendered
automatically — this repo is the first step toward that.

## What's here now

`index.html` — a single, dependency-light page (Three.js loaded from a CDN,
no build step) that animates a traditional **waterbomb base** from a square
sheet: 8 precrease fold/unfold steps, then the collapse. The collapse can't
be built from a chain of ordinary hinge folds (that was tried, and proven
wrong — it drags points together that need to stay apart); instead it's
modeled as a single 3D target pose, solved from the constraint that every
triangular panel keeps its original side lengths. Front/back paper coloring
comes from comparing each panel's live normal to its own flat-state normal,
not from tracked fold history or camera angle — so it can't drift out of
sync with the shape.

Open `index.html` directly in a browser, or serve the repo and visit it —
no build step required.

## Roadmap

- [x] Waterbomb base — precrease + collapse, verified against real folded paper.
- [ ] Traditional crane — needs squash-fold, petal-fold, and reverse-fold
      primitives the engine doesn't have yet (the waterbomb base only needed
      plain hinge folds and one coordinated collapse).
- [ ] Generalize the engine so a new model is a data file, not new code —
      currently the geometry, panels, and fold steps are hand-coded for the
      waterbomb base specifically.
- [ ] Let people upload their own folding instructions (starting with a
      structured description; parsing arbitrary diagram images is a further
      step past that) and have them rendered automatically.
