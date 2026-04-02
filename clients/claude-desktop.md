# Claude Desktop

## Option A — Connectors (easiest)

**Step 1** — Go to **Settings → Connectors → Add custom connector**.

**Step 2** — Fill in the details:

```
Name:    Estaite
URL:     https://mcp.estaite.com?key=YOUR_API_KEY
```

**Step 3** — Click **Add**. The Estaite tools are immediately available in any chat.

---

## Option B — Config file

**Step 1** — Go to **Settings → Developer → Edit Config**. This opens `claude_desktop_config.json`. If it doesn't exist, create it at:

```
Windows:  %APPDATA%\Claude\claude_desktop_config.json
Mac:      ~/Library/Application Support/Claude/claude_desktop_config.json
```

**Step 2** — Add the Estaite MCP server:

```json
{
  "mcpServers": {
    "estaite": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.estaite.com?key=YOUR_API_KEY"
      ]
    }
  }
}
```

If you already have other `mcpServers` entries, add the `"estaite"` block alongside them — don't replace the whole file.

**Step 3** — Save and restart Claude Desktop. Fully quit (don't just close the window) and relaunch. The Estaite tools will appear in the tools panel.

**Step 4** — Test it. Ask Claude: *"What are rent trends in Austin, TX?"* — it will use the Estaite tools automatically.