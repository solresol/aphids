# IMPROVEMENTS.md

_Analysis date: 2026-07-11_

## What this project is

`aphids` is a small collection of static, single-file HTML "explosive growth"
visualisers, each deployed to its own subdomain of `arithmetic.guru`:

- `aphids.arithmetic.guru/index.html` — the original aphid population explosion
  demo (README: "I got nerd sniped into making a visualisation of aphid growth").
- `exponentials.arithmetic.guru/index.html` — generic exponential-growth calculator.
- `k-bonacci.arithmetic.guru/index.html` — a k-bonacci sequence calculator.

Each directory has a trivial `deploy.sh` that `scp`s `index.html` to
`merah.cassia.ifost.org.au`. There is no build step, no tests, no CI, and the
three HTML files are near-duplicates that have drifted apart. Working tree is
clean; last commit `ffd1250` added the k-bonacci calculator.

## Bugs & Fixes

- **`getEmoji()` breaks on fractional populations** (aphids `index.html` ~line 292).
  `calculatePopulation()` returns `Math.pow(offspring, day/genTime)`, a float.
  For `count < 10` the code does `'🐛'.repeat(Math.floor(count))` — fine — but the
  `count === 1` branch only fires on an exact integer `1`, and day 0 gives exactly
  `1`, whereas most other days give non-integer counts that never hit the tidy
  buckets cleanly. Round `count` once at the top of `getEmoji` and branch on the
  rounded value.
- **`repeat(0)` on day-zero-ish small values.** If `Math.floor(count)` is `0`
  (any count in `[0,1)`), `'🐛'.repeat(0)` renders an empty emoji display. Guard
  for `< 1`.
- **`calculatePopulation` comment vs. behaviour mismatch.** The comment says
  "population = initial * (offspring)^(day/genTime)" but `initial` is never a
  factor — it is hardcoded to 1. `totalPopulation` is initialised to 1 and, from
  what is visible, never used. Remove the dead variable or wire up an
  initial-population input.
- **Fixed 30-day horizon is hardcoded** (`for (let day = 0; day <= 30; day++)`).
  The UI lets users change offspring/genTime but not the time window; large
  genTimes make the chart nearly flat. Expose the horizon as a control.
- **No input validation.** `parseInt` on empty/garbage input yields `NaN`, which
  silently produces `NaN` populations throughout. Validate and clamp offspring
  and genTime (genTime must be `>= 1` to avoid div-by-zero-ish `day/0`).

## Improvements

- **De-duplicate the three pages.** They share the same layout, CSS, and number
  formatting. Extract a shared `styles.css` and `growth.js` (or a small template)
  so a fix to `formatNumber`/`getEmoji` doesn't have to be made three times.
- **Accessibility & responsiveness:** the fixed `max-width: 1200px` container and
  emoji-only "chart" are not screen-reader friendly. Add `aria-label`s and a
  tabular fallback of the numbers.
- **Consider a real chart** (log-scale line) alongside the emoji view — exponential
  data is unreadable on a linear emoji strip past a few generations.

## Testing

- There are **no tests at all**. Even for static pages, extract the pure functions
  (`calculatePopulation`, `formatNumber`, `getEmoji`, the k-bonacci recurrence) into
  a `.js` module and add a handful of unit tests (e.g. `vitest` or plain `node
  --test`). The k-bonacci recurrence in particular is easy to get off-by-one.

## Documentation

- README is a single sentence and only exists in `aphids.arithmetic.guru/`. Add a
  top-level `README.md` describing all three sites, the shared deploy pattern, and
  the target host. Document what `offspring`/`genTime` mean and the modelling
  assumptions (the "Fun Fact" box already hints these are unrealistic upper bounds).

## Security

- No committed secrets spotted. `deploy.sh` relies on the operator's SSH config for
  `merah.cassia.ifost.org.au`, which is the right approach — no credentials in repo.
- These are static pages with no user data, so attack surface is minimal. Keep it
  that way; avoid adding third-party script includes.

## Housekeeping / Modernization

- **`.claude/settings.local.json` is checked in.** Confirm this is intended; local
  settings usually belong in `.gitignore`.
- Add a `.gitignore` and a top-level `deploy-all.sh` that loops the three subdir
  deploy scripts (currently you must `cd` into each).
- No Python here, so the uv migration note does not apply.

## Quick Wins

1. Fix `getEmoji` rounding / `repeat(0)` guard (5 minutes, visible bug).
2. Add a top-level README covering all three sites.
3. Add a `deploy-all.sh` wrapper.
4. Delete the unused `totalPopulation` variable and correct the misleading comment.
