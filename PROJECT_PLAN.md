# HALO Simulator

An interactive simulation of HALO's schedule-aware Mobile Zone (MoZo), comparing it against static and learned-routine geofences on synthetic days of daily life.

- **Repo slug:** `halo-simulator`
- **Live URL (after Deploy):** `https://TSanf0rd.github.io/halo-simulator/`
- **Source:** HALO simulator v2 (tabbed single page), split into one self-contained page per section.

## Pages
| Phase | File | Purpose | Illustrative-data label |
|-------|------|---------|-------------------------|
| 2 | `index.html` | Landing: what HALO is, one card linking to each page | No |
| 3 | `how-it-works.html` | Guided walkthrough: conversation → Synthetic Memory → calendar → MoZo, with "ask, don't assume" vs. "fill in defaults" toggle | Yes |
| 4 | `simulated-day.html` | One synthetic day played minute by minute; four detectors on the same GPS track; timeline of real alerts vs. false alarms | Yes |
| 5 | `experiment.html` | Batch runs, charts with 95% intervals, ask-vs-assume and approval-gate comparisons, corridor-width sweep | Yes |
| 6 | `about.html` | Assumptions, real vs. simulated, metric definitions, credits | No (describes the data) |

### Cross-page settings
Pages share no memory, so experiment settings travel in the query string, e.g.
`simulated-day.html?seed=2026&days=168&assistant=assume&gate=1&W=150&buf=20`.
The engine is seeded, so the same query always regenerates the same days.
`experiment.html` links to "Watch a day from this run" with the current settings;
`simulated-day.html` with no query uses the defaults (the paper's configuration).
Unknown or out-of-range parameters fall back to defaults; never `eval` or inject query values into HTML.

## Data Provenance
| Source | Real or Illustrative | Notes |
|--------|---------------------|-------|
| Neighborhood grid and the 9 place names | Illustrative | Fictional 10×7 grid, 400 m blocks |
| Margaret, Sarah, Martha | Illustrative | Fictional characters |
| Walkthrough conversation and pipeline JSON | Illustrative | Scripted to match the real pipeline's stages |
| Outings, wandering, GPS tracks, calendar events | Illustrative | Synthetic, seeded (default seed 2026) |
| Simulation parameters (`DEFAULTS`) | Illustrative | Assumptions, all documented on `about.html` |
| 7% extraction error rate | Real-derived | From pipeline validation F1 ≈ 0.93 (n = 20) |
| Four wandering patterns | Real taxonomy, synthetic instances | Martino-Saltzman et al., 1991 |
| Participant data, real GPS, real transcripts | None | Nothing real about any person appears on the site |

## Never-public terms
The list lives in a local, untracked file (`NEVER_PUBLIC.local.md`, gitignored); Preflight reads its terms from there.

## Protected Functions
(see CLAUDE.md — the list lives there)

## Phase Log
| Phase | Status | Approved | Commit |
|-------|--------|----------|--------|
| 0 — Intake | Done | 2026-10-19 | — |
| 1 — Repo setup | Done | 2026-10-19 | this commit |
| 2 — index.html | Done | 2026-10-20 | this commit |
| 3 — how-it-works.html | Done | 2026-10-20 | this commit |
| 4 — simulated-day.html | Not started | — | — |
| 5 — experiment.html | Not started | — | — |
| 6 — about.html | Not started | — | — |
| Preflight | Not started | — | — |
| Deploy | Not started | — | — |
