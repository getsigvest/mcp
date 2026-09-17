# SigVest MCP

[![MCP registry](https://img.shields.io/badge/MCP%20registry-com.getsigvest%2Fsigvest-6f42c1)](https://registry.modelcontextprotocol.io/v0/servers?search=com.getsigvest/sigvest)
[![Endpoint](https://img.shields.io/badge/endpoint-mcp.getsigvest.com%2Fmcp-0b7285)](https://www.getsigvest.com/mcp)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)

Connect any MCP client to [SigVest](https://www.getsigvest.com) — 14 read-only
portfolio tools for drift, risk, benchmarks, earnings, news impact and tax-loss
harvesting.

- **Endpoint:** `https://mcp.getsigvest.com/mcp` (Streamable HTTP)
- **Auth:** bearer API key, created at [getsigvest.com/mcp](https://www.getsigvest.com/mcp)
- **Official MCP registry:** [`com.getsigvest/sigvest`](https://registry.modelcontextprotocol.io/v0/servers?search=com.getsigvest/sigvest)
- **Free tier:** 50 credits/month, no credit card. Current limits are shown on
  your [dashboard](https://www.getsigvest.com/mcp).

This repo is **config and docs only**. The server itself is hosted; there is
nothing to install or run.

## Connect

### 1. Get a key

Sign in at [getsigvest.com/mcp](https://www.getsigvest.com/mcp) → **Create key**.
The key is shown once. Export it:

```bash
export SIGVEST_API_KEY="sk_..."
```

### 2. Add the server

**Cursor** — `.cursor/mcp.json` in your project, or `~/.cursor/mcp.json` globally:

```json
{
  "mcpServers": {
    "sigvest": {
      "url": "https://mcp.getsigvest.com/mcp",
      "headers": { "Authorization": "Bearer ${env:SIGVEST_API_KEY}" }
    }
  }
}
```

**Claude Code** — `.mcp.json` at your project root:

```json
{
  "mcpServers": {
    "sigvest": {
      "type": "http",
      "url": "https://mcp.getsigvest.com/mcp",
      "headers": { "Authorization": "Bearer ${SIGVEST_API_KEY}" }
    }
  }
}
```

Or in one command:

```bash
claude mcp add --transport http sigvest https://mcp.getsigvest.com/mcp \
  --header "Authorization: Bearer ${SIGVEST_API_KEY}"
```

**Claude Desktop** — remote servers are added as *custom connectors*
(Settings → Connectors), which authenticate over OAuth. SigVest uses a static
bearer key, so bridge it as a local server with
[`mcp-remote`](https://github.com/geelen/mcp-remote) in
`claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "sigvest": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote",
        "https://mcp.getsigvest.com/mcp",
        "--header", "Authorization:${SIGVEST_AUTH}"
      ],
      "env": { "SIGVEST_AUTH": "Bearer YOUR_API_KEY" }
    }
  }
}
```

> `--header` takes `Name:value` with no space around the colon; put the space
> inside the env var, as above.

**Windsurf and other clients** — same shape as the Cursor config: a
`streamable-http` URL plus an `Authorization: Bearer <key>` header.

### 3. Ask something

> "Analyze my portfolio: 50 AAPL, 30 MSFT, 20 NVDA."
> "What news moved my holdings today?"

Holdings are passed in with the tool call — SigVest's MCP server does not read a
linked brokerage account. Brokerage connection lives in the SigVest web app.

## Tools

| Tool | Credits | What it does |
| --- | --- | --- |
| `analyze_portfolio` | 10 | Portfolio analysis with AI commentary |
| `check_drift` | 3 | Allocation drift vs targets |
| `generate_rebalance_plan` | 8 | Suggested rebalance trades (suggestions only) |
| `run_scenario` | 12 | What-if scenario analysis |
| `compare_benchmarks` | 15 | vs SPY / QQQ / AGG with attribution |
| `screen_positions` | 5 | Health signals per holding |
| `get_sector_exposure` | 3 | Sector and factor breakdown |
| `get_risk_metrics` | 5 | Sharpe, VaR, beta, drawdown |
| `get_news_impact` | 5 | News filtered to your holdings |
| `get_morning_brief` | 20 | Daily portfolio brief |
| `get_earnings_calendar` | 5 | Upcoming earnings and implied moves |
| `get_sentiment` | 3 | News sentiment per ticker |
| `get_implied_moves` | 8 | Options-implied earnings moves |
| `scan_tax_harvest` | 15 | Tax-loss harvesting opportunities |

Per-tool credit costs and rate limits are documented at
[getsigvest.com/mcp](https://www.getsigvest.com/mcp).

## Security

- **Read-only.** No tool places, cancels or routes an order. `generate_rebalance_plan`
  returns suggested trades as text; executing them is something you do yourself,
  in your own brokerage.
- **No brokerage credentials.** The MCP server never receives broker logins or
  account access. Holdings come in as tool arguments.
- **You control the key.** Keys are minted and revoked from your SigVest
  dashboard, sent as an `Authorization` header, and scoped to your account.
  Keep them in an environment variable — never commit one.
- **Transport.** HTTPS only, Streamable HTTP, with host and origin checks
  against DNS rebinding.
- **Not financial advice.** Output is AI-generated analysis, including
  third-party headlines and model-written prose. It is not advice from a
  licensed adviser, and it can be wrong.

## Files in this repo

| File | For |
| --- | --- |
| `.mcp.json` | Claude Code project config |
| `.cursor/mcp.json` | Cursor project config |
| `mcp.json` + `plugin.json` | [Agent Plugins 1.0.0](https://agent-plugins.org) package (Cursor, and other clients that load the portable format) |
| `server.json` | The manifest published to the official MCP registry, mirrored from `https://www.getsigvest.com/.well-known/mcp/server.json` |

The Agent Plugins `mcp.json` declares the endpoint only: the spec keeps
credentials out of package data, so add your key in your client after
installing.

## Links

- Product: <https://www.getsigvest.com>
- MCP docs and keys: <https://www.getsigvest.com/mcp>
- Manifest: <https://www.getsigvest.com/.well-known/mcp/server.json>
- Health: <https://mcp.getsigvest.com/health>

## License

[MIT](LICENSE) — covers the configuration and documentation in this repository.
The SigVest service and its server implementation are proprietary and are not
included here.
