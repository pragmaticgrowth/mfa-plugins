---
name: mfa-alerts
description: Price levels, indicator and setup alerts, and Telegram alerts in MFA — proposing and setting stops, targets and alarms, indicator alarms (Supertrend flip, RSI, moving-average and MACD crosses, Ichimoku cloud breaks), setup signals (entry, provisional, near-trigger, exit), arming a recorded call's levels, listing or cancelling alert rules, explaining what fired, and showing exactly what MFA sent to Telegram. Use when the user says "stop kur", "alarm kur", "hedef koy", "200'ü geçerse haber ver", "Supertrend dönerse haber ver", "RSI 30'un altına inerse", "kurulum alarmı", "geçici tetik ne demek", "hangi alarmlar kurulu", "ne tetiklendi", "Telegram'a ne gönderdin", or asks why they did or didn't get an alert.
---

# MFA alerts

Telegram is MFA's alert channel only: the user works here in Claude, and MFA
pushes stops, alert rules, the weekday note and the Monday scorecard to their
Telegram. Nobody chats with the bot. The user sets their Telegram id on the
MFA dashboard (app.getmfa.app → Settings), where they can also send a test
message; if alerts aren't arriving, send them there.

Never volunteer a level. Levels are spoken only when the user asks for one, or
as part of an `mfa-analyst` verdict.

## Propose a level

1. `mfa_get_price_structure` and `mfa_propose_levels` — stop candidates are
   support zones below the price, targets resistance zones above. Both only read.
2. One line per candidate, strongest evidence first, with its source zone and
   its distance in % and in ATRs. A stop inside one ATR of the price is noise;
   say so if one is.
3. `not_available` (thin history, no zone) → say exactly that; never a round
   number or a percentage guess.
4. Wait until they name ONE number. A nod at a list is not a choice — ask which.

## Set it

- A holding's exit: `mfa_update_position` with `stop_price` / `target_price`
  (decimal string; `null` clears it).
- "Tell me when it crosses X" is an alarm, even on a name they hold:
  `mfa_add_watchlist` (new name) or `mfa_update_watchlist` with `alert_above`
  / `alert_below`. That does not move their stop or target.
- A recorded call's levels: `mfa_arm_call_alerts` with the call's id and the
  levels they agreed to. `mfa_list_alert_rules` shows what is armed;
  `mfa_cancel_alert_rule` removes one on their word.

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
  anyway if they still want it — it is their alarm, and its outcomes are
  graded in the scorecard.
- Confirm in one line with the rule's own `description`.

## Setup signals

MFA raises these itself on held and watched names; there is nothing to arm.
Read them with `mfa_get_setups` (`ticker` or `scope`) or `mfa_list_signals`.

- `setup_entry` — a setup fired on a confirmed daily close.
- `setup_provisional` — it fired on an intraday provisional bar: "saat içinde
  geçici tetik, kapanışta teyit". The close decides; if the close does not
  hold, the signal expires.
- `setup_near` — the price is within 0.5 ATR of a breakout trigger.
- `setup_exit` — a position opened on a setup (linked with
  `setup_signal_id`, see `mfa-book`) hit that setup's exit.

Only setups whose status is `tested_live` push entries to Telegram; a
`setup_exit` on a stop or trailing stop always pushes. Everything else is
recorded for the weekday note and graded forward. After the 2026-10 backtest
no setup is `tested_live` (`live_count` in `mfa_get_setups` is the current
truth), so a missing setup push is expected, not a fault.

## What fired, and what was sent

- `mfa_list_signals` — what the engine raised (stops, alert rules
  `rule_touch` / `rule_close`, indicator rules, watch alarms, setup kinds, and
  context kinds such as volume spikes or material filings). Say what fired in
  one sentence and what it means for the thesis in one more — never an
  instruction to act. `mfa_mark_signals_handled` only when they say it's handled.
- `mfa_get_telegram_log` — the exact messages pushed, newest first, with the
  text and whether delivery succeeded. Use it for "what did you send me",
  "did the stop alert go out", "why didn't I get the note".
- Not every signal is pushed, by design: a budget of three pushes a day per
  person, a five-trading-day cooldown for the same alarm at the same level,
  hours outside the session and macro-data days defer the quieter kinds to the
  weekday note. Stops and a call's stop rule always push. A signal with a null
  `delivered_at` and no Telegram log row was held for the note, not lost; name
  the likely reason (budget, cooldown, outside the session, macro day, a
  digest-only kind, a setup that is not `tested_live`) as likely, not certain.

## Rules

- Every level is a decimal string in the ticker's own currency.
- A price you quote carries its `as_of`; never mix currencies.
- No opinion or forecast here unless you are finishing an analysis — then the
  `mfa-analyst` rules and its one disclaimer line apply.
