# 007 — Interactive visualization

**Status:** 🔲 Open
**Labels:** visualization, portfolio
**Depends on:** [001](001-infrastructure-friction-weighting.md), nice to have [003](003-connectivity-metrics.md)
**GitHub issue:** #7

## Problem

Current visualization (`visualise_node_maps.py`) is static PNG, colored by
node degree only. Once friction/circuity/betweenness data exists, a
static map under-sells it — this is also the most visible piece for a
non-technical (planner/policy) audience.

## Proposed approach (scope TBD — see REPORT.md)

Minimum viable: a Folium HTML map per city, edges colored by
`infra_tier`/`friction_weight`, with the top betweenness "choke points"
(#003) marked as callouts. Self-contained HTML, no server needed — cheap
to host in the repo or embed in the report.

Stretch: a small interactive dashboard (Streamlit/Observable) letting a
viewer toggle Planned vs. Actual, swap the color attribute, or select a
city — meaningfully more effort, only worth it if the "who's this for"
question (REPORT.md) lands on wanting a live demo rather than a fixed
report figure.

## Acceptance criteria

- [ ] At least one interactive (non-static) map artifact per city
- [ ] Decision recorded in REPORT.md on Folium-only vs. full dashboard
      before starting the stretch scope
