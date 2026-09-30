---
name: mfa-book
description: Keep the user's MFA book — positions and P&L, importing broker fills (Midas exports or screenshots), adding, editing, closing or deleting positions, sizing ("ne kadar alayım"), risk balance, the watchlist, remembered preferences and risk settings. Use when the user asks "pozisyonlarımı göster", "bu emirleri işle", "ortalamam yanlış", "HUBS'u sattım", "watchlist'e ekle", "riskimi 1%'e çek", or pastes/attaches a trade history.
---

# MFA book

Everything here is bookkeeping: facts about the user's own money. No opinions,
no disclaimer, and every write waits for the user's explicit words in the
same conversation turn — never inferred from a review or a sizing answer.

## Money rules

- Every amount carries its currency. TRY and USD are separate books; never sum
  or convert them (MFA has no FX rate).
- A price carries its source and date: `daily_close` with the date, or
  `intraday_mark` with its time. State staleness once, above the answer.
- `null` means missing, never zero. `not_available` means MFA never collected
  it, never "nothing happened".
- Quantities and prices you write are decimal STRINGS, exactly as confirmed.

## Show the book

`mfa_list_positions { "status": "open" }` (add `mfa_get_ticker` for one name's
fresh mark). One or two names: a sentence each. Three or more: a table —
Ticker | Qty | Avg cost | Now | P&L — with one subtotal line per currency
from the tool's own `totals` block, never one combined total. Closed history:
`status: "closed"`; executions for a ticker: `mfa_get_fills`.

## Import fills (broker history)

The user pastes or attaches a Midas export, a CSV, or screenshots of their
order history. Read every row yourself:

- Keep only executed rows (`Gerçekleşti` / Filled). Skip `İptal`,
  `Süresi Doldu`, `Bekliyor` / cancelled, expired, pending — they executed
  nothing.
- Per row: ticker, `side` (`Alış` → `buy`, `Satış` → `sell`), the **filled**
  quantity and the **average fill** price (never the ordered quantity or the
  limit price), `currency`, `executed_at` as written (don't invent a
  timezone), and `fee` / `external_id` when the source shows them.
- A row you cannot read completely is named and left out, never guessed. If
  the export states a row count that differs from what you parsed, stop and
  say so rather than importing part of it.

Show the rows you will write as a table, and say plainly that this recomputes
each affected ticker's quantity and average cost from its whole ledger. After
their "evet" / "yes", send the whole export in ONE `mfa_record_fills` call.
Re-sending the same export is safe — duplicates come back as `duplicate` and
change nothing. Confirm with one line per ticker: the new quantity and
average, and whether the average excludes fees.

## Add, edit, close, delete

- `mfa_add_positions` — a holding with no execution history (quantity, average
  entry, entry date; the currency follows the ticker unless they say otherwise).
- `mfa_update_position` — note, stop, target, trailing multiple. Stops and
  targets they dictate are theirs; for proposing levels see `mfa-alerts`.
- `mfa_close_position` — they actually **sold**. It records a sale.
- `mfa_delete_positions` — a holding that was never theirs (a typo, a test
  row). It erases the position and its signal / outcome / review history.
  Say what will be erased and wait for a clear yes.
- `mfa_delete_all_positions` — only on an explicit ask to wipe the whole book,
  after you have listed what will be erased and they confirm; it needs the
  exact string `"DELETE ALL POSITIONS"`.

One confirmation line per row written. No table restating their request.

## Sizing — "ne kadar alayım"

`mfa_get_risk_settings` then `mfa_position_size { "ticker": ... }` (send
`entry_price` / `stop_price` only if they named them). Answer in two
sentences: the share count with its currency, then the one binding constraint
("40 adet, çünkü bağlayıcı olan yoğunlaşma; bütçe 55'e izin veriyor"). If the
tool answers `not_available`, relay its reason in one sentence and stop —
never size against a guessed equity.

## Risk balance — "defter dengeli mi"

For each open name use `mfa_position_size` inputs (entry, stop, ATR) and the
current mark; compute each name's share of the book's ATR risk, per currency.
One sentence naming what dominates: "riskin yarısı NBIS'te, sermayenin sadece
üçte biri o isimde."

## Watchlist

`mfa_list_watchlist` / `mfa_get_watchlist_overview` to read.
`mfa_add_watchlist`, `mfa_update_watchlist` (note, alert levels),
`mfa_remove_watchlist` on their words. Price alarms on a watched name are
covered in `mfa-alerts`.

## Memories and risk settings

- `mfa_get_memories` before any substantial reply. `mfa_set_memory` only when
  they state a preference, in their own words; `mfa_delete_memory` only when
  they say to drop it.
- `mfa_get_risk_settings` / `mfa_set_risk_settings` — one book per currency
  (equity, risk per trade, max position). Change only what they named, and
  read the result back.

## Coverage

If a ticker they care about has no data yet, `mfa_request_coverage` queues it
for collection; tell them it will fill in within minutes, and don't answer
with numbers until it has.
