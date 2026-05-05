This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## Setup

### Connectors

- **dune-mcp**: The Dune MCP server. The agent uses it to query wallet balances, 24h P&L, and transaction history for every address in the `WALLETS` env var, and to answer free-form on-chain questions in Slack via SQL. Read-only — the agent never signs or sends a transaction. Add it from the catalog at the org level so other Dune-powered agents can share it.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts the morning treasury recap to whichever channels the bot has been invited to. Slack writes use the auto-injected outbound Slack connector.
- **cron** (cron): Fires the morning treasury recap at 6:30am Eastern, Monday through Friday (`30 6 * * 1-5`, `America/New_York`) — before US market open. Declared inline in `valet.yaml`, so it's created automatically by the dashboard setup flow.

### Secrets

- **DUNE_API_KEY** (org or agent level): Required. Get one at [dune.com](https://dune.com) → Settings → API Keys → "Create new API key". Scope is whatever your Dune plan permits — the agent only runs SELECT queries against on-chain datasets and the wallets you list in `WALLETS`. The free tier is enough for the daily recap; heavy ad-hoc Q&A may push you onto a paid plan with more SQL credits.

### External Setup

1. Set the `DUNE_API_KEY` secret on the agent (or share an org-level secret if you already have one).
2. Set the `WALLETS` env var on the agent: a comma-separated list of `chain:address` pairs, e.g. `eth:0xabc...,polygon:0xdef...,arbitrum:0x123...`. These are the treasury wallets the morning recap queries. If `WALLETS` is unset, the cron run DMs the workspace install user with a config hint instead of posting an empty recap.
3. After deploy, install the agent's Slack bot in your workspace and invite it to whichever channel(s) you want the morning recap in. The agent posts the recap to every channel it's a member of — invite it to one focused channel (e.g. an internal finance/treasury room) or several. If the bot has not been invited anywhere, the recap is sent as a DM to the workspace install user with a one-line nudge to invite it somewhere.
4. Invite the bot to any additional channels where teammates should be able to @mention it for ad-hoc on-chain questions (e.g. a finance ops channel for "what moved on the multi-sig today?" follow-ups).
5. The first cron fire is the next 6:30am Eastern weekday after deploy. To smoke-test sooner, @mention the bot in Slack with a question like *"what moved on the multi-sig today?"* — that exercises the Slack + Dune path without waiting for the cron.

## Customizing

- **Change the schedule**: edit the `cron` and `timezone` on the `cron` channel in `valet.yaml`, then redeploy. The default `30 6 * * 1-5` `America/New_York` is intentional — 6:30am ET is one hour before US equity market open, so the recap lands while traders are at their desks but before the bell.
- **Change which wallets are watched**: update the `WALLETS` env var on the agent. The format is comma-separated `chain:address` pairs, e.g. `eth:0xabc...,polygon:0xdef...`. Optionally label a wallet by prefixing it with `label=` inside the pair (e.g. `multi-sig=eth:0xabc...`) — the SOUL workflow uses the label as the wallet's name in the recap.
- **Tune the unusual-tx thresholds**: defaults in SOUL.md are "single tx > $50,000 USD", "single tx > 10% of wallet USD value", and "outflow to a never-before-seen counterparty". To override, set `UNUSUAL_TX_USD` (e.g. `100000`) and/or `UNUSUAL_TX_PCT` (e.g. `0.05` for 5%) on the agent. The "unknown counterparty" rule has no knob — it's always on, since first-time outflow recipients are the highest-signal anomaly for a treasury wallet.
- **Cap the unusual list size**: the SOUL caps the unusual-tx list at 5 per wallet in the Slack post and links to the full Dune query for the rest. If you want a different cap, override in `SOUL.md` — there is no env var for it.
