# 008 — Repo hygiene & portfolio polish

**Status:** 🟡 Partially done
**Labels:** chore, portfolio
**GitHub issue:** #8

## Done

- [x] `README.md` (dual-audience: planner/policy summary + technical
      deep-dive)
- [x] `requirements.txt`
- [x] `.gitignore` (Python/OS cruft; does **not** yet address the large
      generated-data question below)
- [x] `ROADMAP.md`, `REPORT.md`, and this issue backlog under
      `docs/issues/`

## Not done — needs an explicit decision, not a silent fix

- [ ] **Git history bloat**: `locations/` + `figures/` add up to ~108MB of
      generated, regenerable data committed directly to git. Options: (a)
      leave as-is — simplest, but a heavy clone for a portfolio repo; (b)
      migrate to Git LFS; (c) `.gitignore` the data and document
      "run `pedestraian_nodes.py` to regenerate" instead. (b) and (c) both
      require rewriting git history to actually shrink the repo (`git
      filter-repo` + force-push) — a destructive operation, not doing this
      without explicit sign-off. Recorded, not actioned.
- [ ] **Filename typo**: `pedestraian_nodes.py` → `pedestrian_nodes.py`.
      Trivial fix, but touches an import in `visualise_node_maps.py` and
      (depending on the git-history decision above) might as well happen
      in the same pass as any history rewrite rather than as a separate
      commit.
- [ ] **`METHODOLOGY.md`**: once #001–#004 stabilize, extract the
      "documented judgment call, not a citation" caveats scattered through
      the code docstrings into one place a reader can find without digging
      through source — feeds directly into the final report.

## Acceptance criteria

- [ ] Git history / data-storage decision made and recorded (even if the
      decision is "leave it")
- [ ] Filename typo fixed (if bundled with history rewrite, otherwise
      standalone)
- [ ] `METHODOLOGY.md` exists once the algorithm issues close
