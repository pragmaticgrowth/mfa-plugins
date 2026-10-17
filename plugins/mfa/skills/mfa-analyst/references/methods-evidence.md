# Methods: what the evidence says, and what MFA measured

Use this when the user asks "Supertrend işe yarıyor mu", "neden MACD'ye
güvenmiyorsun", "bu TradingView scripti iyi mi", "momentum çalışır mı",
"bilanço sonrası boşluk", or proposes a rule of their own. Two sources:
the published literature (the prior), and MFA's own 2026-10 backtests: the
setup study (19 short-term entry setups, five years of US and BIST data) and
the strategy study (portfolio rules, docs/research/2026-10-strategies). Quote
the numbers `mfa_get_strategy` returns in `evidence` in preference to the
summaries below; these summaries explain them.

## Contents

1. How MFA tests a method
2. Trend and momentum
3. Moving averages, Supertrend, MACD, Ichimoku
4. TradingView community scripts
5. Mean reversion
6. Earnings, news and other events
7. Stops and exits
8. Market regime
9. Borsa İstanbul
10. Answering "add my rule"

## 1. How MFA tests a method

- Every entry is compared with the same trade entered by every tradable name
  in the same market on the same day. The lift answers "did the method pick
  better than chance", not "did the market go up".
- Entry at the next session's open; costs 0.3% (US) / 0.6% (BIST) round trip.
- Three trade shapes: A10 (−5% stop, +10% target), A20 (−5%, +20%), B3
  (3×ATR(22) chandelier trail, no target), all with a 126-session limit.
- Development data before 2026-03-01; the holdout after it is never used to
  choose anything. Gates are fixed before the run (n ≥ 200 on ≥ 150 dates,
  after-cost mean R ≥ 0.25, hit lift ≥ 5pp, BH q ≤ 0.10, sign held in ≥ 75%
  of years, holdout mean R ≥ 0). No setup passed.
- The strategy study (portfolio rules) is pre-registered the same way:
  monthly returns net of costs, Newey-West t of the excess over a matched
  benchmark, development and holdout both positive, Sharpe and drawdown
  gates. docs/research/2026-10-strategies/PLAN.md and REPORT.md.
- With 111 groups tried, the best t-statistic from pure luck is about 2.6, so
  a single good-looking number is not evidence.

## 2. Trend and momentum

**As a portfolio, 12-1 momentum is MFA's one tested rule (momentum-1).** The
setup study tested it as a short stop-managed trade (7–19 sessions) and found
nothing; the strategy study tested it the way the literature does — hold the
top 20 names, review monthly — and found +21–23%/yr over the matched universe
(t ≈ 2.0–2.2 at every liquidity floor, positive in development and in the
2025-26 holdout), with ~48% volatility and −54% to −58% drawdowns. Adding a
market filter, a hold buffer, a 100-day stop, fewer names or 52-week-high
ranking made it worse in the first study, but those variants were all stacked on the market filter; tested alone (follow-up study, PLAN-2), a hold buffer to rank 40 kept the return with 46% less trading and a volatility-adjusted score helped, while a per-stock stop and 52-week-high ranking still hurt. Blending 50/50 with SPY kept most of the return
(~25%/yr) with SPY-like drawdowns (−21%). The investable factor ETF (MTUM,
diluted and vol-scaled) beat SPY by only ~1.8%/yr over 2014-26; concentrated
momentum is a different, riskier animal. BIST momentum lost to the BIST
universe.

| Method | Literature | MFA backtest |
|---|---|---|
| 12-1 momentum, top 20, monthly (portfolio) | Strong long-short evidence (Jegadeesh & Titman 1993); crashes after bear-market rebounds | Significant excess, high risk: the live rule's sleeve |
| 12-1 momentum as a short stop-managed trade | — | No lift in US or BIST |
| 52-week-high proximity + relative strength | George & Hwang 2004; often not significant after costs | No lift in the US; small positive holdout in BIST, failed the gates |
| Donchian 50/100/250-day breakouts | Wilcox & Crittenden 2005: ~49% win, big winners carry it, needs a wide stop | Within ±0.1R of chance in both markets (BIST slightly below); volume confirmation did not help |
| Minervini trend template, squeeze breakout, Weinstein stage 2 | No independent quantitative evidence; full CAN SLIM replication lost money | No lift |

Read: trend methods' profits in the literature come from rare big winners
held with a wide stop. A tight stop removes exactly those trades.

## 3. Moving averages, Supertrend, MACD, Ichimoku

- Golden cross and 10-month average rules halve index drawdowns but are
  "statistically indistinguishable from buy-and-hold" out of sample: risk
  control, not alpha.
- Supertrend has no academic study; a DJ30 test showed ~43% wins.
- MFA's backtest: Supertrend 10/3 (with and without an ADX/200-day filter),
  Supertrend 14/2, MACD signal cross above the 200-day, golden cross and the
  Ichimoku cloud breakout all sat at chance in both markets.
