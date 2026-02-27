---
name: agent-builder
description: Build AI agents using pai-agent-sdk with Pydantic AI. Covers agent creation via create_agent(), toolset configuration, session persistence with ResumableState, subagent hierarchies, and browser automation. Use when creating agent applications, configuring custom tools, managing multi-turn sessions, setting up hierarchical agents, or implementing HITL approval flows.
---

# Building Agents with pai-agent-sdk

Build production-ready AI agents with Pydantic AI.

## Quick Start

```python
from pai_agent_sdk.agents import create_agent

async with create_agent("openai:gpt-4o") as runtime:
    result = await runtime.agent.run("Hello", deps=runtime.ctx)
    print(result.output)
```

With tools and streaming:

```python
from pai_agent_sdk.agents import create_agent, stream_agent
from pai_agent_sdk.toolsets.core.filesystem import tools as fs_tools
from pai_agent_sdk.toolsets.core.shell import tools as shell_tools

async with create_agent(
    model="anthropic:claude-sonnet-4",
    system_prompt="You are a coding assistant.",
    tools=[*fs_tools, *shell_tools],
) as runtime:
    async with stream_agent(runtime.agent, "List files", runtime.ctx) as stream:
        async for event in stream:
            # Handle streaming events
            pass
```

## Installation

```bash
pip install pai-agent-sdk[all]
# Or selective: pip install pai-agent-sdk[docker,web,document]
```

## Core Concepts

| Concept           | Purpose                                        | Reference                                                  |
| ----------------- | ---------------------------------------------- | ---------------------------------------------------------- |
| AgentContext      | Session state, model config, tool config       | [docs/context.md](docs/context.md)                         |
| Streaming & Hooks | Real-time events, lifecycle hooks              | [docs/streaming.md](docs/streaming.md)                     |
| Events            | Lifecycle and sideband event types             | [docs/events.md](docs/events.md)                           |
| Toolset           | Tool management with hooks and HITL            | [docs/toolset.md](docs/toolset.md)                         |
| Subagent          | Hierarchical agent delegation                  | [docs/subagent.md](docs/subagent.md)                       |
| Environment       | File/shell operations, resource lifecycle      | [docs/environment.md](docs/environment.md)                 |
| Resources         | Resumable resource state export/restore        | [docs/resumable-resources.md](docs/resumable-resources.md) |
| Model             | Provider configuration, gateway mode           | [docs/model.md](docs/model.md)                             |
| Skills            | Markdown-based instruction files               | [docs/skills.md](docs/skills.md)                           |
| Message Bus       | Inter-agent communication, user steering       | [docs/message-bus.md](docs/message-bus.md)                 |
| Logging           | Configurable logging with module-level control | [docs/logging.md](docs/logging.md)                         |

## Task Guide

### Creating Custom Tools

Inherit from `BaseTool`:

```python
from pai_agent_sdk.toolsets.core.base import BaseTool
from pydantic_ai import RunContext
from pai_agent_sdk.context import AgentContext

class MyTool(BaseTool):
    name = "my_tool"
    description = "Does something useful"

    async def call(self, ctx: RunContext[AgentContext], arg: str) -> str:
        return f"Result: {arg}"
```

Full guide: [docs/toolset.md](docs/toolset.md)

### Session Persistence

Export and restore state across restarts:

```python
# Export
state = ctx.export_state()
with open("session.json", "w") as f:
    f.write(state.model_dump_json())

# Restore
from pai_agent_sdk.context import ResumableState
state = ResumableState.model_validate_json(open("session.json").read())

async with create_agent("openai:gpt-4o", state=state) as runtime:
    # Session restored with history and context
    ...
```

Full guide: [docs/context.md](docs/context.md)

### Human-in-the-Loop (HITL)

Require approval for dangerous tools:

```python
async with create_agent(
    model="anthropic:claude-sonnet-4",
    tools=[*fs_tools, *shell_tools],
    need_user_approve_tools=["shell", "edit"],  # These require approval
) as runtime:
    ...
```

Full guide: [docs/toolset.md](docs/toolset.md)

### Subagents

Delegate specialized tasks to child agents:

```python
from pai_agent_sdk.subagents import SubagentConfig

config = SubagentConfig(
    name="researcher",
    description="Research specialist for web searches",
    system_prompt="You are a research specialist...",
    tools=["search_with_tavily", "visit_webpage"],
)

async with create_agent(
    "anthropic:claude-sonnet-4",
    tools=[SearchTool, VisitTool, ViewTool],
    subagent_configs=[config],
    include_builtin_subagents=True,  # debugger, explorer, code-reviewer, searcher
) as runtime:
    ...
```

