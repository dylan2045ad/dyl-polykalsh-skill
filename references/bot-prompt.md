# Bot prompt template

Fill every bracket. Delete unused venue lines. This block is what the skill pastes under **Bot prompt**.

```
POLYKALSH TRADE PLAN
Generated: [ISO timestamp ET]
Account balance to deploy: $[balance]
Max per bet: $1.50
Even split: [N] bets × $[stake]
Max horizon: 48 hours from generation time
Do not place any order that violates size, deadline, or missing-price checks.

RULES FOR THE BOT
- Place only the listed orders.
- If a price has moved more than 3 cents against the plan, skip that market.
- If the book cannot fill the stake at or inside the limit, skip. Do not chase.
- If a market is no longer open or now resolves after the deadline, skip.
- Unused cash stays unallocated.

ORDERS
1) Venue: [Polymarket|Kalshi]
   Market: [full question]
   Ticker/slug: [id]
   URL: [url]
   Side: [YES|NO]
   Limit price: [0.xx]
   Stake USD: [x.xx]
   Deadline: [ISO]
   Thesis: [one line]

2) ...
```
