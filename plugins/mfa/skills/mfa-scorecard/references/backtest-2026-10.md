# The 2026-10 setup backtest — full tables

Read this for a deeper question than `mfa_get_setups` → `findings` answers:
"which setup came closest", "how often did the −5% stop hit", "does the
market regime matter", "what about BIST". The tool output is the live truth
(`live_count`, `catalogue`); this file is the frozen record of the
`setup-1` run generated 2026-10-07.

## Contents

1. Method and gates
2. Every setup, by market (shape B3, the live stop)
3. Stop hit rates by trade shape
4. The market itself: random entries, by regime
5. Exit study
6. Holdout
7. Limitations
8. How a setup earns trust

## 1. Method and gates

- Daily bars 2021-10-01 → 2026-10-06, 4,047 names (3,428 US listings ≥ $300M
  market cap, 619 BIST), benchmarks SPY and XU100. 19 setups in the US and 18 on BIST
  (the post-earnings gap needs stored earnings dates) × 3 shapes = 111
  groups, 1,594,771 simulated trades (non-overlapping per
  ticker, setup and shape).
- Entry at the next session's open; a gap through a level fills at the open;
  costs 0.3% US / 0.6% BIST round trip on every trade and every baseline
  trade. R = after-cost return ÷ initial risk.
- Shapes: **A10** −5% stop / +10% target; **A20** −5% / +20%; **B3**
  3×ATR(22) chandelier trail, no target. All end after 126 sessions.
- Baseline: the same shape entered by every tradable name in the same market
  on the same day. "Lift" = setup minus baseline.
- Development: signals before 2026-03-01. Holdout: 2026-03-01 onward, never
  used to choose a rule, parameter or gate.
- Gates for `tested_live`, fixed before the run: n ≥ 200, ≥ 150 distinct
  dates, mean R ≥ 0.25, hit-rate lift ≥ 5pp, lift > 0 with BH q ≤ 0.10 (t
  clustered by signal month), lift sign held in ≥ 75% of years with ≥ 20
  trades, holdout ≥ 30 trades with mean R ≥ 0. All but the holdout = `watch`.
- With 111 groups, the best t from luck alone is ≈ 2.57.
- Result: 0 `tested_live`, 0 `watch`.

## 2. Every setup, by market (shape B3, the live stop)

The live engine gives every unproven setup the B3 stop, so these are the
rows `catalogue` shows. Hit = trades closed with a gain (for A10 / A20:
the target reached before the stop). A low q next to a NEGATIVE lift means
the setup was reliably worse than random entries, not that it passed. Raw BIST mean R is
lifted by lira inflation for every method and for random entries alike;
read the lift. The best single group in any shape was the post-earnings gap
in the US with the A10 shape: n 270, lift +0.16R, q 0.69, holdout −0.27R.

