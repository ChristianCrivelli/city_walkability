# 004 — Composite True Walkability Index

**Status:** 🔲 Open
**Labels:** algorithm, phase-2, headline-deliverable
**Depends on:** [002](002-planned-vs-actual-baseline-and-comparison.md), [003](003-connectivity-metrics.md)
**GitHub issue:** #4

## Problem

Right now every metric (friction ratio, circuity, betweenness, Planned-vs-
Actual gap) is a separate number. Nothing rolls them into the single,
comparable-across-cities score the project is named after — this is what
makes the five study locations legible to a non-technical audience at a
glance ("Matosinhos scores X, Maastricht scores Y").

## Proposed approach

- Define the index as a weighted combination of (at minimum): average
  friction penalty ratio, average circuity, dead-end density (fraction of
  degree-1 nodes), and the Planned-vs-Actual gap from #002.
- Normalize each component before combining (e.g. min-max or z-score
  across the 5 cities) so no single metric with a wide numeric range
  dominates the composite by accident.
- **Component weights are a judgment call, same as the infra-penalty
  values in #001** — document them explicitly rather than presenting the
  index as if it were derived, not designed. Flag in REPORT.md.
- Sanity-check the index against intuition: does the tiny, sidewalk-free
  campus (Sabancı) score believably relative to a dense European city
  center (Maastricht)? A score that contradicts obvious ground truth is a
  sign the weighting needs revisiting before publishing it.

## Acceptance criteria

- [ ] `true_walkability_index(city) -> float` (plus the component
      breakdown, not just the final number — needed for the report's
      "why did this city score this way" narrative)
- [ ] Computed for all 5 study locations
- [ ] Documented weighting rationale in the code and in REPORT.md
