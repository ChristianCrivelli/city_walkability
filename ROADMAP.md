# Roadmap

This project has two audiences by design (see README): a planner/
policymaker who wants the findings, and a technical reviewer who wants to
see the engineering hold up. The roadmap below is sequenced so that the
*methodology* (how "walkability" gets scored) stabilizes before the
project scales out to more cities or more polished visuals — validating
five cities properly beats generalizing to fifty prematurely.

Each phase links to its tracked issue(s) in `docs/issues/`.

## Phase 1 — Data pipeline ✅ Done

Pedestrian network extraction (OSMnx), infrastructure-tag filtering,
elevation enrichment, grade calculation, GraphML/GeoPackage/CSV export,
and a first static visualization — all five study locations (Maastricht,
Matosinhos, Sabancı University, Lanaken, Mindelo) processed and cached.

_Code: `pedestraian_nodes.py`, `visualise_node_maps.py`_

## Phase 2 — The True Walkability Algorithm (in progress)

The project's actual differentiator. Order matters here — each step
depends on the graph the previous step produced:

1. ✅ [**001 — Friction-weighted edges**](docs/issues/001-infrastructure-friction-weighting.md) (GitHub #1)
   (slope penalty + infrastructure penalty combined into `friction_weight`)
2. ✅ [**002 — Planned vs. Actual comparison**](docs/issues/002-planned-vs-actual-baseline-and-comparison.md) (GitHub #2)
   (`planned_vs_actual.py` — random O-D sampling + per-pair comparison, run against all 5 cities; POI-based sampling deferred to #005)
3. 🔲 [**003 — Connectivity metrics**](docs/issues/003-connectivity-metrics.md) (GitHub #3)
   (circuity, betweenness centrality / choke points)
4. 🔲 [**004 — Composite True Walkability Index**](docs/issues/004-true-walkability-index.md) (GitHub #4)

Running alongside Phase 2, not blocking it:

- 🔲 [**005 — Personal-path validation**](docs/issues/005-personal-path-validation.md) (GitHub #5)
  — anchor the comparison in real, lived routes as soon as start/end
  points are supplied.

## Phase 3 — Scale & polish (after Phase 2 stabilizes)

- 🔲 [**006 — Generalize to an arbitrary city**](docs/issues/006-generalize-to-arbitrary-city.md) (GitHub #6)
  — deliberately sequenced last; see that issue for why.
- 🔲 [**007 — Interactive visualization**](docs/issues/007-interactive-visualization.md) (GitHub #7)
- 🟡 [**008 — Repo hygiene & portfolio polish**](docs/issues/008-repo-hygiene-and-portfolio-polish.md) (GitHub #8)
  — partially done alongside this roadmap; git-history/data-storage
  decision still open.

## Phase 4 — The report

Not an issue — tracked as running notes in [`REPORT.md`](REPORT.md) as we
go, then written up once Phase 2's numbers exist to report on. Audience
and format (written report vs. interactive companion, see #007) still
open — see REPORT.md.

## Explicitly not yet decided

A few open methodology questions block clean progress on Phase 2 and are
tracked in detail in REPORT.md rather than buried in issue text:
what "Planned" means precisely, whether/how municipal data factors in,
and how the infrastructure-penalty and index weights get chosen and
justified.