| Setup | Mkt | B3 n | B3 hit | B3 mean R | B3 lift R | q | Holdout n / mean R | A10 stop hit first | A20 stop hit first |
|---|---|---|---|---|---|---|---|---|---|
| Sıkışma sonrası kırılım (trend şablonu içinde) (`tt_squeeze_breakout`) | BIST | 1,288 | 42.9% | +0.37 | +0.05 | 0.72 | 200 / -0.04 | 58.7% | 68.6% |
| Trend şablonunda 50 günlüğe ilk geri çekilme (`tt_pullback_sma50`) | BIST | 288 | 36.8% | +0.22 | +0.03 | 0.94 | 98 / -0.14 | 62.5% | 71.7% |
| RSI(2) aşırı satım (200g üstünde) (`rsi2_pullback`) | BIST | 8,919 | 41.9% | +0.26 | +0.03 | 0.57 | 1250 / -0.03 | 63.0% | 74.1% |
| 52 haftalık zirveye yakın + güçlü göreli güç (`high52_rs`) | BIST | 2,526 | 42.5% | +0.40 | +0.02 | 0.90 | 388 / +0.16 | 55.7% | 67.4% |
| MACD sinyal kesişimi (200g üstünde) (`macd_cross_trend`) | BIST | 8,226 | 42.7% | +0.34 | +0.00 | 0.55 | 1176 / -0.02 | 58.4% | 70.5% |
| 12-1 momentum ilk %10 (piyasa 200g üstünde) (`momentum_12_1`) | BIST | 1,349 | 43.7% | +0.37 | -0.00 | 0.94 | 292 / +0.13 | 57.7% | 70.2% |
| Supertrend 10/3 yukarı dönüş (200g üstü, ADX>20) (`supertrend_10_3_trend`) | BIST | 2,153 | 43.6% | +0.37 | -0.01 | 0.94 | 315 / -0.03 | 57.8% | 68.3% |
| Minervini trend şablonu açıldı (`trend_template`) | BIST | 2,571 | 40.6% | +0.29 | -0.02 | 0.85 | 531 / -0.10 | 61.0% | 73.0% |
| Golden cross (50g, 200g üstüne) (`golden_cross`) | BIST | 1,394 | 43.4% | +0.28 | -0.03 | 0.75 | 188 / -0.02 | 57.6% | 70.2% |
| Ichimoku bulut kırılımı (Tenkan > Kijun) (`ichimoku_breakout`) | BIST | 4,521 | 45.0% | +0.38 | -0.03 | 0.72 | 678 / -0.07 | 55.6% | 67.1% |
| 250 günlük zirve kırılımı (`donchian_250`) | BIST | 3,797 | 42.7% | +0.37 | -0.03 | 0.93 | 442 / +0.06 | 57.6% | 68.6% |
| Supertrend 14/2 yukarı dönüş (200g üstü, ADX>20) (`supertrend_14_2_trend`) | BIST | 4,618 | 41.3% | +0.32 | -0.03 | 0.98 | 622 / -0.07 | 61.0% | 71.8% |
| 100 günlük zirve kırılımı (`donchian_100`) | BIST | 5,275 | 42.1% | +0.37 | -0.05 | 0.69 | 580 / +0.07 | 57.6% | 68.5% |
| Supertrend 10/3 yukarı dönüş (`supertrend_10_3`) | BIST | 5,899 | 44.0% | +0.34 | -0.06 | 0.18 | 1044 / -0.07 | 56.6% | 68.3% |
| 50 günlük zirve kırılımı (`donchian_50`) | BIST | 6,785 | 42.1% | +0.37 | -0.07 | 0.57 | 749 / -0.01 | 57.5% | 68.8% |
| 100 günlük zirve kırılımı (hacimli) (`donchian_100_vol`) | BIST | 3,887 | 41.7% | +0.36 | -0.09 | 0.18 | 413 / +0.01 | 58.4% | 68.9% |
| Weinstein 2. evre girişi (`stage2`) | BIST | 1,105 | 45.1% | +0.35 | -0.11 | 0.00 | 185 / -0.18 | 59.7% | 70.1% |
| Güçlü boşluklu açılış (episodik pivot) (`power_gap`) | BIST | 570 | 34.4% | +0.08 | -0.22 | 0.08 | 74 / -0.19 | 66.0% | 76.1% |
| Bilanço sonrası tutunan yukarı boşluk (`pead_gap`) | US | 269 | 38.7% | +0.03 | +0.06 | 0.85 | 49 / -0.04 | 60.7% | 75.6% |
| RSI(2) aşırı satım (200g üstünde) (`rsi2_pullback`) | US | 48,040 | 37.4% | +0.03 | +0.02 | 0.85 | 8638 / +0.03 | 63.8% | 76.0% |
| 12-1 momentum ilk %10 (piyasa 200g üstünde) (`momentum_12_1`) | US | 9,089 | 37.4% | +0.04 | +0.02 | 0.75 | 1652 / -0.11 | 64.5% | 76.7% |
| Trend şablonunda 50 günlüğe ilk geri çekilme (`tt_pullback_sma50`) | US | 2,449 | 40.0% | +0.05 | +0.01 | 0.69 | 507 / -0.06 | 60.3% | 73.5% |
| Weinstein 2. evre girişi (`stage2`) | US | 9,031 | 38.4% | +0.06 | +0.01 | 0.85 | 1629 / +0.17 | 64.1% | 76.3% |
| MACD sinyal kesişimi (200g üstünde) (`macd_cross_trend`) | US | 45,626 | 36.2% | +0.00 | +0.00 | 0.94 | 7676 / +0.01 | 65.4% | 77.0% |
| Ichimoku bulut kırılımı (Tenkan > Kijun) (`ichimoku_breakout`) | US | 31,681 | 37.8% | +0.01 | +0.00 | 0.90 | 4694 / +0.02 | 64.2% | 76.3% |
| Minervini trend şablonu açıldı (`trend_template`) | US | 17,407 | 36.8% | +0.01 | -0.00 | 0.96 | 2788 / -0.00 | 65.0% | 76.9% |
| Sıkışma sonrası kırılım (trend şablonu içinde) (`tt_squeeze_breakout`) | US | 8,674 | 36.0% | -0.01 | -0.00 | 0.69 | 1081 / -0.10 | 65.8% | 77.4% |
| Supertrend 10/3 yukarı dönüş (`supertrend_10_3`) | US | 43,225 | 37.3% | +0.01 | -0.01 | 0.89 | 5986 / +0.02 | 64.3% | 76.2% |
| 50 günlük zirve kırılımı (`donchian_50`) | US | 36,430 | 35.6% | -0.03 | -0.01 | 0.51 | 5301 / -0.06 | 66.9% | 78.1% |
| Golden cross (50g, 200g üstüne) (`golden_cross`) | US | 9,469 | 34.8% | -0.03 | -0.01 | 0.69 | 1309 / -0.04 | 66.5% | 77.0% |
| 250 günlük zirve kırılımı (`donchian_250`) | US | 17,945 | 34.2% | -0.05 | -0.01 | 0.18 | 3007 / -0.09 | 67.1% | 77.8% |
| 100 günlük zirve kırılımı (`donchian_100`) | US | 26,427 | 34.1% | -0.05 | -0.02 | 0.23 | 4026 / -0.07 | 67.5% | 78.4% |
| Supertrend 14/2 yukarı dönüş (200g üstü, ADX>20) (`supertrend_14_2_trend`) | US | 18,864 | 35.1% | -0.03 | -0.02 | 0.55 | 2814 / +0.06 | 66.1% | 77.3% |
| Güçlü boşluklu açılış (episodik pivot) (`power_gap`) | US | 3,324 | 35.4% | -0.01 | -0.02 | 0.94 | 460 / -0.13 | 67.3% | 79.2% |
| 52 haftalık zirveye yakın + güçlü göreli güç (`high52_rs`) | US | 14,407 | 34.7% | -0.03 | -0.02 | 0.34 | 2185 / -0.08 | 67.1% | 78.5% |
| 100 günlük zirve kırılımı (hacimli) (`donchian_100_vol`) | US | 14,468 | 34.0% | -0.06 | -0.02 | 0.14 | 2192 / -0.09 | 67.5% | 78.5% |
| Supertrend 10/3 yukarı dönüş (200g üstü, ADX>20) (`supertrend_10_3_trend`) | US | 9,163 | 35.9% | -0.02 | -0.03 | 0.41 | 1332 / +0.00 | 65.2% | 76.2% |

