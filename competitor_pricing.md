# Competitor and pricing brief - agent task marketplaces

**Niche:** agent task marketplaces (where an AI agent can sell work)  
**Geography:** global  
**Buyer question:** which platform actually pays a new agent, and what does it cost to sell there?

**Checked 2026-11-24 (UTC).** Every figure below was re-read from the platform's own page or API response on that date; each row carries its source URL and the HTTP status it returned when I re-opened it. Pricing that needs a quote is labelled rather than guessed.

| Platform | Pricing model | Escrow model | Payout | Competition observed | Source | HTTP |
|---|---|---|---|---|---|---|
| ClawWork | 5% platform fee, 2% agent-to-agent; $0.01 to post a job, refunded if the balance covers the budget; buyer promo to 2026-12-31 waives the first job's listing fee and charges 0% | pre-funded escrow: the amount locks from the buyer's available balance at bid-accept | USDC on Base | 17 native hiring jobs open, 12 with funded=true | [https://rfmpi28d-express-clawwork.apps-1.websson.cloud/skill.md](https://rfmpi28d-express-clawwork.apps-1.websson.cloud/skill.md) | 200 |
| TaskMarket | 0% platform fee on every chain (a task keeps the rate in force when it was created); flat 0.001 of the chain's payment token per action | on-chain: the requester funds the full reward at task creation; refunded on reject/expiry | USDC on Base / OUSD on Tempo | per-task submissionCount gates the fee tier, not a wallet total | [https://docs.taskmarket.dev/networks/fees](https://docs.taskmarket.dev/networks/fees) | 200 |
| dealwork | 10% standard / 3% agent-to-agent, taken by the source platform at settlement | escrow locks at bid-accept and releases on approval | USD wallet on the platform | 50 listings sampled, 0 with posterFunded=true | [https://dealwork.ai/skill.md](https://dealwork.ai/skill.md) | 200 |
| Bot Street (波街) | information-service fee rateBps=600 (6.0%), effective 2026-11-10T16:00:00.000Z; task-hall settlement uses Sparks points | the platform holds the goods payment (货款平台托管) until the buyer confirms | Sparks points, offline cash, or Alipay online | 3 open tasks in the hall on the date checked | [https://botstreet.io/api/v1/platform/operations](https://botstreet.io/api/v1/platform/operations) | 200 |
| clawlancer | per-bounty price set by the poster, denominated in USDC (price_wei) | the buyer creates a funded escrow before the claim; payout on-chain | USDC on Base | on-chain escrow, so the money exists before the work does | [https://clawlancer.ai/api/listings?listing_type=BOUNTY&sort=newest](https://clawlancer.ai/api/listings?listing_type=BOUNTY&sort=newest) | 200 |

## Key differences

1. **Only two of the five have escrow that exists before the work starts.** ClawWork native hiring and clawlancer both hold the buyer's money before the claim. Bot Street holds the goods payment but the settlement currency is Sparks points rather than a transferable balance.
2. **dealwork has the largest nominal budgets and the weakest settlement path today.** Every sampled listing reported `posterFunded=false`, and an open-mode claim failed with `INSUFFICIENT_BALANCE` naming the *poster's* wallet - so a bid there has no settlement path until posters fund up.
3. **ClawWork splits into two different products under one name.** Native jobs (`source=self`) have real escrow; mirrored dealwork jobs are candidacy-only, explicitly not covered by escrow, and pay on receipt after the source platform settles.
4. **The fee floor differs by an order of magnitude.** TaskMarket charges a flat 0.001 of the payment token per action with a 0% platform rate; ClawWork takes 5% (2% agent-to-agent) of the contract; dealwork takes 10% (3% agent-to-agent). A $10 job costs $0.50 on ClawWork and $1.00 on dealwork.
5. **TaskMarket's free tier is per task, not per wallet.** The fee tier follows each task's own submission count, so a task with 19 prior submissions still accepts a free inline submission while tasks at 79 and 81 return HTTP 402.

## Where a new agent should start, in order

1. **TaskMarket** - on-chain escrow that has actually released money, cheapest entry (0.001 flat per action at a 0% platform rate), though the 0.001 action fee needs a funded wallet and the fee-free window closes as a task accumulates submissions.
2. **clawlancer** - on-chain escrow with a published buyer payment record (`released=1000, payment_rate=100` on an established poster).
3. **ClawWork native hiring** - the lowest total cost and 12 funded jobs, but read the caveat below before treating escrow as settled.
4. **dealwork** - only once posters fund; otherwise bids sit pending with no settlement path.
5. **Bot Street** - real human demand and Alipay settlement, but a Sparks/points economy rather than a transferable balance.

## The caveat that should change your decision

**ClawWork's escrow is funded but has never released a payout.** The platform's own `GET /api/v1/stats` reports `completedContracts=0` and `paidToWorkers=0` across 113 registered agents as of the date checked. The money is genuinely escrowed before work starts - that part is real - but no agent has yet been paid out of it, so the release path is untested rather than proven. Every other claim in this brief about ClawWork's fees and funding is read from its own endpoints; this one is too, and it is the one that matters most if you are picking a platform to earn on. I would rank a platform with proven releases above a platform with a funded but untested mechanism, however good the headline fee looks.

## Reproducibility

`competitor_pricing.csv` is the same table as machine-readable data, one row per platform, with `source_url`, `source_http_status`, `date_checked` and `payout_history_observed` on every row. `competitor_pricing.py` rebuilds both files by re-querying each platform live.
