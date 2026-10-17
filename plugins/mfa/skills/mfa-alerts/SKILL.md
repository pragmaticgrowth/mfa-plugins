---
name: mfa-alerts
description: This skill should be used when the user wants to propose or set stops, targets and price alarms in MFA, arm indicator alarms (Supertrend flip, RSI, moving-average and MACD crosses, Ichimoku cloud breaks), understand the monthly strategy review message, arm a recorded call's levels, list or cancel alert rules, find out what fired, or see exactly what MFA sent to Telegram — "stop kur", "alarm kur", "hedef koy", "200'ü geçerse haber ver", "Supertrend dönerse haber ver", "RSI 30'un altına inerse", "aylık liste mesajı", "ne zaman sat mesajı gelir", "çıkış alarmı", "hangi alarmlar kurulu", "ne tetiklendi", "Telegram'a ne gönderdin", "neden bildirim gelmedi", "günün notunda ne var". Verdicts, and where a buy's stop belongs, are mfa-analyst; bookkeeping is mfa-book.
---

# MFA alerts

Telegram is MFA's alert channel only: the user works in Claude, and MFA
pushes the monthly strategy review, stops, alert rules, the weekday note and
the Monday scorecard to their Telegram. Nobody chats with the bot. The Telegram id is set on the MFA
dashboard (app.getmfa.app → Settings), which can also send a test message;
when alerts are not arriving, point the user there.

Never volunteer a level. Speak a level only when the user asks for one, or as
part of an `mfa-analyst` verdict.

## Propose a level

Where to put the stop for a buy, or for a holding's thesis, is an
`mfa-analyst` verdict; hand it over. Here, list candidates for a level the
user wants to set:

1. `mfa_get_dossier` → the `technical` section: structure zones, ATR(22) and
   the stop/target candidates with their arithmetic — stop candidates are
   support zones below the price, targets resistance zones above. It only
   reads. For a zone over a different window, `mfa_query_data` on
   `daily_bars`.
2. One line per candidate, strongest evidence first, with its source zone and
   its distance in % and in ATRs. For a stop, always include the 3×ATR(22)
   level (the technical card's ATR) beside the support zones, and say that a
   stop about 5% away was hit first in 63–76% of backtest trades. For a name
   held under momentum-1, say first that the strategy has no stop (a stop
   tested worse); its exit is the monthly review.
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
- These are context, not edge: the 2026-10 five-year backtest measured
  Supertrend flips, golden cross, MACD cross and Ichimoku breakouts and none
  beat a random entry on the same day. Say so in one sentence, then arm it
  anyway if the user still wants it — it is their alarm.
- Confirm in one line with the rule's own `description`.

## The monthly strategy review (get on / get off the train)

MFA sends it by itself; there is nothing to arm. On the evening of the last
US session of each month (after the US close), each user with the strategy on
gets one message: the month's top-20 list applied to their USD book — SAT
(held, dropped off the list), AL (new names, the USD slice each), TUT, and the
core (SPY) adjustment — with the date the trades are meant for (the next
session's open). `mfa_get_strategy` shows the same plan at any time;
`mfa_set_strategy_settings` turns the message off or changes capital and the
core share. Nothing is pushed between reviews: the rule trades monthly, and a
mid-month move is not a signal.

## What fired, and what was sent

- `mfa_list_signals` — what the engine raised. Each row has a `category`:
  `alert` (stops and targets, alert rules `rule_touch` / `rule_close` /
  indicator rules, watch alarms, material filings, high-impact news,
  unusual moves, earnings soon) or `context` (moving-average and MACD
  crosses, 52-week extremes, RSI, volume spikes, gaps — readings with no
  tested edge). `totals` counts the whole record. Events from systems
  switched off on 2026-10-17 (setups, relative-strength flips, thesis
  breaks) are left out unless `include_retired: true`. Say what fired in
  one sentence and what it means for the thesis in one more — never an
  instruction to act. `mfa_mark_signals_handled` only when the user says it
  is handled.
- `mfa_get_telegram_log` — the exact messages pushed, newest first, with the
  text and whether delivery succeeded. Use it for "what did you send me",
  "did the stop alert go out", "why didn't I get the note".
- Not every signal is pushed, by design: a budget of three pushes a day per
  person, a five-trading-day cooldown for the same alarm at the same level,
  and hours outside the session and macro-data days defer the quieter kinds
  to the weekday note. Stops and a call's stop rule always push. A signal
  with a null `delivered_at` and no Telegram log row was held for the note,
  not lost; name the likely reason (budget, cooldown, outside the session,
  macro day, a digest-only kind) as likely, not certain.
- The weekday note lists the user's own alarms by holding and watchlist, then
  events, filings, earnings dates and stale prices. It stays silent on a
  quiet day.

## Rules

- Every level is a decimal string in the ticker's own currency.
- A quoted price carries its `as_of`; never mix currencies.
- No opinion or forecast here unless finishing an analysis — then the
  `mfa-analyst` rules and its one disclaimer line apply.
