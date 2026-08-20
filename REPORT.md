# Report notes (working document)

This is a running scratchpad, not the report itself. It exists so that
methodology caveats, open questions, and interesting findings get written
down the moment they surface — during implementation, not reconstructed
from memory once it's time to write the final report. Organized into three
sections: things we still need to decide, things we already decided (and
why, for the technical reader), and things worth telling a non-technical
reader.

---

## Open questions — need a decision before the report can be finalized

**Who is the report actually for?** Still undecided. Options discussed:
a technical audience (methodology-first, findings second) vs. a
planner/policy audience (findings first, methodology in an appendix).
This also determines whether "interactive element" (#007) means a Folium
map dropped in the repo or a real dashboard — very different amounts of
work. → Revisit once Phase 2 (docs/issues 002–004) has real numbers to
show either audience.

**What does "Planned" walkability mean, precisely?** Current working
assumption (see docs/issues/002): derive it from OSM too (naive, no
penalty), rather than sourcing 5 different municipal GIS datasets across
4 countries. This is a scope/consistency tradeoff, not obviously the
"correct" choice — worth a paragraph in the report either justifying it
or noting it as a limitation if we later decide it undersells the
"planned" side of the comparison.

**Infrastructure penalty values (docs/issues/001) and index component
weights (docs/issues/004) are designed, not derived.** PLOS gives
categories, not universal numbers — most published penalty studies
calibrate locally via survey or GPS-trace data, which this project
doesn't have. The report needs to be explicit about this rather than
presenting the multipliers as if they were empirical findings. Candidate
framing: "a first-pass, documented model open to recalibration," possibly
with a sensitivity check (does the index ranking across the 5 cities
change much if penalty values move ±30%?) as a way to show the choice
isn't arbitrary even without hard calibration data.

**Personal path anchors (docs/issues/005)** — waiting on specific
start/end points per city. Good report material once available: "the
model predicts you'd have avoided X street, and you did" (or didn't) is a
concrete, human-scale validation the aggregate stats can't provide on
their own.

**Git history / data storage (docs/issues/008)** — 108MB of regenerable
data is committed directly to git. Not a report topic per se, but worth
one sentence in a "methodology & reproducibility" section either way
("data is cached and versioned in-repo" vs. "regenerate via the pipeline
script") depending on what we decide.

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
  honest limitation to state, and the seed of docs/issues/006's "tagging
  density" sanity-check idea for scaling to new cities later.

- **Penalty-ratio range across cities (friction-km ÷ raw-km)**: Lanaken
  1.16, Maastricht 1.17, Sabancı 1.22, Mindelo 1.21, Matosinhos 1.27 (run
  against the current, first-pass penalty values — expect these to move
  once #004's index and any recalibration land). Matosinhos having the
  highest ratio despite being a mid-sized coastal city is worth digging
  into once #002/#003 add route-level and choke-point detail — right now
  this is just an aggregate, not yet a "why."
