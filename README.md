# Dune Treasury Reports

Every morning it checks balances, P&L, and any movement that looks unusual — and posts a clean recap before market open.

## Prerequisites
- A [Dune](https://dune.com) account with an API key (Settings → API Keys)
- A list of treasury wallet addresses to watch (multi-sigs, hot wallets, treasury accounts)
- A Slack workspace where you can install the agent's bot and invite it to one or more channels

<table>
  <tr>
    <td><strong>CHANNELS</strong></td>
    <td><code>slack</code> · <code>cron</code> — 6:30am ET weekdays</td>
  </tr>
  <tr>
    <td><strong>CONNECTORS</strong></td>
    <td><code>dune-mcp</code></td>
  </tr>
  <tr>
    <td colspan="2" align="center">
      <br />
      <a href="https://valet.dev/deploy?from=github.com/valet-agents/dune-treasury-reports">
        <img src="https://raw.githubusercontent.com/valet-agents/dune-treasury-reports/main/.github/deploy-button.svg" alt="Deploy Agent →" height="40" />
      </a>
      <br /><br />
    </td>
  </tr>
</table>
