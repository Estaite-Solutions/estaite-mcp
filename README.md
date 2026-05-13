# Estaite MCP Server

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![MCP](https://img.shields.io/badge/MCP-2025--03--26-blue)](https://modelcontextprotocol.io)
[![Free Tier](https://img.shields.io/badge/Free%20Tier-1%2C000%20calls%2Fmo-green)](https://estaite.com/developers)
[![smithery badge](https://smithery.ai/badge/developer-ziup/estaite-mcp)](https://smithery.ai/servers/developer-ziup/estaite-mcp)
[![MCP Badge](https://lobehub.com/badge/mcp/estaite-solutions-estaite-mcp)](https://lobehub.com/mcp/estaite-solutions-estaite-mcp)


**Add current US rental market intelligence to any AI agent in under 60 seconds.** 1,500+ submarkets, monthly-refreshed rent / vacancy / affordability / trends, 1,000 free calls per month. No credit card required.

---

## What you can build

- **Rental market research agents** — Ask Claude *"compare rents in Austin vs Nashville for a 2-bedroom"* and get current, cited data without any data pipeline of your own.
- **Investment underwriting bots** — Let your AI evaluate cap rates, rent growth, and vacancy trends across hundreds of submarkets in a single conversation.
- **PropTech AI features** — Drop into your existing app — tenant portals, brokerage tools, CRM enrichment — and your agent suddenly knows real estate.

## 30-second install (Claude Desktop)

Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "estaite": {
      "command": "npx",
      "args": ["-y", "mcp-remote@latest", "https://mcp.estaite.com", "--header", "x-api-key:YOUR_API_KEY"]
    }
  }
}
```

Restart Claude Desktop. Done. Get your free API key at [estaite.com/developers](https://estaite.com/developers).

> Setting up a different client? See [`clients/`](./clients) — copy-paste configs for 13 popular MCP clients.

## Example interaction

> **You:** *What's the rental market like in Austin right now?*
>
> *Claude finds the right submarket via `search_estaite_submarkets`, then calls `get_estaite_market_snapshot`.*
>
> **Claude:** *Austin is currently a renter's market. Vacancy is running above the balanced threshold, median 2BR rent is down year-over-year, and properties are sitting on market longer than the national average. New construction outpaced demand over the past 12 months. Source: estaite.com.*

Every response includes structured data plus a citation back to the source. No hallucinated rent numbers.

## Pricing

| Plan | Monthly Calls | Rate Limit | Price | Best for |
|------|---------------|------------|-------|----------|
| **Free** | 1,000 | 2 req/sec | $0 | Hobby projects, testing, demos |
| **Starter** | 10,000 | 10 req/sec | $49 / mo | MVPs, small production apps |
| **Pro** | 100,000 | 30 req/sec | $149 / mo | Production AI agents, internal tools |
| **Enterprise** | Custom | Custom | [Contact us](mailto:success@estaite.com?subject=Enterprise%20MCP%20Plan) | Teams, high-volume agents, SSO |

Free tier requires no credit card. Upgrade in-app any time. Full pricing details at [estaite.com/developers](https://estaite.com/developers).

## All 13 tools

### Discovery
- **`search_estaite_submarkets`** — Find submarkets by name. Returns IDs to feed into metric tools.
- **`list_estaite_submarkets`** — Browse submarkets; filter by state or metro. Returns up to 200 results.
- **`find_estaite_submarkets_by_criteria`** — Filter by rent range, YoY growth, vacancy ceiling, state, or CBSA.

### Single-submarket analysis
- **`query_estaite_submarket_index`** — Full metrics: rent, vacancy, affordability, days on market, trend history.
- **`get_estaite_market_snapshot`** — Quick summary: condition label, vacancy, rent direction.
- **`get_estaite_rent_trends`** — Historical rent change: MoM, 3M, 6M, 9M, YoY.
- **`get_estaite_affordability`** — Rent-to-income ratio + affordability label (e.g. *Cost Burdened*).

### Comparison & ranking
- **`compare_estaite_submarkets`** — Side-by-side comparison across all key metrics. Up to 10 submarkets per call.
- **`rank_estaite_submarkets`** — Rank by rent, growth, vacancy, affordability, or days on market.
- **`get_estaite_comparable_markets`** — Find submarkets with similar rent levels to a given one.

### Geographic
- **`get_estaite_zip_metrics`** — Rental metrics for a specific ZIP code.
- **`get_estaite_cbsa_overview`** — Metro-level averages and full submarket list.
- **`get_estaite_cbsa_trends`** — Monthly aggregated trends for an entire metro.

Detailed input/output schemas in [`llms.txt`](./llms.txt) (also designed to be readable by AI agents auto-discovering the server).

## Client integrations

Copy-paste config for 13 MCP clients. Pick yours:

| | | |
|---|---|---|
| [Claude Desktop](./clients/claude-desktop.md) | [Claude Code](./clients/claude-code.md) | [Cursor](./clients/cursor.md) |
| [VS Code](./clients/vscode.md) | [ChatGPT](./clients/chatgpt.md) | [Copilot Studio](./clients/copilot-studio.md) |
| [LangChain](./clients/langchain.md) | [LlamaIndex](./clients/llamaindex.md) | [CrewAI](./clients/crewai.md) |
| [OpenAI SDK](./clients/openai-sdk.md) | [OpenAI Agent Builder](./clients/openai-agent-builder.md) | [n8n](./clients/n8n.md) |
| [Direct HTTP](./clients/http.md) | | |

## Data coverage

- **Geography:** 1,500+ submarkets across the largest US Metropolitan Statistical Areas — expanding monthly
- **Property types:** `apt` (apartment), `sfr` (single-family rental), `ct` (condo/townhome)
- **Bedroom counts:** 1–5 BR per property type
- **Update frequency:** Monthly
- **Citations:** Every tool response includes source attribution

## Server details

- **MCP server URL:** `https://mcp.estaite.com`
- **Protocol:** Streamable HTTP (MCP 2025-03-26)
- **AI agent manifest:** [`llms.txt`](./llms.txt) — discoverable, machine-readable

## Authentication

Pass your API key with one of:

| Method | Example | Notes |
|--------|---------|-------|
| **Header (recommended)** | `x-api-key: YOUR_API_KEY` | Preferred for code-driven clients (curl, custom agents, MCP SDKs). |
| Bearer token | `Authorization: Bearer YOUR_API_KEY` | Standard Bearer scheme. Used by clients that complete the OAuth 2.1 flow. |
| Query param | `https://mcp.estaite.com?key=YOUR_API_KEY` | Convenient for clients that only accept a URL (Claude Desktop Connectors, ChatGPT custom GPTs). |

### API key security

Treat your API key like a password:
- Never commit it to git, and don't paste it into URLs you'll share publicly
- Rotate it immediately at [estaite.com/developers](https://estaite.com/developers) if exposed
- Use environment variables, not hardcoded strings, when integrating

## Get an API key

Free tier, no credit card, at **[estaite.com/developers](https://estaite.com/developers)**.

## License

The configs, examples, and docs in this repository are MIT licensed (see [LICENSE](./LICENSE)). The Estaite MCP server itself is a hosted commercial service; see [pricing](https://estaite.com/developers) for plan details.
