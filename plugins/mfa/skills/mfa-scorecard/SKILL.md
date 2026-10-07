---
name: mfa-scorecard
description: This skill should be used when the user asks how MFA's calls, signals and setups have actually performed, or what the setup backtest found — the graded call ledger, hit rates by verdict and horizon, which inputs, recipes and setups have earned trust, the five-year backtest's results, and a ticker's track record. Trigger phrases include "nasıl gidiyoruz", "çağrıların ne kadar tuttu", "scorecard", "hangi sinyallere güvenebilirim", "kurulumlar tutuyor mu", "backtest ne dedi", "XYZ'de geçmiş kararlar", and before leaning on a call type that may have been losing. Why a method is or is not trusted, and where a stop belongs, is mfa-analyst.
---

# MFA scorecard

The Monday Telegram scorecard carries the same numbers as
`mfa_get_scorecard`. Nothing in MFA has demonstrated an edge yet, so every recorded call, card
part, news event, recipe match, screen result and setup signal is graded
forward. The scorecard is how the user learns what to trust; report it
plainly, small samples included.

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

- **The backtest** (2026-10 run, 5 years, 4,047 names, 19 setups in the US
  and 18 on BIST × 3 trade shapes, 1.59M trades, after costs): no setup beat a random entry
  in the same market on the same day, so every one is `context`. A planted
  look-ahead control was detected clearly, so the flat result is real. The
  tools carry it: `setups.backtest` (the catalogue row per setup and market),
  and `findings` / `shapes` in `mfa_get_scorecard` → `setups` and in
  `mfa_get_setups`. Quote those; `references/backtest-2026-10.md` has the
  full tables (ranking, stop rates by shape, the market baseline by regime,
  the holdout month by month, limitations) for a deeper question. The
  literature behind each method is `mfa-analyst`'s
  `references/methods-evidence.md`.
- **Forward grading** (`setups.live`): every confirmed setup signal MFA
  recorded on held and watched names, replayed nightly on its own trade shape
  from the next open. Show the table — setup, market, n, win rate, mean R —
  and name the setups still under 20 samples (`not_available`) in one line.
  A forward table that disagrees with the backtest is a finding; say it, and
  say how small n is.
- **How a setup earns trust:** only by passing the same pre-registered gates
  on a later backtest run under a new catalogue version. A good forward run
  alone does not promote it; say so if the user asks why a winning setup
  still does not push.

## Answer

- Lead with the one finding that should change behaviour ("almam çağrıları 30
  örnekte %40 tuttu; bu tür kararlarda daha temkinli olmalıyım").
- Then a short table of the buckets with enough samples, exactly as returned:
  bucket, n, hit rate, average excess return (or mean R for setups).
- Name the buckets still below 20 samples in one line.
- Excess returns are against the local benchmark; say which. Never average
  across currencies or markets yourself.

Reply in the user's language. This is a report of measured results — no
disclaimer unless the answer adds a forward-looking view.
