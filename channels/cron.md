# Morning Treasury Recap

The cron channel fires once on its schedule (6:30am Eastern,
Monday through Friday — one hour before US equity market open).
There is no payload to parse — your job is to run the morning
recap workflow and post the result to Slack.

## Steps

1. Read the `WALLETS` env var. It's a comma-separated list of
   `chain:address` pairs (e.g. `eth:0xabc...,polygon:0xdef...`),
   optionally with `label=` prefixes inside the pair.
2. Follow the **Morning Recap Workflow** in SOUL.md (Phases 1–3):
   for each wallet, query Dune for balances, 24h P&L, and the last
   24h of transactions; flag unusual txs; compose the recap.
3. Resolve target channels per the SOUL **Where to post** section:
   list every channel the bot is a member of and post once to
   each. If the bot is in zero channels, DM the workspace install
   user instead with the recap and a one-line invite hint.
4. Post exactly once per resolved destination. Do not retry on
   failure — log the error in your session and continue with the
   remaining destinations. The next cron fire is the recovery.
5. Do not send any follow-ups, reactions, or thread replies after
   the initial post. Your turn ends after the posts complete.

## Skip conditions

- **`WALLETS` env var is unset or empty**: skip the post entirely.
  Instead, DM the workspace install user with a single line:
  *"No wallets configured. Set the `WALLETS` env var on the agent
  to a comma-separated list of `chain:address` pairs (e.g.
  `eth:0xabc...,polygon:0xdef...`) and I'll start posting the
  morning recap."* Then stop.
- **A wallet has zero activity in the last 24h**: do NOT skip the
  whole post. Render that wallet as a single line: `<wallet
  label or short addr> · no movement` and continue with the
  remaining wallets.
- **All watched wallets had zero activity**: still post the
  one-liner-per-wallet recap. Silence is worse than a "nothing
  happened" recap — the team needs to know the agent ran.
- **Dune is unreachable or returns errors for every wallet**: log
  the error in your session and stop silently. The next cron fire
  is the recovery; a half-broken recap is worse than no post.
