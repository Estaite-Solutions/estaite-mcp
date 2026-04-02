# OpenAI SDK (Python)

```python
from openai import OpenAI

client = OpenAI()

resp = client.responses.create(
    model="gpt-4o",
    tools=[
        {
            "type": "mcp",
            "server_label": "estaite",
            "server_url": "https://mcp.estaite.com",
            "headers": {
                "x-api-key": "YOUR_API_KEY"
            },
            "require_approval": "never",
        },
    ],
    input="What is the average rent in Austin, TX?",
)

print(resp.output_text)
```