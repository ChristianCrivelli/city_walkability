# 002 — "Planned" baseline graph + Planned-vs-Actual comparison

**Status:** ✅ Resolved — `planned_vs_actual.py`
**Labels:** algorithm, phase-2, core-differentiator
**Depends on:** [001](001-infrastructure-friction-weighting.md)
**GitHub issue:** #2

## Problem

This is the project's stated "unique angle" and doesn't exist yet: comparing
naive/"planned" walkability (what a standard tool like WalkScore assumes)
against "actual" walkability (the friction-weighted graph from #001).

## Resolution

Added `planned_vs_actual.py`:

- **Planned weights** (`add_planned_weights`): `planned_weight = length` on
  every edge — every pedestrian-eligible edge assumed equally walkable, no
  slope or infrastructure penalty. Derived from the same OSM pull as the
  Actual graph rather than 5 separate municipal GIS datasets, so the
  comparison stays consistent across all 5 study locations. **This is a
  scope/consistency tradeoff, not obviously the "correct" definition of
  "Planned"** — still an open framing question for the report, not
  resolved by this issue (see REPORT.md).
- **O-D sampling** (`random_od_pairs`): random pairs drawn from the graph's
  largest strongly-connected component (guarantees every pair is reachable
  both ways, so no sampled pair is wasted). `poi_based_od_pairs` is present
  as an explicit stub, not implemented — see "Not yet done" below.
- **Comparison** (`compare_route` / `compare_od_pairs`): for each O-D pair,
  runs `nx.shortest_path` once with `weight="planned_weight"` and once with
  `weight="friction_weight"`, and reports per pair: physical length delta
  (metres and %), friction-cost savings (%), route overlap (% of the
  shorter route's edges also used by the other route), and whether the
  routes share zero edges (`diverges_completely`).

## Results — all 5 study locations, 200 random O-D pairs each (seed=42)

| City | mean length delta | mean friction savings | mean route overlap | fully divergent |
|---|---|---|---|---|
| Maastricht | +2.9% | 6.0% | 57.7% | 1/200 |
| Matosinhos | +4.2% | 6.2% | 50.8% | 0/200 |
| Sabancı University | +2.3% | 2.4% | 81.5% | 2/200 |
| Lanaken | +3.2% | 6.0% | 65.0% | 6/200 |
| Mindelo | +0.6% | 1.1% | 89.8% | 1/200 |

Full per-pair output: `locations/<city>/<city>_planned_vs_actual.csv`.
Aggregate table: `locations/all_locations_planned_vs_actual_summary.csv`.

Reading these together: the Actual (friction-aware) route is consistently a
little *physically longer* than the naive Planned route (0.6–4.2% on
average) but consistently *cheaper in effort* (1.1–6.2% friction savings) —
i.e. the model is doing exactly what it's supposed to: trading a bit of
extra distance for meaningfully less exposure to slope and poor
infrastructure. Lanaken has the highest full-divergence rate (3%) and
Sabancı the highest overlap (81.5%, unsurprising for a small, mostly
dedicated-path campus with few detour options).

## Hand-checked validation route

Lanaken, node `2204882099` → `13192415045` (one of the 6 fully-divergent
pairs, chosen for illustration):

- **Planned route**: 8,804 m, 69% of it (6,073 m across 80 of 127 edges) on
  `high_traffic_no_sidewalk` roads — the naive shortest path runs straight
  down busy arterials because it has no way to know they're unsafe.
- **Actual route**: 9,967 m (+13.2% distance) but only 500 m of
  `high_traffic_no_sidewalk` exposure — an 11.8x reduction — and 23.6%
  lower friction cost overall.

This is the concrete, human-legible version of the aggregate numbers above:
a pedestrian following the Actual route walks noticeably further to almost
entirely avoid a stretch of dangerous, sidewalk-free arterial road. Good
candidate for a report figure/anecdote once #007 (interactive
visualization) exists to show the two routes on a map.

## Not yet done (tracked separately)

- **POI-based O-D pairs** (`poi_based_od_pairs`) — stubbed, not
  implemented. Needs a live Overpass API call this pipeline otherwise
  doesn't depend on, which wasn't confirmed reachable from the analysis
  environment. Deferred in favor of [005](005-personal-path-validation.md)
  (real, human-picked start/end points) as the stronger source of
  human-meaningful O-D pairs — revisit only if the report specifically
  needs synthetic nearest-amenity pairs in addition to #005's real ones.
- **"Planned" definition** is still an open framing question (OSM-derived
  naive baseline vs. sourcing real municipal planning data) — see REPORT.md.
- Route overlap is computed as % of shared *edges* (edge count), not
  length-weighted overlap. Reasonable for v1; the Lanaken example above
  shows it can diverge in edge-count terms even when the two routes share
  a long common stretch elsewhere, worth a length-weighted refinement if a
  future example suggests edge-count overlap is misleading.
