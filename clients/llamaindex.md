# LlamaIndex

**Step 1** — Install the MCP toolkit:

```sh
pip install llama-index-tools-mcp
```

**Step 2** — Connect and call tools:

```python
import asyncio
from llama_index.tools.mcp import BasicMCPClient

async def main():
    client = BasicMCPClient("https://mcp.estaite.com?key=YOUR_API_KEY")

    # List available tools
    tools = await client.list_tools()
    print("Tools:", [t.name for t in tools])

    # Call a tool
    result = await client.call_tool(
        "search_estaite_submarkets",
        {"query": "Austin"}
    )
    print(result)

asyncio.run(main())
```