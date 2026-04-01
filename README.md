# Estaite MCP Server

Current US rental market data for AI agents and LLMs via the Model Context Protocol (MCP).
The Estaite Submarket Index covers 1500+ submarkets across the largest MSAs in the United States and expanding, with monthly-updated rent, vacancy, affordability, and trend data.

## Getting Started

- **MCP Server URL:** `https://mcp.estaite.com`
- **Protocol:** Streamable HTTP (MCP 2025-03-26)

**Authentication** — pass your API key using any of these methods:

| Method | Example |
|--------|---------|
| Header | `x-api-key: YOUR_API_KEY` |
| Bearer token | `Authorization: Bearer YOUR_API_KEY` |
| Query param | `https://mcp.estaite.com?key=YOUR_API_KEY` |

Get your API key at [estaite.com/developers](https://estaite.com/developers).

## Available Tools

- **Search Submarkets** — Search for submarkets by name. Use this to get a submarket ID before calling metrics tools.
- **List Submarkets** — List all submarkets with optional filters by state or CBSA.
- **ZIP Metrics** — Rental metrics for a specific ZIP code — rent, YoY growth, vacancy, listings, and household income.
- **Submarket Metrics** — Full metrics for a submarket: rent, vacancy, affordability, days on market, saturation, and trend history.
- **Compare Submarkets** — Side-by-side comparison of two or more submarkets across all key metrics.
- **Market Snapshot** — Quick summary of a submarket including market condition label, vacancy, and rent trends.
- **Rent Trends** — Historical rent trend data — MoM, 3M, 6M, 9M, and YoY changes over configurable months.
- **Affordability** — Rent-to-income ratio and affordability index with trend changes over 3, 6, 9, and 12 months.
- **Find by Criteria** — Filter submarkets by rent range, YoY growth minimum, vacancy cap, state, or CBSA.
- **Metro Overview** — Aggregated rent, growth, and vacancy averages for all submarkets within a metro area.
- **Rank Submarkets** — Rank submarkets within a metro or state by median rent, rent growth, vacancy, affordability, or days on market.
- **Comparable Submarkets** — Find submarkets with similar rent levels to a given submarket.
- **Metro Historical Trends** — Monthly aggregated rent and vacancy trends for an entire metro area over up to 12 months.

## Data Coverage

- **Property types:** `apt` (apartment), `sfr` (single-family), `ct` (condo/townhome)
- **Bedroom counts:** 1–5 bedrooms per property type
- **Geography:** 1500+ submarkets across the largest MSAs in the United States and expanding
- **Frequency:** Monthly updates

## Quick Connect

### Claude Desktop
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

### Claude Code
```bash
claude mcp add --transport http estaite "https://mcp.estaite.com" --header "x-api-key:YOUR_API_KEY"
```

### OpenAI SDK
```python
resp = client.responses.create(
    model="gpt-4o",
    tools=[{
        "type": "mcp",
        "server_label": "estaite",
        "server_url": "https://mcp.estaite.com",
        "headers": {"x-api-key": "YOUR_API_KEY"},
        "require_approval": "never",
    }],
    input="What is the average rent in Austin, TX?",
)
```