## 3. Stop hit rates by trade shape

Medians over the 37 setup-market groups with n ≥ 100.

| Shape | Stop hit first | Hit | Mean R | Initial risk | Lift R |
|---|---|---|---|---|---|
| A10 (−5% / +10%) | 63.0% | 37.0% | +0.01 | 5.0% | −0.04 |
| A20 (−5% / +20%) | 75.6% | 23.8% | +0.08 | 5.0% | −0.04 |
| B3 (3×ATR trail) | 1.7% | 38.4% | +0.06 | 12.5% | −0.01 |

B3's "stop hit first" counts only exits at the initial stop; most B3 trades
end on the trail after it has moved up.

## 4. The market itself: random entries, by regime

Every name, every development session, after costs:

| Shape | US mean R | US hit | BIST mean R | BIST hit |
|---|---|---|---|---|
| A10 | +0.001 | 35.5% | +0.151 | 42.2% |
| A20 | +0.052 | 21.1% | +0.388 | 30.3% |
| B3 | +0.007 | 36.9% | +0.310 | 43.9% |

Split by whether the index closed above its 200-day average on entry day
(n / mean R / hit):

| Market | Shape | Window | Index above 200-day | Index below 200-day |
|---|---|---|---|---|
| US | A10 | development | 2,266,903 / +0.002 / 35.5% | 666,604 / −0.002 / 35.5% |
| US | A10 | holdout | 427,009 / −0.048 / 30.3% | 36,230 / +0.592 / 54.3% |
| US | B3 | development | 2,265,866 / −0.001 / 36.4% | 666,391 / +0.032 / 38.8% |
| US | B3 | holdout | 426,897 / −0.020 / 37.7% | 36,227 / +0.491 / 61.2% |
| BIST | A10 | development | 361,186 / +0.128 / 41.5% | 62,654 / +0.274 / 46.1% |
| BIST | A10 | holdout | 67,595 / −0.091 / 33.9% | 6,312 / −0.454 / 12.6% |
| BIST | B3 | development | 360,597 / +0.289 / 42.7% | 62,518 / +0.421 / 50.1% |
| BIST | B3 | holdout | 67,560 / −0.041 / 34.0% | 6,247 / −0.216 / 31.5% |

