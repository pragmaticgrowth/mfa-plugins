---
name: mfa-scorecard
description: How MFA's calls, signals and setups have actually performed — the graded call ledger, hit rates by verdict and horizon, which inputs, recipes and setups have earned trust, the setup backtest, and a position's track record. Use when the user asks "nasıl gidiyoruz", "çağrıların ne kadar tuttu", "scorecard", "hangi sinyallere güvenebilirim", "kurulumlar tutuyor mu", "backtest ne dedi", "XYZ'de geçmiş kararlar", or before leaning on a call type that may have been losing.
---

# MFA scorecard

Nothing in MFA has demonstrated an edge yet, so every recorded call, card
part, news event, recipe match, screen result and setup signal is graded
forward. The scorecard is how the user learns what to trust.

## Read

- `mfa_get_scorecard` — the graded buckets: calls by verdict and horizon, card
  parts, news events, recipes, and `setups` (`backtest` and `live`). A bucket
  with fewer than 20 samples reads `not_available`: say "not enough graded
  samples yet", never a rate.
- `mfa_get_setups` — the setup catalogue (each setup's backtest status
  `tested_live` / `watch` / `context`, `n`, `hit`, `mean_r`, `lift_r`, holdout
  figures, `live_count`) and, per ticker, each signal's forward `outcome`.
- `mfa_get_calls` — the ledger (open and graded calls, newest first; filter by
  ticker), each graded against the local benchmark (SPY for US, XU100 for
  Borsa İstanbul).
- `mfa_get_track_record` — one ticker's calls and outcomes.
- `mfa_get_position_reviews` — past reviews of a holding.

## Setups: two different measurements

- **The backtest** (2026-10 run, 5 years, 4,047 names, 19 setups × 3 trade
  shapes × US/BIST, after costs): no setup beat a random entry in the same
  market on the same day, so every one is `context`. A planted look-ahead
  control was detected clearly, so the flat result is real. These rows are
  `setups.backtest`. The full report is `docs/research/2026-10-setups/REPORT.md`
  in the MFA repo (not in this plugin); quote only what the tools return.
- **Forward grading** (`setups.live`): every confirmed setup signal MFA
  recorded on held and watched names, graded on its own trade shape. Show the
  table — setup, market, n, win rate, mean R — and name the setups still
  under 20 samples (`not_available`) in one line. A forward table that
  disagrees with the backtest is a finding; say it, and say how small n is.

## Answer

- Lead with the one finding that should change behaviour ("almam çağrıları 30
  örnekte %40 tuttu; bu tür kararlarda daha temkinli olmalıyım").
- Then a short table of the buckets with enough samples, exactly as returned:
  bucket, n, hit rate, average excess return (or mean R for setups).
- Name the buckets still below 20 samples in one line.
- Excess returns are against the local benchmark; say which. Never average
  across currencies or markets yourself.

Reply in the user's language. This is a report of measured results — no
disclaimer unless you add a forward-looking view.
