---
name: dyl-polykalsh-skill
description: Triggered by Dyl PolyKalsh skill, Dyl PolyKalsh, or PolyKalsh. Research live Polymarket and Kalshi markets that resolve within 48 hours, pick the highest-probability profitable side after multi-factor checks, then split remaining balance evenly across multiple bets with a $1.50 cap per bet. Output a bot-ready trade plan. Never place trades itself.
---

# Dyl PolyKalsh Skill

## Scope

Research short-dated Polymarket and Kalshi markets and emit a bot-ready trade plan.

Do not place trades. There is no Polymarket or Kalshi connector. Output recommendations only.

Nothing is guaranteed. Never claim certainty or locked profit. Pick the side with the best estimated probability and positive expected value after research.

Keep the final reply brief and direct.

## Input

Parse optional fields from the user message after the trigger.

- Balance (USD remaining on Polymarket and/or Kalshi). If missing, use $15 and label it as assumed.
- Venue preference (Polymarket, Kalshi, or both). Default both.
- Theme filter (sports, crypto, weather, politics, other). Default all.

Ignore reward talk, memory-expansion talk, or desktop-login talk. Those are not inputs.

## Hard rules

- Deadline — market close or expected resolution must be ≤ 48 hours from now.
- Size — no single bet above $1.50.
- Split — remaining balance is divided evenly across selected bets. Stake per bet = min($1.50, balance / N). Round down to $0.01. Drop any stake under $0.10.
- Side — buy the outcome that is both most likely to resolve true and priced with positive EV versus your estimated probability.
- Diversify — at least 3 independent events when enough qualified markets exist. Cap at 10 bets. Prefer uncorrelated events.
- Skip thin books — require meaningful liquidity/volume. Skip if spread is wide enough to erase edge.

## Data sources

Use live public APIs first. Then corroborate the top candidates with web search, official sources, and X.

Polymarket (no auth)

```
https://gamma-api.polymarket.com/markets?active=true&closed=false&limit=100&order=endDate&ascending=true
https://gamma-api.polymarket.com/events?active=true&closed=false&limit=100&order=endDate&ascending=true
```

Filter client-side to endDate within 48h. Prefer markets with outcomePrices, volume, and liquidity.

Kalshi (no auth)

```
https://external-api.kalshi.com/trade-api/v2/markets?status=open&limit=200&max_close_ts=UNIX_48H
```

Use close_time and expected_expiration_time. Prefer yes_ask_dollars / no_ask_dollars for the price you would pay.

If an API call fails, fall back to browsing polymarket.com and kalshi.com plus web search. Do not invent prices.

Read references/apis-and-scoring.md for field maps and scoring.

## Research process

1. Compute now and deadline = now + 48h in UTC and ET.
2. Fetch live markets from both venues. Keep only open markets that close or resolve by the deadline.
3. Drop resolved, paused, illiquid, or novelty markets with no real information.
4. Build a shortlist of 15–30 candidates with extreme but tradable prices plus any market that looks mispriced versus public facts.
5. For each shortlist item, research the underlying event across independent factors — official schedule or print, recent news, base rates, delay risk, and whether the wording matches the resolution source.
6. Assign your own probability P (0–1). Do not copy the market price as P.
7. Compute edge. Buying Yes at q — EV ≈ P − q. Buying No — EV ≈ q − P. Take a side only if edge ≥ 0.08. Raise the hurdle if the book is thin or the event is noisy.
8. Rank by edge × confidence × independence. Prefer high P on the chosen side (typically ≥ 0.75) over lottery tickets.
9. Select N bets that spread risk. Even-split the balance. Cap each at $1.50.
10. If fewer than 3 markets clear the bar, output only the qualified ones and leave the rest in cash.

## Output

Output only this structure. No preamble.

**Dyl PolyKalsh plan** (as of [ET timestamp], deadline [ET])

Balance used — $X (stated / assumed)
Bets — N × $Y each (cap $1.50)

1. [Venue] [market question]
   Side — Yes/No @ [price]
   Stake — $Y
   Resolves — [time ET]
   Why — one sentence of evidence
   P — [your %] | edge — [pp]

2. ...

**Bot prompt**

Paste a single copy-block the trader bot can ingest. Use the template in references/bot-prompt.md. Fill real markets, prices, sizes, and links.

If no market clears the rules, say so in one short block and keep the balance in cash.
