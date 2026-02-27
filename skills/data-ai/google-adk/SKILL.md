---
name: google-adk
description: Build AI agents using Google's Agent Development Kit (ADK) for Python. Use this skill when the user wants to create ADK agents, multi-agent systems, agents with tools, workflow agents (Sequential, Parallel, Loop), or deploy agents to Google Cloud. Triggers include mentions of "ADK", "Agent Development Kit", "google-adk", "adk agent", "multi-agent system", or requests to build AI agents with Google's framework.
---

# Google ADK Agent Development

Build AI agents using Google's Agent Development Kit (ADK) - a flexible, modular Python framework for developing, evaluating, and deploying AI agents.

## Project Structure

Standard ADK project layout:

```
my_agent/
├── agent.py          # Agent definition (must export root_agent)
├── tools.py          # Custom tool functions (optional)
├── __init__.py       # Package init
└── .env              # API keys (GOOGLE_API_KEY or GOOGLE_CLOUD_PROJECT)
```

For multi-agent projects:

```
my_project/
├── agents/
│   ├── coordinator.py
│   ├── specialist_a.py
│   └── specialist_b.py
├── tools/
│   └── custom_tools.py
├── agent.py          # Root agent entry point
└── .env
```

## Core Patterns

### Single Agent with Tools

```python
from google.adk.agents import Agent

def get_weather(city: str) -> dict:
    """Get weather for a city. The docstring is critical - ADK uses it for tool schema."""
    return {"status": "success", "temp": "72F", "city": city}

root_agent = Agent(
    name="weather_agent",
    model="gemini-3.0-flash-preview",
    description="Provides weather information.",
    instruction="You help users get weather. Use the get_weather tool when asked.",
    tools=[get_weather],
)
```

### Multi-Agent with Delegation

```python
from google.adk.agents import LlmAgent

billing = LlmAgent(name="billing", model="gemini-3.0-flash-preview",
                   description="Handles billing and payment questions.")
support = LlmAgent(name="support", model="gemini-3.0-flash-preview",
                   description="Handles technical support.")

root_agent = LlmAgent(
    name="coordinator",
    model="gemini-3.0-flash-preview",
    instruction="Route billing questions to billing agent, technical issues to support.",
    sub_agents=[billing, support],
)
```

### Sequential Pipeline

```python
from google.adk.agents import SequentialAgent, LlmAgent

step1 = LlmAgent(name="researcher", instruction="Research the topic.", output_key="research")
step2 = LlmAgent(name="writer", instruction="Write based on {research}.", output_key="draft")
step3 = LlmAgent(name="editor", instruction="Polish the {draft}.")

root_agent = SequentialAgent(name="content_pipeline", sub_agents=[step1, step2, step3])
```

### Parallel Execution

```python
from google.adk.agents import ParallelAgent, SequentialAgent, LlmAgent

fetch_news = LlmAgent(name="news", output_key="news_data")
fetch_weather = LlmAgent(name="weather", output_key="weather_data")

gatherer = ParallelAgent(name="info_gather", sub_agents=[fetch_news, fetch_weather])
synthesizer = LlmAgent(name="synth", instruction="Combine {news_data} and {weather_data}.")

root_agent = SequentialAgent(name="pipeline", sub_agents=[gatherer, synthesizer])
```

### Loop with Termination

```python
from google.adk.agents import LoopAgent, LlmAgent, BaseAgent
from google.adk.events import Event, EventActions

class QualityChecker(BaseAgent):
    async def _run_async_impl(self, ctx):
        status = ctx.session.state.get("quality", "fail")
        yield Event(author=self.name, actions=EventActions(escalate=(status == "pass")))

refiner = LlmAgent(name="refiner", instruction="Improve {draft}.", output_key="draft")
checker = LlmAgent(name="checker", instruction="Rate quality.", output_key="quality")

root_agent = LoopAgent(
    name="refinement_loop",
    max_iterations=5,
    sub_agents=[refiner, checker, QualityChecker(name="gate")],
)
```

## Key Concepts

### Agent Parameters

| Parameter | Required | Purpose |
|-----------|----------|---------|
| `name` | Yes | Unique identifier (avoid "user") |
| `model` | Yes | LLM model string (e.g., "gemini-3.0-flash-preview") |
| `instruction` | No | System prompt guiding behavior |
| `description` | No | Used by parent agents for delegation decisions |
| `tools` | No | List of functions or Tool instances |
| `sub_agents` | No | Child agents for multi-agent systems |
| `output_key` | No | Auto-save response to session state |

### State Management

Use `{var}` in instructions to read from state. Use `output_key` to write to state.

```python
agent_a = LlmAgent(name="a", output_key="result_a")  # Writes to state["result_a"]
agent_b = LlmAgent(name="b", instruction="Process {result_a}.")  # Reads state["result_a"]
```

### Tool Design

Tool functions must have:
- Clear docstrings (ADK parses these for the LLM)
- Type hints for parameters
- Return dict or simple types

```python
def search_database(query: str, limit: int = 10) -> dict:
    """Search the database for matching records.

    Args:
        query: The search query string.
        limit: Maximum results to return (default 10).

    Returns:
        dict with 'status' and 'results' keys.
    """
    return {"status": "success", "results": [...]}
```

## Running Agents

### Via CLI (Development)

```bash
# Install
pip install google-adk

# Set API key
export GOOGLE_API_KEY="your-key"

# Run CLI
adk run my_agent

# Run web UI (development only)
adk web --port 8000

# Run API server
adk api_server my_agent --port 8080
```

### Programmatically (Production)

For APIs, UIs, or custom integrations, use Runner and Session directly:

```python
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.genai import types

# Setup
agent = Agent(name="assistant", model="gemini-3.0-flash-preview", instruction="Be helpful.")
session_service = InMemorySessionService()
runner = Runner(agent=agent, app_name="my_app", session_service=session_service)

# Create session
await session_service.create_session(
    app_name="my_app", user_id="user_123", session_id="session_456"
)

# Run agent
message = types.Content(role="user", parts=[types.Part(text="Hello!")])
async for event in runner.run_async(
    user_id="user_123", session_id="session_456", new_message=message
):
    if event.is_final_response():
        print(event.content.parts[0].text)
```

See **[runtime.md](examples/runtime.md)** for complete API/UI integration examples.

## examples

For detailed patterns and examples:
- **[agents.md](examples/agents.md)**: LlmAgent, workflow agents, custom agents
- **[tools.md](examples/tools.md)**: Function tools, MCP tools, OpenAPI tools, AgentTool
- **[patterns.md](examples/patterns.md)**: Multi-agent design patterns
- **[runtime.md](examples/runtime.md)**: Runner, Session, Events, and building APIs/UIs