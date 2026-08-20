# 001 — Friction-weighted edges (slope + infrastructure penalty)

**Status:** ✅ Resolved — `friction_weighting.py`
**Labels:** algorithm, phase-2
**GitHub issue:** #1 (filed independently — "Pedestrian filter treats unsidewalked arterials as fully walkable"; this doc supersedes it with the actual resolution)

## Problem

`PEDESTRIAN_HIGHWAY_VALUES` in `pedestraian_nodes.py` whitelists `primary`,
`secondary`, `tertiary`, `trunk`, and `residential` outright. A busy
arterial with no `sidewalk`/`footway` tag was therefore treated as just as
walkable as a dedicated footpath — directly undercutting the project's
"true walkability" premise. Grade was already being computed
(`add_grade_to_edges`) but never used as a routing cost.

## Resolution

Added `friction_weighting.py`, implementing the project's own Friction
Factor formula:

```
friction_weight = length * (1 + slope_penalty + infrastructure_penalty)
```

- **slope_penalty** — derived from Tobler's hiking function (Tobler, 1993).
- **infrastructure_penalty** — a 4-tier categorical penalty (dedicated /
  low-traffic-no-sidewalk / moderate-traffic-no-sidewalk /
  high-traffic-no-sidewalk), inspired by Pedestrian Level of Service
  (PLOS) grading, plus a stacked penalty for `highway=steps`.

Output: every edge gets `slope_penalty`, `infra_tier`, `infra_penalty`,
`wheelchair_ok`, and `friction_weight`. Saved as
`locations/<name>/<name>_friction.graphml` (kept separate from the base
graph so the Phase 1 pipeline output stays untouched/reproducible).

## Notable findings along the way

1. **Real data bug**: the original filter's `bool(data.get("sidewalk"))`
   check treats `sidewalk="no"` as truthy. Mindelo alone has 30 edges
   explicitly tagged `sidewalk=no` that were silently mis-classified as
   "has sidewalk". Fixed via `_tag_is_positive()`.
2. **Tobler's function doesn't peak at flat ground** — it peaks at a
   gentle ~5% downhill, where people walk fastest. A naive ratio would
   make friction *dip below* raw distance near that point, contradicting
   the project's own `1 + penalties` formula (which should never go below
   1). Resolved by flooring the slope penalty at 0 — see
   `friction_weighting.py` docstring for the full reasoning. Documented
   as a modeling decision in REPORT.md, not silently patched over.

## Validation

Ran against all 5 cached graphs (0 missing grades in every city). Asserted
`friction_weight >= length` on every edge, every city — holds. Penalty
ratio (total friction-km / total raw-km) ranges 1.16–1.27 across the five
locations; Sabancı University (a car-free campus) has zero
moderate/high-traffic-no-sidewalk edges, as expected.

## Not yet done (tracked separately)

- The specific infrastructure penalty *values* are a documented judgment
  call, not literature-derived constants — see REPORT.md.
- Nothing yet *routes* using `friction_weight` — that's
  [002](002-planned-vs-actual-baseline-and-comparison.md).
