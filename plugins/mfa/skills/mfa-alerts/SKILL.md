---
name: mfa-alerts
description: Price levels, indicator alerts and Telegram alerts in MFA — proposing and setting stops, targets and alarms, indicator alarms (Supertrend flip, RSI, moving-average and MACD crosses, Ichimoku cloud breaks), arming a recorded call's levels, listing or cancelling alert rules, explaining what fired, and showing exactly what MFA sent to Telegram. Use when the user says "stop kur", "alarm kur", "hedef koy", "200'ü geçerse haber ver", "Supertrend dönerse haber ver", "RSI 30'un altına inerse", "hangi alarmlar kurulu", "ne tetiklendi", "Telegram'a ne gönderdin", or asks why they did or didn't get an alert.
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

1. `mfa_get_price_structure` and `mfa_propose_levels` for the ticker — stop
   candidates are support zones below the price, target candidates resistance
   zones above. Both only read.
2. One line per candidate, strongest evidence first, with its source zone,
   e.g. "₺347,10 — below the 352–355 zone; tested 4 times, last on 12 August".
   `not_available` (thin history, no zone) → say exactly that; never a round
   number or a percentage guess.
3. Wait until they name ONE number. A nod at a list is not a choice — ask which.

## Set it

- A holding's exit: `mfa_update_position` with `stop_price` / `target_price`
  (decimal string; `null` clears it).
- "Tell me when it crosses X" is an alarm, including on a name they hold:
  `mfa_add_watchlist` (new name) or `mfa_update_watchlist` with `alert_above`
  / `alert_below`. That does not move their stop or target.
- A recorded call's levels: `mfa_arm_call_alerts` with the call's id and the
  levels they agreed to arm. `mfa_list_alert_rules` shows what is armed;
  `mfa_cancel_alert_rule` removes one on their word.

Confirm in one line what was stored and what it will push.

## Indicator alerts

"Supertrend dönerse haber ver", "RSI 30'un altına inerse", "200 günlüğü
kırarsa", "golden cross olursa": arm them with `mfa_add_indicator_alert`.
Conditions: Supertrend flip up/down, RSI(14) crossing a threshold, the close
crossing the 20/50/100/200-day average, golden/death cross (50/200), MACD
crossing its signal line, and an Ichimoku cloud break up/down.

- They are judged on the **daily close**, and only on sessions after the
  rule was armed. Say so: "kapanışta bakılır, bugünden sonraki seanslar".
- One-shot by default; `repeat: true` keeps it armed (at most one push per
  session). Ask which they want when it isn't obvious.
- Before arming, look at the dossier's technical card for that indicator's
  label: `tested` carries a measured hit rate, `context` means it has shown
  no edge across MFA's 18-month replay (Supertrend is `context` today). Tell
  the user in one sentence, then arm it anyway if they still want it — it is
  their alarm, and its outcomes are graded in the scorecard.
- Confirm in one line with the rule's own description.

## What fired, and what was sent

- `mfa_list_signals` — what the engine raised (stops, alert rules
  `rule_touch` / `rule_close`, watch alarms, and context kinds such as volume
  spikes or material filings). Explain what fired in one sentence and what it
  means for the thesis in one more — never an instruction to act.
  `mfa_mark_signals_handled` only when they say it's handled.
- `mfa_get_telegram_log` — the exact messages MFA pushed to their Telegram,
  newest first, with the text and whether delivery succeeded. Use it for
  "what did you send me", "did the stop alert go out", "why didn't I get the
  note".
- Not every signal is pushed, by design: a push budget of three a day per
  person, a five-trading-day cooldown for the same alarm at the same level,
  hours outside the trading session and macro-data days defer the quieter kinds to the weekday note.
  Stops and a call's stop rule always push. A signal whose `delivered_at` is
  null and that has no row in the Telegram log was held for the note, not
  lost; name the likely reason (budget, cooldown, outside the session, macro day,
  or a digest-only kind) as likely, not certain.

## Rules

- Every level is a decimal string in the ticker's own currency.
- A price you quote carries its `as_of`; never mix currencies.
- No opinion or forecast here unless you are finishing an analysis — then the
  `mfa-analyst` rules and its one disclaimer line apply.