Full guide: [docs/subagent.md](docs/subagent.md)

### Browser Automation

Use DockerBrowserSandbox for headless Chrome:

```python
from pai_agent_sdk.sandbox.browser.docker_ import DockerBrowserSandbox
from pai_agent_sdk.toolsets.browser_use import BrowserUseToolset

# See examples/browser_use.py for complete implementation
```

Full example: [examples/browser_use.py](examples/browser_use.py)

### Streaming Events

Handle real-time events with lifecycle hooks:

```python
from pai_agent_sdk.agents import stream_agent
from pai_agent_sdk.stream import AgentStartContext, AgentCompleteContext

async def on_start(ctx: AgentStartContext):
    print(f"Agent started with prompt: {ctx.user_prompt}")

async def on_complete(ctx: AgentCompleteContext):
    print(f"Agent completed in {ctx.duration_ms}ms")

async with stream_agent(
    runtime.agent, "Hello", runtime.ctx,
    on_agent_start=on_start,
    on_agent_complete=on_complete,
) as stream:
    async for event in stream:
        pass
```

Full guide: [docs/streaming.md](docs/streaming.md)

### Message Bus

Inter-agent communication and user steering:

```python
# Send message from external code
await ctx.bus.send("user", "Please focus on security issues")

# Agent can read messages via bus tools
```

Full guide: [docs/message-bus.md](docs/message-bus.md)

## Complete Examples

| Example                                              | Description                                              |
| ---------------------------------------------------- | -------------------------------------------------------- |
| [examples/general.py](examples/general.py)           | Production pattern: streaming, HITL, session persistence |
| [examples/deepresearch.py](examples/deepresearch.py) | Autonomous research agent with web tools                 |
| [examples/browser_use.py](examples/browser_use.py)   | Browser automation with Docker sandbox                   |

## Configuration Reference

### Model Strings

```python
# Native pydantic-ai format
"openai:gpt-4o"
"anthropic:claude-sonnet-4"
"gemini:gemini-1.5-pro"

# Gateway mode (via proxy)
"mygateway@openai:gpt-4o"
# Requires: MYGATEWAY_API_KEY, MYGATEWAY_BASE_URL
```

### Environment Variables

Check examples/.env.example for all available environment variables

| Variable                                     | Purpose              |
| -------------------------------------------- | -------------------- |
| `TAVILY_API_KEY`                             | Web search           |
| `FIRECRAWL_API_KEY`                          | Web scraping         |
| `GOOGLE_SEARCH_API_KEY` + `GOOGLE_SEARCH_CX` | Google Custom Search |

### ModelConfig

```python
from pai_agent_sdk.context import ModelConfig, ModelCapability

ModelConfig(
    context_window=200_000,
    capabilities={ModelCapability.vision},
)
```

### ToolConfig

```python
from pai_agent_sdk.context import ToolConfig

ToolConfig(
    tavily_api_key="...",
    firecrawl_api_key="...",
    google_search_api_key="...",
    google_search_cx="...",
)
```

## Builtin Tools

Import from `pai_agent_sdk.toolsets.core.*`:

| Module       | Tools                                                                 |
| ------------ | --------------------------------------------------------------------- |
| `filesystem` | view, edit, multi_edit, replace, ls, glob, grep                       |
| `shell`      | shell (command execution)                                             |
| `web`        | search_with_tavily, search_with_google, visit_webpage, save_web_files |
| `document`   | pdf_convert, office_to_markdown                                       |
| `content`    | validate_json                                                         |
| `context`    | thinking, handoff, to_do_read, to_do_write                            |
| `enhance`    | screenshot                                                            |
| `multimodal` | view (images, video, audio)                                           |

## Builtin Subagents

Available via `include_builtin_subagents=True`:

| Name          | Purpose               | Required Tools       |
| ------------- | --------------------- | -------------------- |
| debugger      | Root cause analysis   | glob, grep, view, ls |
| explorer      | Codebase navigation   | glob, grep, view, ls |
| code-reviewer | Code quality analysis | glob, grep, view, ls |
| searcher      | Web research          | search               |

## Troubleshooting

**Missing API key errors**: Set required environment variables or pass via `ToolConfig`.

**Tool not available**: Check `tool.is_available()` or set `skip_unavailable=True` in Toolset.

**Session restore fails**: Ensure message history and ResumableState are both saved/restored.

**Docker sandbox issues**: Ensure Docker daemon is running and user has Docker permissions.
