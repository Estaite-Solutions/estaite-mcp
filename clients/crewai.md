# CrewAI

**Step 1** — Install CrewAI MCP tools:

```sh
pip install crewai crewai-tools
```

**Step 2** — Configure your agent:

```python
from crewai import Agent, Task, Crew
from crewai_tools import MCPServerAdapter

server_params = {
    "url": "https://mcp.estaite.com?key=YOUR_API_KEY",
    "transport": "streamable_http",
}

with MCPServerAdapter(server_params) as mcp_tools:
    agent = Agent(
        role="Real Estate Analyst",
        goal="Analyze rental market data using Estaite tools",
        backstory="Expert in US rental market analysis.",
        tools=mcp_tools,
        verbose=True,
        llm="gpt-4o-mini",
    )

    task = Task(
        description="What is the median rent and YoY growth in San Diego, CA?",
        expected_output="A summary of rental market conditions in San Diego.",
        agent=agent
    )

    crew = Crew(agents=[agent], tasks=[task], verbose=True)
    result = crew.kickoff()
    print(result)
```