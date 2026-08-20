# 006 — Generalize pipeline to an arbitrary city (e.g. "Roermond")

**Status:** 🔲 Open — deliberately backlogged
**Labels:** enhancement, future
**Depends on:** [001](001-infrastructure-friction-weighting.md)–[004](004-true-walkability-index.md) landing first
**GitHub issue:** #6

## Idea

A single `analyze_city("Roermond, Netherlands")` (or CLI equivalent) that
runs the full pipeline — fetch, filter, elevation, grade, friction,
metrics, index — for any place, not just the 5 hardcoded study locations.

## Why this is sequenced last, not first

The core methodology (penalty weights, what "planned" means, index
weighting) is still being actively decided across #001–#004. Generalizing
the pipeline before that settles means re-validating every change against
N cities instead of 5, for no benefit yet. The `LOCATIONS` dict + fallback
pattern already generalizes reasonably well mechanically — the real risk
here isn't plumbing, it's data quality:

- Smaller towns or less-mapped regions may have far sparser
  sidewalk/footway tagging than the 5 study cities. The friction function
  can't distinguish "genuinely no sidewalk" from "sidewalk exists but
  nobody's mapped it in OSM" — both look identical in the data, but only
  one should be scored as low walkability.

## Proposed approach (once unblocked)

- Add a per-city "tagging density" sanity metric (e.g. % of residential
  edges with *any* sidewalk-related tag present, positive or negative)
  before trusting a new city's score — surface it prominently, don't bury
  it.
- Wrap the existing pipeline functions in a single entry point; the
  `LOCATIONS` dict becomes a default set of examples rather than a
  hardcoded requirement.

## Acceptance criteria

- [ ] Not started — explicitly waiting on #001–#004 to stabilize first
