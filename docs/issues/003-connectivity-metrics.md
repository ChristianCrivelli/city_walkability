# 003 — Connectivity metrics: circuity & betweenness centrality

**Status:** 🔲 Open
**Labels:** algorithm, phase-2
**Depends on:** [001](001-infrastructure-friction-weighting.md)
**GitHub issue:** #3

## Problem

The project plan calls for circuity (network distance ÷ straight-line
distance) and betweenness centrality to surface dead-ends and choke
points. Neither is implemented.

## Proposed approach

- **Circuity**: for a sample of node pairs (reuse the O-D sampling from
  #002), compute `friction_path_length / great_circle_distance`. Aggregate
  per city (mean, and a distribution — circuity is not uniform across a
  city, and the interesting finding is likely *where* it's high, not just
  the average).
- **Betweenness centrality**: `nx.betweenness_centrality(G, weight=...)`
  on the friction-weighted graph — expensive on the larger graphs
  (Matosinhos: ~15k nodes), so budget for either a k-sample approximation
  (`k=` parameter) or an overnight batch run; note the runtime tradeoff in
  the code comments.
- Surface the top-N highest-betweenness nodes per city as candidate
  "choke points" — these are the most policy-actionable output of this
  issue (a specific intersection a planner could look at), not just an
  aggregate number.

## Acceptance criteria

- [ ] Circuity computed per city, with both an aggregate value and a
      per-region view (even a coarse grid-cell bucket is fine for v1)
- [ ] Betweenness centrality computed per city on the friction-weighted
      graph
- [ ] Top-10 choke-point nodes exported per city (node id, lat/lon,
      betweenness score) for later use in visualization
