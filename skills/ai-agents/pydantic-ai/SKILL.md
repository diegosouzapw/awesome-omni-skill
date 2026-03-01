---
name: pydantic-ai
description: "Python framework for building production-grade AI agents with LLMs. Use when creating agents that need structured outputs, tools, dependency injection, or type-safe interactions. Specifically use for: (1) Building AI agents with OpenAI, Anthropic, Google, or other LLM providers, (2) Creating agents that require structured output validation via Pydantic models, (3) Implementing tool-calling agents with function tools, (4) Building multi-agent applications or A2A (Agent2Agent) protocol servers, (5) Adding observability with Pydantic Logfire, (6) Streaming responses or events from agents"
---

# Pydantic AI

## Overview

Pydantic AI is a type-safe Python framework for building AI agents. It provides tools, structured outputs, dependency injection, and comprehensive model support for production-grade applications.

## When to Use Pydantic AI

Use this skill when you need to:

- Build AI agents with any LLM provider (OpenAI, Anthropic, Google, Groq, etc.)
- Ensure type-safe, validated structured outputs using Pydantic models
- Create agents that can call tools (functions) to gather information
- Implement dependency injection for testable, maintainable agents
- Stream agent responses or events in real-time
- Build multi-agent workflows or A2A servers
- Add observability with Pydantic Logfire

## Quick Start

### Installation

```bash
uv add pydantic-ai
```

Or for slim installs with only specific model dependencies:

```bash
uv add "pydantic-ai-slim[openai,anthropic]"
```

### Basic Agent

```python
from pydantic_ai import Agent

agent = Agent('openai:gpt-4o', instructions='Be helpful and concise.')

result = agent.run_sync('What is 2+2?')
print(result.output)
```

### Agent with Tools and Structured Output

```python
from dataclasses import dataclass
from pydantic import BaseModel, Field
from pydantic_ai import Agent, RunContext

@dataclass
class Dependencies:
    api_key: str

class Output(BaseModel):
    response: str
    confidence: float

agent = Agent(
    'openai:gpt-4o',
    deps_type=Dependencies,
    output_type=Output,
    instructions='Help users with their queries.',
)

@agent.tool
async def get_info(ctx: RunContext[Dependencies], query: str) -> str:
    """Fetch information about a topic."""
    return f"Information about {query}"

result = await agent.run('Tell me about Python', deps=Dependencies(api_key='key'))
print(result.output)  # Output(response='...', confidence=0.95)
```

## Running Agents

- `agent.run()` - Async execution
- `agent.run_sync()` - Synchronous execution
- `agent.run_stream()` - Stream text/structured output
- `agent.run_stream_events()` - Stream all events (tool calls, text, etc.)
- `agent.iter()` - Iterate over graph nodes

## Agent Components

| Component | Description |
|-----------|-------------|
| **Instructions** | Static or dynamic instructions for the LLM |
| **Tools** | Functions the LLM can call (`@agent.tool`) |
| **Output Type** | Pydantic model for structured output validation |
| **Dependencies** | Type-safe dependency injection for tools/instructions |
| **Model** | LLM model (OpenAI, Anthropic, Google, etc.) |

## Model Selection

Specify models by provider: `openai:gpt-4o`, `anthropic:claude-3-5-sonnet`, `google:gemini-2.0-flash`, etc.

See `references/models.md` for all supported providers and models.

## Common Patterns

### Dynamic Instructions

```python
@agent.instructions
async def add_context(ctx: RunContext[Dependencies]) -> str:
    return f"Current user ID: {ctx.deps.user_id}"
```

### Tool Parameters

```python
@agent.tool
async def search(
    ctx: RunContext[Dependencies],
    query: str,
    max_results: int = 10,
) -> list[str]:
    """Search a database with the given query."""
    # Implementation
    pass
```

### Streaming Responses

```python
async with agent.run_stream('Tell me a story') as response:
    async for chunk in response.stream_text():
        print(chunk, end='')
```

## Advanced Features

- **Graphs**: Complex workflows using `pydantic_graph`
- **Multi-Agent**: Agent-to-agent communication with A2A protocol
- **Durable Execution**: DBOS, Prefect, or Temporal integration
- **MCP Integration**: Model Context Protocol support
- **UI Streams**: AG-UI or Vercel AI SDK integration

## Resources

### references/
- `models.md` - All supported LLM providers and models
- `api_reference.md` - API documentation for core classes
- `examples.md` - Detailed examples for common use cases

### scripts/
No executable scripts included. Pydantic AI is a framework, not a tool collection.

### assets/
No assets included. This is a pure Python framework.

## Development

- Test agents with `agent.run_sync()` for quick iteration
- Use `uv run pytest` for testing (project must have tests configured)
- Enable Logfire for observability: `logfire.instrument_pydantic_ai()`
