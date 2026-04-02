# LangChain

**Step 1** — Install dependencies:

```sh
pip install langchain-mcp-adapters langchain-openai langgraph python-dotenv
```

**Step 2** — Set up environment variables. Create a `.env` file:

```
OPENAI_API_KEY=your_openai_api_key_here
```

**Step 3** — Build your agent:

```python
import asyncio
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import create_react_agent
from langchain_mcp_adapters.client import MultiServerMCPClient
from dotenv import load_dotenv

load_dotenv()

async def main():
    async with MultiServerMCPClient({
        "estaite": {
            "url": "https://mcp.estaite.com?key=YOUR_API_KEY",
            "transport": "streamable_http",
        }
    }) as client:
        tools = await client.get_tools()
        print("Available tools:", [tool.name for tool in tools])

        llm = ChatOpenAI(model="gpt-4o-mini")

        system_prompt = """
        You are a US rental market analyst powered by Estaite data. Your tools include:
        - search_estaite_submarkets: find a submarket by name
        - get_estaite_market_snapshot: quick rent, vacancy, and affordability summary
        - get_estaite_rent_trends: historical rent trends over time
        - compare_estaite_submarkets: side-by-side comparison of multiple submarkets
        - find_estaite_submarkets_by_criteria: filter by rent range, growth, vacancy, state
        - rank_estaite_submarkets: rank submarkets within a metro or state by a metric

        Always search for the submarket first to get its ID, then use that ID with other tools.
        """

        agent = create_react_agent(model=llm, tools=tools, prompt=system_prompt)

        result = await agent.ainvoke({
            "messages": [("human", "What are rent trends in Austin, TX over the last 12 months?")]
        })

        print(result["messages"][-1].content)

if __name__ == "__main__":
    asyncio.run(main())
```