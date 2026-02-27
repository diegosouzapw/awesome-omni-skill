---
name: ai-assistant-chat
description: Build self-hosted AI chat assistants using CopilotKit + LangGraph. Use when implementing conversational AI interfaces with agentic backends, streaming responses, shared state between frontend and backend, or generative UI. This pattern uses NO hosted services - both CopilotKit runtime and LangGraph agent run on your own infrastructure. Triggers on requests to build chat interfaces, AI assistants, conversational agents, or integrate LangGraph with React frontends.
---

# Self-Hosted AI Assistant Chat (CopilotKit + LangGraph)

Build production-ready AI chat interfaces with full infrastructure control. This pattern connects a Next.js frontend (CopilotKit) to a Python backend (LangGraph) without relying on any hosted AI orchestration services.

## What This Pattern Is NOT

- **NOT Hosted CopilotKit**: No `cloud.copilotkit.ai` - the CopilotKit runtime runs in your Next.js API route
- **NOT Hosted LangGraph**: No LangSmith Cloud or LangGraph Platform - the agent runs on a self-hosted FastAPI server
- **NOT SaaS Dependencies**: Full control over data flow, no external orchestration services

## Architecture Overview

```
Browser
   ↓ (HTTP/SSE)
Next.js Frontend (port 3000)
   ├── React UI (CopilotKit components)
   └── /api/copilotkit (CopilotRuntime bridge)
       ↓ (HTTP POST)
FastAPI Backend (port 8123)
   ├── LangGraph Agent (state machine)
   └── Tool Execution (backend + frontend actions)
```

For detailed architecture diagrams and data flow: See [references/architecture.md](references/architecture.md)

## Quick Start

### 1. Frontend Setup (Next.js + CopilotKit)

Install CopilotKit packages:
```bash
pnpm add @copilotkit/react-core @copilotkit/react-ui @copilotkit/runtime
```

Wrap app with CopilotKit provider pointing to local runtime:
```tsx
// layout.tsx
import { CopilotKit } from "@copilotkit/react-core";

export default function Layout({ children }) {
  return (
    <CopilotKit runtimeUrl="/api/copilotkit" agent="my_agent">
      {children}
    </CopilotKit>
  );
}
```

Create the runtime bridge (this is the self-hosted runtime):
```ts
// app/api/copilotkit/route.ts
import { CopilotRuntime, ExperimentalEmptyAdapter } from "@copilotkit/runtime";
import { LangGraphHttpAgent } from "@copilotkit/runtime/agents";

const AGENT_URL = process.env.AGENT_URL || "http://localhost:8123";

export async function POST(req: Request) {
  const agent = new LangGraphHttpAgent({
    name: "my_agent",
    url: AGENT_URL,
    agentId: "my_agent",
  });

  const runtime = new CopilotRuntime({
    agents: [agent],
    modelAdapter: new ExperimentalEmptyAdapter(),
  });

  return runtime.streamHttpServerResponse(req, req.headers);
}
```

For complete frontend setup: See [references/frontend-setup.md](references/frontend-setup.md)

### 2. Backend Setup (FastAPI + LangGraph)

Install Python dependencies:
```bash
pip install langgraph langchain-openai copilotkit ag-ui-langgraph fastapi uvicorn
```

Create the LangGraph agent:
```python
# agent/src/agent.py
from langgraph.graph import StateGraph, END
from copilotkit import CopilotKitState

class AgentState(CopilotKitState):
    # Add custom state fields synced with frontend
    custom_data: list[str] = []

def chat_node(state: AgentState, config):
    model = ChatOpenAI(model="gpt-4o")
    # Bind tools from CopilotKit state
    model_with_tools = model.bind_tools(state.copilotkit.tools)
    response = model_with_tools.invoke(state.messages, config)
    return {"messages": [response]}

# Build graph
workflow = StateGraph(AgentState)
workflow.add_node("chat", chat_node)
workflow.set_entry_point("chat")
graph = workflow.compile(checkpointer=MemorySaver())
```

Create FastAPI server:
```python
# agent/main.py
from fastapi import FastAPI
from ag_ui.langgraph import LangGraphAGUIAgent

app = FastAPI()
agent = LangGraphAGUIAgent(graph=graph, config={"configurable": {"thread_id": "1"}})

@app.post("/")
async def run_agent(request: Request):
    return agent.run(request)
```

For complete backend setup: See [references/backend-setup.md](references/backend-setup.md)

## Key Integration Points

### Shared State

State defined in `AgentState` automatically syncs between frontend and backend:

```tsx
// Frontend: Access shared state
const { state } = useCoAgent<AgentState>();
console.log(state.custom_data);
```

```python
# Backend: Modify shared state
def my_node(state: AgentState):
    return {"custom_data": state.custom_data + ["new item"]}
```

### Frontend Actions (Generative UI)

Define actions in React that the backend can invoke:

```tsx
useCopilotAction({
  name: "render_weather",
  description: "Display weather card",
  parameters: [{ name: "location", type: "string" }],
  renderAndWaitForResponse: ({ args }) => <WeatherCard location={args.location} />,
});
```

The agent can call `render_weather` and receive user responses.

### Backend Tools

Define tools that execute server-side:

```python
@tool
def search_database(query: str) -> str:
    """Search the internal database."""
    return db.search(query)

# Add to model binding
model.bind_tools([search_database] + state.copilotkit.tools)
```