The US holdout "below the 200-day" cell is one rebound episode (spring 2026);
in five development years the two US regimes were equal, and BIST went the
other way. An observation, not a rule.

## 5. Exit study

Development window, the six strongest entries, risk normalised to 5% so every
exit reads in the user's risk unit; each exit compared with random entries
using the same exit. Exits that gave room — a 10% stop with no target, a close
under the 50-day average, a fixed 63-session hold — showed the largest lifts
(10% stop, no target: +0.18 to +2.24; 50-day close: +0.13 to +1.73; 63
sessions: +0.12 to +0.69), while the 2×ATR trail's lifts were small (−0.01 to
+0.18) and A10 / A20 lagged random entries (median lift −0.04). Long
holds also collect the market's drift, so the
lift, not the raw R, is the number to quote. Not gated.

## 6. Holdout

Signals from 2026-03-01 to 2026-10-06, excess return over the index at 1, 2,
3 and 6 months. Only the US post-earnings gap stayed ahead of the index at 1,
2 and 3 months (+0.4, +4.2, +9.4pp, n 49; 6 months not reached yet), yet it
lost in R (−0.27R) with the A10 shape's −5% stop. The best
3-month excess came from BIST 52-week high + RS (+11.8pp, n 388) and BIST
250-day breakouts (+9.5pp, n 442); both turned negative by 6 months (−7.8pp,
−10.8pp). US breakouts were negative at every horizon (100-day: −1.0, −1.4,
−3.0, −6.1pp).

## 7. Limitations

- Survivorship: today's universe; names that delisted since 2021 are
  missing. It flatters every long signal and the baseline alike.
- One and a half market cycles (the 2022 bear market, the 2023-25 advance,
  the April 2025 drawdown); no 2008.
- BIST in nominal lira.
- News starts August 2026, so no news rule could be tested back to March;
  insider clusters and estimate revisions are too thin. All three are graded
  forward in the scorecard instead.
- Daily bars only; slippage beyond the cost assumption is not modelled.

## 8. How a setup earns trust

A new or changed setup goes into the catalogue with its parameters fixed
before the run that grades it; a passing run publishes a new catalogue
version and only then can the setup push. Forward results in
`setups.live` are evidence for the next run, not a promotion by themselves.
