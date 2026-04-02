# n8n

**Step 1** — In your n8n workflow, create an **AI Agent** node. Attach a **Chat Model** (e.g. GPT-4o) and an **MCP Client** tool node.

**Step 2** — In the MCP Client node, configure the connection:

```
Transport:    Streamable HTTP
Server URL:   https://mcp.estaite.com?key=YOUR_API_KEY
```

**Step 3** — Select the Estaite tools you want the agent to use, then map your workflow inputs and run.