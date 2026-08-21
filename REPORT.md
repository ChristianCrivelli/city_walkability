# Report notes (working document)

This is a running scratchpad, not the report itself. It exists so that
methodology caveats, open questions, and interesting findings get written
down the moment they surface — during implementation, not reconstructed
from memory once it's time to write the final report. Organized into three
sections: things we still need to decide, things we already decided (and
why, for the technical reader), and things worth telling a non-technical
reader.

Issue status and resolution notes live on
[GitHub Issues](https://github.com/ChristianCrivelli/city_walkability/issues)
— this file is where the *methodology* narrative accumulates across
issues, not a duplicate of issue tracking.

---

## Open questions — need a decision before the report can be finalized

**Who is the report actually for?** Still undecided. Options discussed:
a technical audience (methodology-first, findings second) vs. a
planner/policy audience (findings first, methodology in an appendix).
This also determines whether "interactive element" (#7) means a Folium
map dropped in the repo or a real dashboard — very different amounts of
work. #2 and #3 now have real numbers (see Findings below) — worth
revisiting this question sooner rather than after #4 too.

**What does "Planned" walkability mean, precisely?** Implemented for #2
as the working assumption: derive it from OSM too (naive, no penalty),
rather than sourcing 5 different municipal GIS datasets across 4
countries. This is a scope/consistency tradeoff, not obviously the
"correct" choice — worth a paragraph in the report either justifying it
or noting it as a limitation if we later decide it undersells the
"planned" side of the comparison. Implementing it didn't resolve the
framing question, just made the tradeoff concrete.

**Infrastructure penalty values (#1) and index component weights (#4)
are designed, not derived.** PLOS gives categories, not universal
numbers — most published penalty studies calibrate locally via survey or
GPS-trace data, which this project doesn't have. The report needs to be
explicit about this rather than presenting the multipliers as if they
were empirical findings. Candidate framing: "a first-pass, documented
model open to recalibration," possibly with a sensitivity check (does
the index ranking across the 5 cities change much if penalty values move
±30%?) as a way to show the choice isn't arbitrary even without hard
calibration data.

**Personal path anchors (#5)** — waiting on specific start/end points
per city. Good report material once available: "the model predicts
you'd have avoided X street, and you did" (or didn't) is a concrete,
human-scale validation the aggregate stats can't provide on their own.
Also the natural home for human-meaningful O-D pairs that #2 deliberately
punted on (see below) — no need for synthetic nearest-amenity sampling if
real routes are coming anyway.

**Git history / data storage (#8)** — decided: leaving the ~108MB as-is
for now (simplest option). Migrating to LFS or gitignore+regenerate would
need a git filter-repo history rewrite plus a force-push — done locally
if/when revisited, not from an automated pass. Still worth one sentence
in the report's "methodology & reproducibility" section either way.

---

## Decisions made, and why (technical-reader material)

- **Friction formula is additive, not multiplicative**:
  `friction_weight = length * (1 + slope_penalty + infra_penalty)`,
  matching the project's own original plan (`W = d × (1 + slope_penalty +
  infrastructure_penalty)`) rather than a multiplicative combination
  (`length * slope_mult * infra_mult`) — the two aren't equivalent
  (multiplicative has a cross-term), and additive is what was actually
  designed.

- **Tobler's hiking function doesn't peak at flat ground** — it peaks at
  a gentle ~5% downhill. A literal speed-ratio penalty would therefore be
  *negative* (a "discount") right around there, which contradicts a
  strictly-additive `1 + penalties` formula. Resolved by flooring the
  slope penalty at 0 (gentle downhill costs nothing extra, but is never
  cheaper than flat). This is a deliberate simplification of Tobler's
  function, not a property of the function itself — worth a sentence in
  the report's methodology section since it's a subtle point a technical
  reviewer would likely ask about.

- **Elevation data source**: Open-Elevation (free tier) returned 0
  missing values across all 5 cities (3,552–15,230 nodes each) — no need
  to fall back to a local SRTM DEM. Worth noting as a "we checked, it held
  up" line rather than silently assuming it.

- **#2 O-D sampling scope: random pairs only, POI-based sampling
  deferred.** POI-based pairs (residential → nearest amenity) need a live
  Overpass API call the pipeline doesn't otherwise depend on, and #5
  (personal-path validation, real human-picked routes) is a stronger
  source of human-meaningful O-D pairs than synthetic nearest-amenity
  sampling would be anyway. Random sampling is drawn from each graph's
  largest strongly-connected component so every sampled pair is
  guaranteed reachable — no wasted samples. `poi_based_od_pairs` exists
  as an explicit stub rather than being silently dropped, in case a report
  narrative later wants synthetic POI pairs in addition to #5's real
  ones.

- **Route overlap metric (#2) is edge-count based**, not
  length-weighted: % of the shorter route's edges also used by the other
  route. Simple and symmetric for v1; flagged as a candidate refinement
  if a future example shows edge-count overlap reads misleadingly against
  a length-weighted version.

- **#3 betweenness centrality: exact for 3 cities, k-sample approximation
  for 2.** Exact is O(V·E·log V) with networkx's pure-Python Dijkstra —
  fine for Sabancı (364 nodes), Mindelo (504), Lanaken (3,552); impractical
  for Maastricht (12,506) and Matosinhos (15,230) at reasonable runtime.
  Used k=500 (fixed seed=42) for those two. Recorded per-row in each
  city's output (`betweenness_method` column), not silently applied as one
  blanket method. Total runtime for all 5 cities: ~3 minutes.

- **#3 circuity uses physical route length, not friction cost.** Circuity
  is a geometric detour ratio (network distance ÷ straight-line distance),
  so it's computed against `actual_length_m` (physical metres) from #2,
  not `friction_weight` (effort units) — the two answer different
  questions and shouldn't be conflated.

---

## Findings worth telling a non-technical reader

- **A real bug, found and fixed**: the original filter treated
  `sidewalk=no` (an explicit "there is no sidewalk here" OSM tag) as if it
  meant a sidewalk was present, because the code only checked whether the
  tag existed, not what it said. Mindelo alone had 30 street segments
  affected. Good, concrete "we found and fixed a real data-quality issue"
  anecdote for a portfolio-facing readme/report.

- **Sabancı University has essentially no sidewalk tagging at all** — 0
  edges with an explicit sidewalk tag, out of 958. Two possible readings:
  the campus genuinely has no separated sidewalks (plausible — it's a
  small, mostly car-light campus), or OSM simply hasn't been mapped in
  that detail there. Can't distinguish the two from this data alone — an
  honest limitation to state, and the seed of #6's "tagging density"
  sanity-check idea for scaling to new cities later.

- **Penalty-ratio range across cities (friction-km ÷ raw-km)**: Lanaken
  1.16, Maastricht 1.17, Sabancı 1.22, Mindelo 1.21, Matosinhos 1.27 (run
  against the current, first-pass penalty values — expect these to move
  once #4's index and any recalibration land). Matosinhos having the
  highest ratio is now something #3's choke-point/circuity detail can
  help explain in the report — see below.

- **#2 Planned-vs-Actual results (200 random O-D pairs/city, seed=42)**:
  the friction-aware ("Actual") route is consistently a little physically
  *longer* than the naive ("Planned") route, but consistently *cheaper in
  effort* — exactly the behavior the model is supposed to produce.

  | City | length delta | friction savings | route overlap | fully divergent |
  |---|---|---|---|---|
  | Maastricht | +2.9% | 6.0% | 57.7% | 1/200 |
  | Matosinhos | +4.2% | 6.2% | 50.8% | 0/200 |
  | Sabancı University | +2.3% | 2.4% | 81.5% | 2/200 |
  | Lanaken | +3.2% | 6.0% | 65.0% | 6/200 |
  | Mindelo | +0.6% | 1.1% | 89.8% | 1/200 |

  Sabancı has the highest overlap (81.5%) — makes sense for a small,
  mostly-dedicated-path campus with few detour options. Lanaken has the
  highest full-divergence rate (3%).

- **Hand-checked validation route (Lanaken, node 2204882099 →
  13192415045)** — a strong, concrete anecdote for the report: the
  Planned route runs 8,804m, with 69% of that (6,073m across 80 of 127
  edges) on `high_traffic_no_sidewalk` roads, because a naive
  shortest-path has no way to know those roads are unsafe for a
  pedestrian. The Actual route is 9,967m (+13.2% distance) but cuts
  `high_traffic_no_sidewalk` exposure to just 500m — an 11.8x reduction —
  for 23.6% lower friction cost overall. In plain terms: the model routes
  you noticeably further to almost entirely avoid a stretch of dangerous,
  sidewalk-free arterial road. Worth a map figure once #7 (interactive
  visualization) exists.

- **#3 connectivity results (same 200-pair sample as #2)**: mean circuity
  across the 5 cities ranges 1.34–1.52 (a typical pedestrian route is
  34–52% longer than straight-line distance) — Sabancı highest (1.52),
  Mindelo lowest (1.34).

  | City | mean circuity | median circuity | top choke point (betweenness) |
  |---|---|---|---|
  | Maastricht | 1.448 | 1.352 | node 618628852 (0.281) |
  | Matosinhos | 1.358 | 1.320 | node 10024023734 (0.174) |
  | Sabancı University | 1.516 | 1.385 | node 2265527528 (0.260) |
  | Lanaken | 1.362 | 1.303 | node 1477629854 (0.137) |
  | Mindelo | 1.338 | 1.307 | node 1428182504 (0.409) |

  Mindelo's single top choke point carries unusually high betweenness
  (0.409, notably above the other cities' top scores) — plausibly a
  single bridge/bottleneck street that most cross-town routes are forced
  through, worth checking against a map once #7 exists. Good candidate
  policy anecdote: "here is the one intersection that would benefit most
  from a targeted fix."

  **Caveat, checked not a bug**: Sabancı's max per-pair circuity hit 11.8
  — traced to a real O-D pair only 12m apart in a straight line but 143m
  apart on the network (two points close across a building/obstacle with
  no direct path). Known property of circuity ratios at short distances:
  a small absolute detour produces a huge ratio. Use median, not mean, as
  the headline circuity figure per city for this reason.
