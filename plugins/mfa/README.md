# MFA for Claude

MFA (Market Financial Analysis) is an invite-only research desk for US and
Borsa İstanbul stocks. This plugin lets you run your whole MFA account from
Claude: prepared analysis with verdicts that are recorded and graded, your
positions and broker fills, your watchlist and price alerts, and the
scorecard that shows which calls have held up.

## Use it

1. Install the `mfa` plugin from the `pragmaticgrowth/mfa-plugins` marketplace
   (Customize → Plugins → Add → Add marketplace).
2. Open the plugin's **Connectors** tab and connect **MFA**. Sign in with your
   MFA email and password when asked. You need an MFA account; ask the owner
   for an invite.
3. Ask in your own words, for example "XYZ'yi analiz et", "Kurulum var mı?",
   "Pozisyonlarımı göster", "Bu Midas emirlerini işle" with a screenshot
   attached, or "Hangi alarmlar kurulu?".

Alerts (stops, alert rules, setup exits, the weekday note, the Monday
scorecard) arrive on Telegram. Set your Telegram id at app.getmfa.app → Settings.

## Skills

- `mfa-analyst` — prepare, read the dossier and its setups, give a verdict
  (one entry, one stop with its exit rule, two targets, a horizon and a size
  that keeps the loss near 5% of the money planned), record it. Its
  `references/methods-evidence.md` explains what the research and MFA's own
  five-year backtest say about breakouts, momentum, Supertrend, MACD,
  TradingView scripts, earnings gaps and stops.
- `mfa-book` — positions, fills import, linking a position to its setup so
  its exit is alerted, sizing, risk balance, watchlist, memories, risk
  settings.
- `mfa-alerts` — levels, indicator and setup alerts, how a setup trade exits,
  alert rules, what fired, what was sent to Telegram and what goes in the
  weekday note.
- `mfa-scorecard` — how the recorded calls, signals and setups have
  performed, and the backtest's full tables
  (`references/backtest-2026-10.md`).

## How MFA decides when to say "enter"

MFA tested 19 entry methods on five years of US and Borsa İstanbul prices
against random entries on the same day, after costs. None beat chance, so
Claude says "Doğrulanmış kurulum yok" instead of dressing up a chart signal,
and no setup alert is pushed until a method passes the same pre-registered
test. Setups are still recorded hourly on your names and graded forward, and
the stops MFA proposes are volatility-based (3×ATR), with the position sized
so that the stop costs about 5%.

## Data

The plugin itself stores nothing. Its connector talks to
`https://api.getmfa.app/mcp/claude` over OAuth as your MFA account, and every
read and write is scoped to that account. Market facts come from MFA's own
collected data; Claude is told never to state a number no MFA tool returned.
