---
name: mfa-scorecard
description: How MFA's calls and signals have actually performed — the graded call ledger, hit rates by verdict and horizon, which inputs and recipes have earned trust, and a position's track record. Use when the user asks "nasıl gidiyoruz", "çağrıların ne kadar tuttu", "scorecard", "hangi sinyallere güvenebilirim", "HUBS'ta geçmiş kararlar", or before leaning on a call type that may have been losing.
---

# MFA scorecard

Nothing in MFA has demonstrated an edge yet, so everything — every recorded
call, card part, news event, recipe match and screen result — is graded
forward, and the scorecard is how the user learns what to trust.

## Read

- `mfa_get_scorecard` — the graded buckets: calls by verdict and horizon,
  card parts, news events, recipes. A bucket with fewer than 20 samples reads
  `not_available`: say "not enough graded samples yet", never a rate.
- `mfa_get_calls` — the ledger itself (open and graded calls, newest first;
  filter by ticker). Each graded call has its outcome against the benchmark
  (SPY for US names, XU100 for Borsa İstanbul).
- `mfa_get_track_record` — one ticker's history of calls and outcomes.
- `mfa_get_position_reviews` — past reviews of a holding.

## Answer

- Lead with the one finding that should change behaviour ("almam çağrıları 30
  örnekte %40 tuttu; bu tür kararlarda daha temkinli olmalıyım").
- Then a short table of the buckets that have enough samples: bucket, n, hit
  rate, average excess return — exactly as the tool returned them.
- Name the buckets still below 20 samples in one line, so the user knows what
  is still unknown.
- Excess returns are against the local benchmark; say which. Never average
  across currencies or markets yourself.

Reply in the user's language. This is a report of measured results — no
disclaimer unless you add a forward-looking view.
