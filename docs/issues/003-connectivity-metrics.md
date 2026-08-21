# 003 — Connectivity metrics: circuity & betweenness centrality

**Status:** ✅ Resolved — `connectivity_metrics.py`
**Labels:** algorithm, phase-2
**Depends on:** [001](001-infrastructure-friction-weighting.md)
**GitHub issue:** #3

## Problem

The project plan calls for circuity (network distance ÷ straight-line
distance) and betweenness centrality to surface dead-ends and choke
points. Neither is implemented.

## Resolution

Added `connectivity_metrics.py`:

- **Circuity** (`compute_circuity`): reuses #002's exact O-D sampling
  (`random_od_pairs`, same seed) and its already-computed `actual_length_m`
  (physical length of the friction-weighted shortest path) rather than
  re-running shortest paths — circuity is `actual_length_m /
  great_circle_m`, a geometric detour ratio, not an effort ratio, so it's
  deliberately computed against physical length rather than friction cost.
  Great-circle distance via haversine on node lat/lon.
- **Per-region view** (`circuity_by_grid_cell`): buckets each pair's
  origin into a projected 500m×500m grid cell and reports mean circuity +
  pair count per cell — a coarse v1 substitute for a real spatial density
  surface, per the issue's own acceptance criteria.
- **Betweenness centrality** (`compute_betweenness`): computed on a
  simplified `DiGraph` (parallel edges collapsed to the cheapest by
  `friction_weight` — the same resolution rule `nx.shortest_path` applies
  implicitly). Kept **directed**, not collapsed to undirected: friction is
  slope-dependent, so a choke point can genuinely differ by direction of
  travel.
- **Top-10 choke points** (`top_choke_points`): highest-betweenness nodes
  per city, with lat/lon and score, tagged with which betweenness method
  produced them.

### Betweenness scope decision (recorded, not silently applied)

Exact betweenness is O(V·E·log V) with networkx's pure-Python
Dijkstra-based implementation — fine for the three smaller graphs,
impractical for the two largest at this graph size. Decision:

| City | Nodes | Method |
|---|---|---|
| Maastricht | 12,506 | k-sample approximation (k=500, seed=42) |
| Matosinhos | 15,230 | k-sample approximation (k=500, seed=42) |
| Sabancı University | 364 | Exact |
| Lanaken | 3,552 | Exact |
| Mindelo | 504 | Exact |

Total runtime for all 5 cities (circuity + betweenness): ~3 minutes.
`betweenness_method` is recorded per row in each city's
`*_betweenness_top10.csv` rather than left implicit.

## Results — all 5 study locations, 200 O-D pairs each (same sample as #002)

| City | mean circuity | median circuity | max circuity | top choke point (betweenness) |
|---|---|---|---|---|
| Maastricht | 1.448 | 1.352 | 3.28 | node 618628852 (0.281) |
| Matosinhos | 1.358 | 1.320 | 3.16 | node 10024023734 (0.174) |
| Sabancı University | 1.516 | 1.385 | 11.80 | node 2265527528 (0.260) |
| Lanaken | 1.362 | 1.303 | 4.52 | node 1477629854 (0.137) |
| Mindelo | 1.338 | 1.307 | 3.54 | node 1428182504 (0.409) |

Full per-pair circuity, per-grid-cell aggregates, and top-10 choke points:
`locations/<city>/<city>_circuity.csv`,
`locations/<city>/<city>_circuity_by_grid_cell.csv`,
`locations/<city>/<city>_betweenness_top10.csv`. Aggregate table:
`locations/all_locations_connectivity_summary.csv`.

**Caveat worth flagging (not a bug, checked)**: Sabancı's max circuity of
11.8 comes from a real O-D pair only 12m apart in a straight line but
143m apart on the network (two points close across a building/obstacle
with no direct path) — a known property of circuity ratios at very short
distances: a small absolute detour produces a huge ratio. Median circuity
is the more robust central-tendency figure for this reason; mean is
reported alongside it, not instead of it.

## Not yet done (tracked separately)

- The composite index (#004) is the next consumer of these numbers
  (circuity + dead-end density feed into it directly).
- Grid-cell bucketing is coarse (500m, fixed size, no minimum-sample
  filtering) — fine for a v1 "where is it high" view, not yet a proper
  spatial surface. Revisit if #007 (interactive visualization) wants to
  render this as a heatmap.
