# HALO Simulator — build rules

## Protected functions
These stay byte-identical once written. Changing one requires explicit approval
naming the function. Baseline copies live in `.baseline/`.

The engine is seeded, so changing the order of random draws anywhere upstream of
the detectors changes every number in the paper. The protected unit is therefore
the **whole engine**, as two blocks copied verbatim from simulator v2:

- `worldEngine` — the complete v2 `js/world.js` (SHA-256 `f44c7873022b1af47dd8a72f34032c0ccc28dd24ee7f7cd88c3c0baaf6746373`).
  Contains the RNG (`mulberry32`, `gauss`), routing (`dijkstra`, `roadCost`, `isArterial`, `neighbors`, `route`, `detour`, `walkPath`, `makeRoute`), MoZo geometry (`polyLength`, `pointAt`, `distToSeg`, `distToPoly`, `dist`), and the map constants (`SP`, `NX`, `NY`, `ART_ROWS`, `ART_COLS`, `PLACES`).
- `simEngine` — the complete v2 `js/sim.js` (SHA-256 `d4b140ce5e34191a0a8463770187475ef350dfb49d15bfd9605fe7e0b79d5fc1`).
  Contains the detectors and scoring (`evaluate`), MoZo zones (`allowance`, `mozoAllowances`, `inAllowance`), the learned-routine baseline (`learnRoutine`), metrics (`summarize`, `median`, `pct`), day generation and the conversation-to-calendar model (`planOutings`, `buildOuting`, `legsTrack`, `splitPoly`, `planWanders`, `wanderStart`, `wanderNodes`, `extendRandom`, `planVenueWanders`, `planSpurious`, `calendarFor`, `simulateDay`, `run`), and every assumed parameter (`DEFAULTS`, `HABITS`, `IRREGULAR`, `PATTERNS`, `DAY`, `DRIVE`, `WALK`).

Each page that runs the engine (`simulated-day.html`, `experiment.html`) embeds
both blocks inline, bracketed exactly like this:

```html
<script>
// FUNCTION_START worldEngine — PROTECTED, baseline in .baseline/worldEngine.txt
...v2 world.js, unchanged...
// FUNCTION_END worldEngine
</script>
<script>
// FUNCTION_START simEngine — PROTECTED, baseline in .baseline/simEngine.txt
...v2 sim.js, unchanged...
// FUNCTION_END simEngine
</script>
```

`how-it-works.html` may embed `worldEngine` only (it needs routes, not the simulator).
Not protected: walkthrough script, map renderer, playback, experiment UI, CSS.

### Byte check (every phase that touches a page with an engine block)
Use `verify_protected` from the skill's ProtectedFunctions.md for each block in each page
(exactly one start and one end sentinel, non-empty baseline, non-empty extraction, then diff).

### Golden-numbers check (every phase that touches a page with `simEngine`)
A byte match proves nothing was edited; this proves the page still produces the paper's results.

```bash
for f in simulated-day.html experiment.html; do
  sed -n '/FUNCTION_START worldEngine\b/,/FUNCTION_END worldEngine\b/p' "$f" > /tmp/w.js
  sed -n '/FUNCTION_START simEngine\b/,/FUNCTION_END simEngine\b/p'   "$f" > /tmp/s.js
  node -e '
    require("/tmp/w.js"); require("/tmp/s.js");
    const r = HALO.sim.run({}), s = r.summary, f = x => x.toFixed(2);
    for (const k of ["static","routine","mozo","hybrid"])
      console.log(k, f(s[k].faWeek), f(s[k].faLo), f(s[k].faHi), Math.round(s[k].fastRate*100)+"%", s[k].openHoursWeek.toFixed(1));
    console.log("outings", r.totals.outings, "wanders", r.totals.wanders, "events", r.totals.events);
  ' > /tmp/golden.out
  diff /tmp/golden.out .baseline/golden.txt && echo "GOLDEN OK $f" || echo "GOLDEN FAIL $f"
done
```

Expected `.baseline/golden.txt` (matches Table I of the paper):
```
static 5.42 4.92 5.96 100% 0.0
routine 4.04 3.33 4.75 95% 3.9
mozo 3.58 2.96 4.21 96% 1.2
hybrid 2.96 2.29 3.58 95% 4.6
outings 131 wanders 57 events 84
```

## Build rules
- One self-contained HTML file per page. CSS and JS inline. No build step.
- Shared logic is duplicated, never extracted to a module. Each copy carries a
  cross-comment naming the canonical source (`.baseline/<block>.txt`).
- Illustrative data is labeled as illustrative in the UI on every page that renders it
  (how-it-works, simulated-day, experiment).
- Settings pass between pages only through the query string, parsed against an allow-list
  with numeric range checks. Never `eval` or write raw query values into HTML.
- Keep v2's design tokens (Atkinson Hyperlegible; teal = home geofence and the MoZo + routine detector, indigo = MoZo,
  ochre = learned routine, crimson = alert) and light/dark support on every page.
- Pages must work from `file://` and offline (system-font fallback).
- Never-public terms: see `NEVER_PUBLIC.local.md` (local, untracked, gitignored). These must not appear anywhere,
  including commit messages and git history.
- `.gitignore` includes `.env`, `credentials.json`, `token.json`, `*.pdf`, `paper/`.
- Design work hands off to the Webdesign skill.
- Every phase ends with a verification report and a STOP. No commit before approval.