- The strategy study confirmed it on ETFs over 2005-26: the 10-month SMA on
  SPY or QQQ, dual momentum and sector rotation halved the worst drawdown
  (−24% to −34% vs −55%) but gave up 0.4–4pp of CAGR and lost clearly in the
  2019-26 bull holdout. None passed its gate.
- These remain useful as alarms (`mfa_add_indicator_alert`) when the user
  wants to be told; say they are context, not edge.

## 4. TradingView community scripts

Squeeze Momentum, UT Bot, QQE, WaveTrend, Hull MA, HalfTrend, Range Filter,
Zero-Lag MACD, Parabolic SAR: no independent evidence of an edge. Several
REPAINT — the chart shows signals that were not there in real time:
Nadaraya-Watson envelopes, anything built on pivots or zigzags, Smart Money
Concepts structure, kernel regression over a window, fixed-range volume
profile used inside its own window, Heikin Ashi fills. Lorentzian
classification's on-chart win rate is in-sample by construction. A script's
own "win rate" panel is never evidence. Offer to test the idea instead.

## 5. Mean reversion

Short-term reversal is mostly a liquidity premium in small, illiquid names.
Connors RSI(2) is a 2–10 day, high-hit, small-win trade whose edge a stop
destroys — a different shape from 3–6 months. MFA's RSI(2) pullback above
the 200-day: no lift. On BIST the Turkish literature finds contrarian effects
stronger than momentum.

## 6. Earnings, news and other events

| Signal | Literature | In MFA |
|---|---|---|
| Post-earnings gap / drift (PEAD) | Gone for US large caps since ~2006; survives in micro caps; BIST ~2.9% 60-day spread | `pead_gap` was the best US setup with the −5% / +10% shape (+0.16R lift, n = 270) but failed the gates (q 0.69) and lost in the holdout |
| Analyst estimate revisions | Alive in liquid names, 1–6 months, best with price momentum | Snapshots only since 2026-09-24: graded forward, not backtested |
| Insider buying | Opportunistic buys ~0.8%/month, small caps; sales uninformative | Rows for a small set of names since 2024-08: graded forward |
| Dilution (S-3, 424B, offerings) | Negative drift | A screen and a dossier flag, not an entry |
| 8-K impairments, non-reliance (4.02) | Negative drift | An exclusion |
| Headline sentiment (LLM) | Absorbed in ~2 days; unprofitable after costs | Not a 3–6 month signal; news direction is graded forward |

## 7. Stops and exits

- Under a random walk a stop always lowers expected return; it helps only
  when trends persist. Studies on momentum stocks favour 10–20% stops; a 5%
  stop did worse than buy-and-hold.
- Barrier arithmetic: for a driftless stock, P(+a before −b) = b ÷ (a + b):
  33% for +10/−5, 20% for +20/−5. At 30–40% annual volatility a −5% stop is
  touched within six months about 80–86% of the time, usually in 2–6 weeks.
  5% is only ~1.25–1.7 daily ATRs.
- MFA measured it: −5% hit first in 63% (+10% target) and 76% (+20%) of
  trades. The B3 chandelier starts a median 12.5% under the price.
- In the exit study (development window, the six strongest entries) exits
  that gave room — a 10% stop with no target, a close under the 50-day
  average, a fixed 63-session hold — beat random entries with the same exit
  more often than tight fixed exits or a 2×ATR trail. An observation, not a
  gated result.
- The strategy study: a stop below the 100-day average on each momentum name
  cut the excess from +23%/yr to +2.6%/yr (follow-up study, on plain momentum) — it sold the dips the names recovered
  from. momentum-1 therefore has no stop; its exit is the monthly review.
- For a discretionary position outside the strategy: wide, volatility-scaled
  stop; small enough position that the stop costs the risk the user chose.

## 8. Market regime

Index-above-200-day filters reduce drawdowns and cost a few points a year in
bull markets: risk control, not alpha. On the momentum sleeve it was worse
than that: going to cash when SPY fell under its 200-day average cost
~17pp/yr (2020-26), because it sold at the March-April 2025 and March 2026
lows and missed the rebounds. MFA saw one striking episode: in the
2026 holdout, US random entries made while SPY was under its 200-day did far
better (+0.59R) than those above it (−0.05R); in the five development years
the two were equal. One episode — mention it as an observation if asked,
never as a reason to buy.

## 9. Borsa İstanbul

- Lira inflation lifts every nominal BIST return: raw BIST mean R looks good
  for every method and for random entries alike. Judge a BIST method only by
  its lift over the same-day BIST baseline, and returns against XU100.
- Momentum evidence is weak in the region; reversal is stronger.
- Daily ±10% limits and frequent bonus or rights issues create artificial
  gaps: check corporate actions before reading a gap as news.

## 10. Answering "add my rule"

A rule of the form "buy when X, sell when Y" is a setup to test, not a
sentence to follow. Restate it precisely (entry trigger, stop, exit, horizon),
say it will be backtested the same way (a pre-registered rule, a baseline,
costs, a holdout, fixed gates) before it can push an alert, and point to
momentum-1 as the one rule that has been tested in the meantime. Never promise
the result.
