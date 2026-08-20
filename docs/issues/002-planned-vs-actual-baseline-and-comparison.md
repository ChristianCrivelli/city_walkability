# 002 — "Planned" baseline graph + Planned-vs-Actual comparison

**Status:** 🔲 Open
**Labels:** algorithm, phase-2, core-differentiator
**Depends on:** [001](001-infrastructure-friction-weighting.md)
**GitHub issue:** #2

## Problem

This is the project's stated "unique angle" and doesn't exist yet: comparing
naive/"planned" walkability (what a standard tool like WalkScore assumes)
against "actual" walkability (the friction-weighted graph from #001).

## Proposed approach

- **Planned graph**: the same OSMnx pull, *without* the infrastructure
  penalty — i.e. every non-motorway edge assumed equally walkable
  (`friction_weight = length`, no slope or infra penalty). This keeps the
  comparison derivable entirely from OSM data across all 5 (very
  different) municipalities, rather than sourcing 5 separate municipal GIS
  datasets. **Open decision** — see REPORT.md.
- **Actual graph**: the `_friction.graphml` output from #001.
- **Comparison**: for a shared set of origin-destination (O-D) pairs, run
  `nx.shortest_path` once with `weight="length"` (Planned) and once with
  `weight="friction_weight"` (Actual). Report, per pair: path length
  delta, route overlap %, and whether the routes diverge onto different
  streets entirely.
- O-D pairs: start with randomly sampled node pairs for statistical
  coverage, then layer in POI-based pairs (residential → nearest amenity)
  and personal-experience pairs — see
  [005](005-personal-path-validation.md).

## Acceptance criteria

- [ ] `planned_graph.py` (or equivalent) producing a planned-weighted graph
      per city
- [ ] O-D sampling function (random + POI-based)
- [ ] Comparison function returning a per-pair-and-per-city DataFrame
- [ ] Runs cleanly against all 5 cached cities, output validated for at
      least one hand-checked route
