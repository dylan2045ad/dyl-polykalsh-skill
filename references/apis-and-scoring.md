# APIs and scoring

## Time window

- `now` = current UTC
- `deadline_unix` = now + 172800
- Keep a market only if close/end/expected expiration ≤ deadline
- Prefer expected resolution inside 48h, not just a far close_time with early-close flags

## Polymarket fields

Endpoint: `GET https://gamma-api.polymarket.com/markets`

Useful query params: `active=true`, `closed=false`, `limit`, `offset`, `order=endDate`, `ascending=true`, `end_date_max` (ISO8601) when accepted.

Key fields: `question`, `slug`, `endDate`, `closed`, `active`, `outcomePrices` (JSON string list), `outcomes`, `liquidityNum` / `liquidity`, `volumeNum` / `volume24hr`, `clobTokenIds`.

Link: `https://polymarket.com/event/{eventSlug}` or `https://polymarket.com/market/{slug}`

Implied Yes ≈ float(outcomePrices[0]). Implied No ≈ 1 − Yes.

Minimums (soft): volume24hr ≥ 1000 or liquidity ≥ 500. Relax only if the event is nearly certain and the book can fill $1.50.

## Kalshi fields

Endpoint: `GET https://external-api.kalshi.com/trade-api/v2/markets`

Useful query params: `status=open`, `limit=200`, `cursor`, `min_close_ts`, `max_close_ts`.

Key fields: `ticker`, `event_ticker`, `title`, `yes_sub_title`, `close_time`, `expected_expiration_time`, `status`, `yes_ask_dollars`, `no_ask_dollars`, `yes_bid_dollars`, `volume_fp`, `open_interest_fp`.

Link: `https://kalshi.com/markets/{series}/{event}` when known, else search by ticker.

Pay `yes_ask_dollars` to buy Yes. Pay `no_ask_dollars` to buy No.

Minimums (soft): enough size at the ask to fill $1.50 without walking the book more than 2 cents.

## Scoring

For candidate side S at ask q and model probability P_s:

- edge = P_s − q
- require edge ≥ 0.08 unless P_s ≥ 0.92 and the resolution source is mechanical (clock, official print, already-occurred fact)
- confidence 1–5 from source quality
- score = edge * confidence * 10 − correlation_penalty

Reject:

- Ambiguous resolution text
- Markets that can slip past 48h (delayed games, timezone traps)
- Both sides looking +EV (data error)
- Sports sides that are just market favorites with no extra information

## Sizing

```
N = min(10, max(1, number_of_qualified))
raw = balance / N
stake = min(1.50, floor(raw * 100) / 100)
if stake < 0.10: drop that slot and recompute
```

Unused remainder stays cash.
