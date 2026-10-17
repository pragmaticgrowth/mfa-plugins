# MFA for Claude

MFA (Market Financial Analysis) is an invite-only research desk for US and
Borsa İstanbul stocks. This plugin lets you run your whole MFA account from
Claude: the one strategy MFA has tested (what to buy, what to sell, monthly),
prepared analysis with verdicts that are recorded and graded, your positions
and broker fills, your watchlist and price alerts, and the scorecard.

## Use it

1. Install the `mfa` plugin from the `pragmaticgrowth/mfa-plugins` marketplace
   (Customize → Plugins → Add → Add marketplace).
2. Open the plugin's **Connectors** tab and connect **MFA**. Sign in with your
   MFA email and password when asked. You need an MFA account; ask the owner
   for an invite.
3. Ask in your own words, for example "Portföyümle ne yapayım?", "Bu ayın
   listesi ne?", "XYZ'yi analiz et", "Pozisyonlarımı göster", "Bu Midas
   emirlerini işle" with a screenshot attached, or "Hangi alarmlar kurulu?".

The monthly strategy review, stops, alert rules, the weekday note and the
Monday scorecard arrive on Telegram. Set your Telegram id at app.getmfa.app →
Settings.

## Skills

- `mfa-analyst` — strategy-first answers from momentum-1 (this month's list
  applied to your book), single-name verdicts with their evidence label,
  recorded and graded, and — when no other tool answers — a read-only SQL
  query over MFA's market data and your own rows. Its
  `references/methods-evidence.md` explains what the
  research and MFA's own backtests say about momentum, breakouts, Supertrend,
  MACD, TradingView scripts, earnings gaps, market filters and stops.
- `mfa-book` — positions, fills import (including a review's trades), sizing,
  risk balance, watchlist, memories, strategy capital, risk settings.
- `mfa-alerts` — levels, indicator alerts, the monthly review message, alert
  rules, what fired, what was sent to Telegram and what goes in the weekday
  note.
- `mfa-scorecard` — the strategy's forward record vs SPY, how the recorded
  calls have performed, and the backtests' tables.

## How MFA decides when to get in and out

MFA tested 19 short-term entry methods on five years of US and Borsa İstanbul
prices against random entries on the same day, after costs: none beat chance.
It then tested 13 portfolio rules, pre-registered, with a holdout, and a
follow-up after two outside reviews. What held up: hold 20 US stocks ranked
by 12-month momentum (skipping the last month) divided by their volatility,
keep a name while it stays in the top 40, review monthly, next to a core in
SPY. No stops and no market-timing exits — both were tested and lost money.
In 2020-26 the 50/50 mix made about 25% a year with a −21% worst drawdown
(SPY: 16%, −24%); survivorship-free data since 1927 says expect much less of
an edge than that, and market-like falls in a crash. It is evidence, not a
promise, and MFA keeps grading every monthly list.

## Data

The plugin itself stores nothing. Its connector talks to
`https://api.getmfa.app/mcp/claude` over OAuth as your MFA account, and every
read and write is scoped to that account. Market facts come from MFA's own
collected data; Claude is told never to state a number no MFA tool returned.
