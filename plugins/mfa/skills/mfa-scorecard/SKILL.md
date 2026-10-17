---
name: mfa-scorecard
description: This skill should be used when the user asks how MFA's strategy, calls and signals have actually performed, or what the backtests found — the momentum-1 forward record vs SPY, the graded call ledger, hit rates by verdict and horizon, the strategy and setup backtests' results, and a ticker's track record. Trigger phrases include "nasıl gidiyoruz", "strateji ne kazandırdı", "çağrıların ne kadar tuttu", "scorecard", "hangi sinyallere güvenebilirim", "kurulumlar tutuyor mu", "backtest ne dedi", "XYZ'de geçmiş kararlar", and before leaning on a call type that may have been losing. Why a method is or is not trusted, and where a stop belongs, is mfa-analyst.
---

# MFA scorecard

The Monday Telegram scorecard carries the same numbers as
`mfa_get_scorecard`. One rule has passed a pre-registered backtest
(momentum-1, significant but high-risk); its forward record is building month
by month. Every recorded call and news event is graded forward too. Report it
plainly, small samples included.

## Read

- `mfa_get_strategy` → `track` — each monthly list held from the open after
  its review to the next one, equal weight, after costs, vs SPY over the same
  dates, plus `cumulative` / `spyCumulative`; `evidence` — the backtest.
  Under about a year of rows this is not evidence either way; say how many
  months there are.
- `mfa_get_scorecard` — the graded buckets: calls by verdict and horizon,
  and news events. A bucket with fewer than 20 samples reads
  `not_available`: say "not enough graded samples yet", never a rate.
- `mfa_get_calls` — the ledger (open and graded calls, newest first; filter by
  ticker), each graded against the local benchmark (SPY for US, XU100 for
  Borsa İstanbul).
- A cut the scorecard does not make (by month, by ticker, by market, only
  calls with a stop): `mfa_describe_data`, then `mfa_query_data` over
  `my_calls` joined to `my_call_outcomes`. Apply the same rule: under 20 graded
  samples in a bucket is "not enough yet", never a rate.

## The backtests

- **Strategy study** (2026-10-17, docs/research/2026-10-strategies): 13
  pre-registered portfolio rules. Only plain 12-1 momentum (top 20, monthly)
  showed significant excess over the matched universe (+21–23%/yr, t ≈ 2.0,
  positive in development and in the 2025-26 holdout) — but with ~48%
  volatility and a −54% drawdown it failed the risk gates alone, so it runs
  blended with SPY (50/50: ~25%/yr, −21% worst drawdown vs SPY 16% / −25% in 2020–26; in the 1927–2026 survivorship-free history the mix fell −50% in 2008–09, like the market, and momentum's recent edge was ~3.7%/yr, not 23%).
  Market filters, stops, buffers, BIST momentum and ETF timing all failed.
  Quote `mfa_get_strategy` → `evidence`.
- **Setup study** (2026-10-07, 5 years, 4,047 names, 19 setups × 3 trade
  shapes, 1.59M trades, after costs): no short-term entry setup beat a random
  entry in the same market on the same day; the setup engine was stopped on
  2026-10-17. `references/backtest-2026-10.md` has the full tables (ranking,
  stop rates by shape, the market baseline by regime, the holdout) for a
  deeper question. The literature behind each method is `mfa-analyst`'s
  `references/methods-evidence.md`.
- **How a rule earns trust:** only by passing pre-registered gates on data it
  never saw. A good forward month alone proves nothing; a year of forward
  months against SPY starts to.

## Answer

- Lead with the one finding that should change behaviour ("almam çağrıları 30
  örnekte %40 tuttu; bu tür kararlarda daha temkinli olmalıyım").
- Then a short table of the buckets with enough samples, exactly as returned:
  bucket, n, hit rate, average excess return; and the strategy's forward
  months vs SPY.
- Name the buckets still below 20 samples in one line.
- Excess returns are against the local benchmark; say which. Never average
  across currencies or markets yourself.

Reply in the user's language. This is a report of measured results — no
disclaimer unless the answer adds a forward-looking view.
