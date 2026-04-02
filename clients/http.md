# Custom Code (HTTP)

All tools are available via standard JSON-RPC 2.0 over HTTP.

**Endpoint:**

```
POST https://mcp.estaite.com?key=YOUR_API_KEY
Content-Type: application/json
```

**Discover available tools:**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list"
}
```

**Call a tool:**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "search_estaite_submarkets",
    "arguments": {
      "query": "Austin"
    }
  }
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Found 3 submarkets matching 'Austin'..."
      }
    ]
  }
}
```

**Error response:**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "error": {
    "code": -32602,
    "message": "Invalid params",
    "data": "query is required"
  }
}
```

Replace `search_estaite_submarkets` with any tool name. All tools follow the same JSON-RPC pattern.