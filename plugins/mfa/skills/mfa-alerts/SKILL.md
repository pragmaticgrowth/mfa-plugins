---
name: mfa-alerts
description: This skill should be used when the user wants to propose or set stops, targets and price alarms in MFA, arm indicator alarms (Supertrend flip, RSI, moving-average and MACD crosses, Ichimoku cloud breaks), understand setup signals (entry, provisional, near-trigger, exit) and how a setup trade exits, arm a recorded call's levels, list or cancel alert rules, find out what fired, or see exactly what MFA sent to Telegram — "stop kur", "alarm kur", "hedef koy", "200'ü geçerse haber ver", "Supertrend dönerse haber ver", "RSI 30'un altına inerse", "kurulum alarmı", "geçici tetik ne demek", "çıkış alarmı", "iz süren stop nasıl çalışıyor", "hangi alarmlar kurulu", "ne tetiklendi", "Telegram'a ne gönderdin", "neden bildirim gelmedi", "günün notunda ne var". Verdicts, and where a buy's stop belongs, are mfa-analyst; bookkeeping is mfa-book.
---

# MFA alerts

Telegram is MFA's alert channel only: the user works in Claude, and MFA
pushes stops, alert rules, the weekday note and the Monday scorecard to their
Telegram. Nobody chats with the bot. The Telegram id is set on the MFA
dashboard (app.getmfa.app → Settings), which can also send a test message;
when alerts are not arriving, point the user there.

Never volunteer a level. Speak a level only when the user asks for one, or as
part of an `mfa-analyst` verdict.

## Propose a level

Where to put the stop for a buy, or for a holding's thesis, is an
`mfa-analyst` verdict; hand it over. Here, list candidates for a level the
user wants to set:

1. `mfa_get_price_structure` and `mfa_propose_levels` — stop candidates are
   support zones below the price, targets resistance zones above. Both only read.
2. One line per candidate, strongest evidence first, with its source zone and
   its distance in % and in ATRs. For a stop, always include the 3×ATR(22)
   level (the setup chandelier; `atr22` from `mfa_get_setups`) beside the
   support zones, and say that a stop about 5% away was hit first in 63–76%
   of backtest trades.
3. `not_available` (thin history, no zone) → say exactly that; never a round
   number or a percentage guess.
4. Wait until the user names ONE number. A nod at a list is not a choice —
   ask which.

## Set it

- A holding's exit: `mfa_update_position` with `stop_price` / `target_price`
  (decimal string; `null` clears it).
- "Tell me when it crosses X" is an alarm, even on a name they hold:
  `mfa_add_watchlist` (new name) or `mfa_update_watchlist` with `alert_above`
  / `alert_below`. That does not move their stop or target.
- A recorded call's levels: `mfa_arm_call_alerts` with the call's id and the
  levels the user agreed to. `mfa_list_alert_rules` shows what is armed;
  `mfa_cancel_alert_rule` removes one on the user's word.

Confirm in one line what was stored and what it will push.

## Indicator alerts

"Supertrend dönerse haber ver", "RSI 30'un altına inerse", "200 günlüğü
kırarsa", "golden cross olursa": `mfa_add_indicator_alert`. Conditions:
Supertrend flip up/down, RSI(14) crossing a threshold, the close crossing the
20/50/100/200-day average, golden/death cross (50/200), MACD crossing its
signal line, Ichimoku cloud break up/down.

- Judged on the **daily close**, only on sessions after arming: "kapanışta
  bakılır, bugünden sonraki seanslar".
- One-shot by default; `repeat: true` keeps it armed (at most one push per
  session). Ask when it isn't obvious.
- These are `context`, not edge: the 2026-10 five-year backtest measured
  Supertrend flips, golden cross, MACD cross and Ichimoku breakouts and none
  beat a random entry on the same day. Say so in one sentence, then arm it
  anyway if the user still wants it — it is their alarm, and its outcomes
  are graded in the scorecard.
- Confirm in one line with the rule's own `description`.

## Setup signals

MFA raises these itself, hourly in session, on held and watched names; there
is nothing to arm. Read them with `mfa_get_setups` (`ticker` or `scope`), the
dossier's `setups` section, or `mfa_list_signals`.

- `setup_entry` — a setup fired on a confirmed daily close.
- `setup_provisional` — it fired on today's hourly bars: "saat içinde geçici
  tetik; kapanış teyit eder". The close decides; if the close does not hold, the
  signal expires.
- `setup_near` — the price is within 0.5 ATR of a breakout trigger.
- `setup_exit` — a position opened on a setup (linked with
  `setup_signal_id`, see `mfa-book`) hit that setup's exit: `stop`, `trail`,
  `target` or `time`.

**How a setup trade exits** is the signal's `exit_rule` (all shapes in
`mfa_get_setups` → `shapes`). An unproven setup uses shape B3: the stop
starts 3×ATR(22) under the entry, rises after each close to (highest high
since entry − 3×ATR(22)), never falls, and has no profit target; the trade
also ends after about six months. To be told when it exits, link the
position (`mfa-book`); MFA then sends `setup_exit` by itself. To watch a
level without a position, arm a watchlist alarm at the signal's `stop`.

**What reaches the user, and where:**

| Kind | Telegram push | Weekday note ("Trenler") | Only in `mfa_get_setups` / dossier |
|---|---|---|---|
| `setup_entry` / `setup_provisional`, `tested_live` | yes (own budget, see below) | yes | — |
| `setup_entry` / `setup_provisional`, `watch` | no | yes | — |
| `setup_entry` / `setup_provisional`, `context` | no | no | yes |
| `setup_near` | no | no | yes |
| `setup_exit` on `stop` / `trail` | always, any hour | yes | — |
| `setup_exit` on `target` / `time` | in session, within budget | yes | — |

After the 2026-10 backtest no setup is `tested_live` (`live_count` is the
current truth), so the absence of setup pushes and of a "Trenler" section is
expected, not a fault.

## What fired, and what was sent

- `mfa_list_signals` — what the engine raised (stops, alert rules
  `rule_touch` / `rule_close`, indicator rules, watch alarms, setup kinds, and
  context kinds such as volume spikes or material filings). Say what fired in
  one sentence and what it means for the thesis in one more — never an
  instruction to act. `mfa_mark_signals_handled` only when the user says it
  is handled.
- `mfa_get_telegram_log` — the exact messages pushed, newest first, with the
  text and whether delivery succeeded. Use it for "what did you send me",
  "did the stop alert go out", "why didn't I get the note".
- Not every signal is pushed, by design: a budget of three pushes a day per
  person, a five-trading-day cooldown for the same alarm at the same level,
  and hours outside the session and macro-data days defer the quieter kinds
  to the weekday note. Setup pushes have their own separate budget of three a
  day and are never deferred for a macro-data day. Stops and a call's stop
  rule always push. A signal with a null `delivered_at` and no Telegram log
  row was held for the note (or, for an unproven setup or `setup_near`, only
  recorded), not lost; name the likely reason (budget, cooldown, outside the
  session, macro day, a digest-only kind, a setup that is not `tested_live`)
  as likely, not certain.
- The weekday note lists the user's own alarms by holding and watchlist, a
  "Trenler" section first when a `tested_live` or `watch` setup or a setup
  exit fired (a `watch` setup is shown for information, not as validated
  timing), then
  events, filings, earnings dates and stale prices. It stays silent on a
  quiet day.

## Rules

- Every level is a decimal string in the ticker's own currency.
- A quoted price carries its `as_of`; never mix currencies.
- No opinion or forecast here unless finishing an analysis — then the
  `mfa-analyst` rules and its one disclaimer line apply.
