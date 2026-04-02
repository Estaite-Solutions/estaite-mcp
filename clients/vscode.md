# VS Code

**Step 1** — Create or edit `.vscode/mcp.json` in your project root.

**Step 2** — Add the server:

```json
{
  "servers": {
    "estaite": {
      "type": "http",
      "url": "https://mcp.estaite.com",
      "headers": {
        "x-api-key": "YOUR_API_KEY"
      }
    }
  }
}
```

**Step 3** — Reload the window. The Estaite tools will be available via GitHub Copilot Chat.