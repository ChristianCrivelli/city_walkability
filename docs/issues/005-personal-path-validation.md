# 005 — Personal-path validation set

**Status:** 🔲 Open
**Labels:** validation, data-collection
**Depends on:** [002](002-planned-vs-actual-baseline-and-comparison.md)
**GitHub issue:** #5

## Idea

Use routes actually walked (commutes, regular errands) in the study
locations lived in as a hand-verified validation/anchor set for the
Planned-vs-Actual comparison — grounds the analysis in lived experience
rather than only abstract sampled node pairs, and gives the eventual
report a narrative hook.

## What's needed to start

A short list per relevant city of: start point, end point (addresses,
landmarks, or informal descriptions like "home to the train station" are
fine — they'll be geocoded), and optionally a note on why that route is
worth including (e.g. "always avoided this street, no sidewalk").

## Proposed approach

- Geocode supplied points (Nominatim, same stack as the rest of the
  pipeline).
- Snap to nearest graph node, run the Planned vs Actual comparison
  (#002) on each pair.
- Where the *actual* recollection of the route differs from what the
  friction-weighted shortest path suggests, that's worth a sentence in the
  report — either the model is missing something, or it's confirming a
  real avoidance pattern.

## Acceptance criteria

- [ ] Waiting on input: specific start/end points per city
- [ ] Geocoding + snap-to-node helper
- [ ] At least one city with a fully worked personal-path example in the
      report
