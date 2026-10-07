---
name: mfa-book
description: This skill should be used when the user wants to see or change their MFA book — positions and P&L, importing broker fills (Midas exports or screenshots), adding, editing, closing or deleting positions, linking a position to the setup signal it was bought on, sizing a trade ("ne kadar alayım"), checking risk balance, the watchlist, remembered preferences and per-currency risk settings. Trigger phrases include "pozisyonlarımı göster", "bu emirleri işle", "ortalamam yanlış", "XYZ'yi sattım", "kurulum alarmıyla aldım", "ne kadar alayım", "defter dengeli mi", "watchlist'e ekle", "riskimi 1%'e çek", or a pasted or attached trade history. Proposing levels and alarms is mfa-alerts; what to buy is mfa-analyst.
---

# MFA book

Bookkeeping: facts about the user's own money. No opinions, no disclaimer.
Every write waits for the user's explicit words in the same turn — never
inferred from a review or a sizing answer.

## Money rules

- Every amount carries its currency. TRY and USD are separate books; never sum
  or convert them (MFA has no FX rate).
- A price carries its source and date (`daily_close` with the date, or
  `intraday_mark` with its time). State staleness once, above the answer.
- `null` is missing, never zero. `not_available` means MFA never collected it.
- Quantities and prices written to the book are decimal STRINGS, exactly as
  confirmed.

## Show the book

`mfa_list_positions { "status": "open" }` (`mfa_get_ticker` for one name's
fresh mark). One or two names: a sentence each. Three or more: a table —
Ticker | Qty | Avg cost | Now | P&L — with one subtotal line per currency from
the tool's own `totals`, never one combined total. Closed history:
`status: "closed"`; a ticker's executions: `mfa_get_fills`.

## Import fills (broker history)

Read every row of the pasted or attached Midas export, CSV or screenshots:

- Keep only executed rows (`Gerçekleşti` / Filled); skip `İptal`,
  `Süresi Doldu`, `Bekliyor` / cancelled, expired, pending.
- Per row: ticker, `side` (`Alış` → `buy`, `Satış` → `sell`), the **filled**
  quantity and the **average fill** price (never the ordered quantity or the
  limit price), `currency`, `executed_at` as written (no invented timezone),
  and `fee` / `external_id` when shown.
- A row that cannot be read completely is named and left out, never guessed.
  If the export's stated row count differs from the rows parsed, stop and say
  so.

Show the rows as a table and say this recomputes each affected ticker's
quantity and average cost from its whole ledger. After their "evet" / "yes",
send the whole export in ONE `mfa_record_fills` call (re-sending is safe;
duplicates come back as `duplicate`). Confirm one line per ticker: the new
quantity and average, and whether the average excludes fees.

## Add, edit, close, delete

- `mfa_add_positions` — a holding with no execution history (quantity, average
  entry, entry date; currency follows the ticker unless they say otherwise).
- `mfa_update_position` — note, stop, target, currency, quantity / average,
  and `setup_signal_id`. Levels they
  dictate are theirs; proposing levels is `mfa-alerts`.
- `mfa_close_position` — they actually **sold**. It records a sale.
- `mfa_delete_positions` — a holding that was never theirs (a typo, a test
  row). It erases the position and its signal / outcome / review history; say
  what will be erased and wait for a clear yes.
- `mfa_delete_all_positions` — only on an explicit ask to wipe the whole book,
  after listing what will be erased and their confirmation; it needs the exact
  string `"DELETE ALL POSITIONS"`.

One confirmation line per row written.

## Bought on a setup alert

When they say they bought on a setup signal ("kurulum alarmıyla aldım"), find
the signal with `mfa_get_setups { "ticker": ... }` or `mfa_list_signals`,
confirm which one in one line, then link it: `mfa_update_position` with
`setup_signal_id` = the signal's `id` from `mfa_get_setups`
(it reads like `setup:XYZ:donchian_50:2026-10-06`). MFA then sends that setup's exit
(stop, trail, target or time) as a `setup_exit` alert; `null` unlinks. Record
the position first (fills or `mfa_add_positions`) if it is not in the book yet.
Tell the user the exit in one sentence from the signal's `exit_rule` — for an
unproven setup, a 3×ATR(22) trailing stop that only rises, no profit target,
about six months at most — and that a stop or trail exit always pushes.

## Sizing — "ne kadar alayım"

`mfa_get_risk_settings`, then `mfa_position_size { "ticker": ... }` (send
`entry_price` / `stop_price` only if they named them). Two sentences: the
share count with its currency, then the one binding constraint. On
`not_available` with no sum named, relay the reason in one sentence and stop —
never size against a guessed equity. If the user named a sum for the trade,
use the formula below and show the arithmetic. The ~5% loss is carried by size, not by a tight
stop: with a named sum, position = sum × 5% ÷ stop distance %, never more
than the sum (a setup's
3×ATR stop is usually 10–13% away, so the position is about 40–50% of the
sum).
The full reasoning is in `mfa-analyst`.

## Risk balance — "defter dengeli mi"

Per open name, from `mfa_position_size` inputs (entry, stop, ATR) and the
current mark: each name's share of the book's risk to stops, per currency. One
sentence naming what dominates ("riskin yarısı tek isimde, sermayenin üçte
biri orada").

## Watchlist, memories, risk settings

- `mfa_list_watchlist` / `mfa_get_watchlist_overview` to read;
  `mfa_add_watchlist`, `mfa_update_watchlist` (note, alert levels),
  `mfa_remove_watchlist` on their words. Price alarms are `mfa-alerts`.
- `mfa_get_memories` before any substantial reply. `mfa_set_memory` only when
  they state a preference, in their own words; `mfa_delete_memory` only when
  they say to drop it.
- `mfa_get_risk_settings` / `mfa_set_risk_settings` — one book per currency
  (equity, risk per trade). Change only what they named; read the result back.
- A ticker with no data yet: `mfa_request_coverage` queues it; say it fills in
  within minutes, and give no numbers until it has.
