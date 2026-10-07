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

Alerts (stops, alert rules, the weekday note, the Monday scorecard) arrive on
Telegram. Set your Telegram id at app.getmfa.app → Settings.

## Skills

- `mfa-analyst` — prepare, read the dossier and its setups, give a verdict
  (one entry, one stop, two targets, a horizon, a size), record it.
- `mfa-book` — positions, fills import, linking a position to its setup, sizing,
  watchlist, memories, risk settings.
- `mfa-alerts` — levels, indicator and setup alerts, alert rules, what fired,
  what was sent to Telegram.
- `mfa-scorecard` — how the recorded calls, signals and setups have performed.

## Data

The plugin itself stores nothing. Its connector talks to
`https://api.getmfa.app/mcp/claude` over OAuth as your MFA account, and every
read and write is scoped to that account. Market facts come from MFA's own
collected data; Claude is told never to state a number no MFA tool returned.
