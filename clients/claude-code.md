# Claude Code

**Step 1** — Add the MCP server. Run this command in your terminal:

```sh
claude mcp add --transport http estaite "https://mcp.estaite.com?key=YOUR_API_KEY"
```

**Step 2** — Verify the connection:

```sh
claude mcp list
```

You should see `estaite` listed as connected.

**Step 3** — Start using it. Ask Claude about any US rental market and it will use the Estaite tools automatically.