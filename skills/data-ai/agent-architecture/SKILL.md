---
name: agent-architecture
description: Use when designing or implementing AI agent systems. Covers tool-using agents with mandatory guardrails, SSE streaming (FastAPI → Next.js via Vercel AI SDK v6), LangGraph stateful multi-agent graphs, episodic memory via pgvector, MCP overview, and production failure modes with anti-pattern/fix code pairs.
---

# Agent Architecture

> See `references/multi-agent-patterns.md` for LangGraph orchestrator implementation details.
> See `references/memory-orchestration.md` for episodic memory, context management, and state persistence.

## Quick Start

Two steps. Step 1 is under 5 minutes. By the end of Step 2 you have a tool-using agent with mandatory guardrails. Install: `npm install @anthropic-ai/sdk` (TypeScript) or `pip install anthropic` (Python).

### Step 1: Running Agent, No Tools

No loop, no guardrails needed — one call, one response.

```python
# Python
import anthropic
client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from env
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "What is the capital of France?"}],
)
print(response.content[0].text)
```

```typescript
// TypeScript
import Anthropic from "@anthropic-ai/sdk";
const client = new Anthropic();
const response = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "What is the capital of France?" }],
});
const block = response.content[0];
console.log(block.type === "text" ? block.text : "");
```

### Step 2: Tool-Using Agent with Guardrails

Add a `fetch_url` tool and a manual tool-use loop. `MAX_TURNS` and `MAX_TOKENS_PER_RUN` are required from the first tool example onward — no example ships without them.

```python
# Python
import anthropic
import urllib.request

MAX_TURNS = 10              # REQUIRED: hard limit on agent iterations
MAX_TOKENS_PER_RUN = 4096   # REQUIRED: caps per-call output tokens

client = anthropic.Anthropic()

tools = [
    {
        "name": "fetch_url",
        "description": "Fetch the content of a URL and return the text (first 2000 chars)",
        "input_schema": {
            "type": "object",
            "properties": {
                "url": {"type": "string", "description": "The URL to fetch"},
            },
            "required": ["url"],
        },
    }
]

def execute_tool(name: str, input_data: dict) -> str:
    if name == "fetch_url":
        with urllib.request.urlopen(input_data["url"]) as r:
            return r.read().decode("utf-8")[:2000]
    raise ValueError(f"Unknown tool: {name}")

def run_agent(user_message: str) -> str:
    messages = [{"role": "user", "content": user_message}]

    for turn in range(MAX_TURNS):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=MAX_TOKENS_PER_RUN,
            tools=tools,
            messages=messages,
        )

        if response.stop_reason == "end_turn":
            text = next((b for b in response.content if b.type == "text"), None)
            return text.text if text else ""

        if response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result,
                    })
            messages.append({"role": "user", "content": tool_results})

    raise RuntimeError(
        f"Agent exceeded MAX_TURNS ({MAX_TURNS}). Aborting to prevent runaway costs."
    )

print(run_agent("Fetch https://example.com and summarise it."))
```

```typescript
// TypeScript
import Anthropic from "@anthropic-ai/sdk";

const MAX_TURNS = 10; // REQUIRED: hard limit on agent iterations
const MAX_TOKENS_PER_RUN = 4096; // REQUIRED: caps per-call output tokens

const client = new Anthropic();

const tools: Anthropic.Tool[] = [
  {
    name: "fetch_url",
    description:
      "Fetch the content of a URL and return the text (first 2000 chars)",
    input_schema: {
      type: "object",
      properties: {
        url: { type: "string", description: "The URL to fetch" },
      },
      required: ["url"],
    },
  },
];

async function executeTool(
  name: string,
  input: { url: string },
): Promise<string> {
  if (name === "fetch_url") {
    const res = await fetch(input.url);
    return (await res.text()).slice(0, 2000);
  }
  throw new Error(`Unknown tool: ${name}`);
}

async function runAgent(userMessage: string): Promise<string> {
  const messages: Anthropic.MessageParam[] = [
    { role: "user", content: userMessage },
  ];

  for (let turn = 0; turn < MAX_TURNS; turn++) {
    const response = await client.messages.create({
      model: "claude-sonnet-4-6",
      max_tokens: MAX_TOKENS_PER_RUN,
      tools,
      messages,
    });

    if (response.stop_reason === "end_turn") {
      const text = response.content.find((b) => b.type === "text");
      return text?.type === "text" ? text.text : "";
    }

    if (response.stop_reason === "tool_use") {
      messages.push({ role: "assistant", content: response.content });

      const toolResults: Anthropic.ToolResultBlockParam[] = await Promise.all(
        response.content
          .filter((b): b is Anthropic.ToolUseBlock => b.type === "tool_use")
          .map(async (toolUse) => ({
            type: "tool_result" as const,
            tool_use_id: toolUse.id,
            content: await executeTool(
              toolUse.name,
              toolUse.input as { url: string },
            ),
          })),
      );

      messages.push({ role: "user", content: toolResults });
    }
  }

  throw new Error(
    `Agent exceeded MAX_TURNS (${MAX_TURNS}). Aborting to prevent runaway costs.`,
  );
}

console.log(await runAgent("Fetch https://example.com and summarise it."));
```

> **Shortcut:** `client.beta.messages.tool_runner()` (Python) / `client.beta.messages.toolRunner()` (TypeScript) handle the loop automatically. The manual loop above is the better teaching pattern because guardrail placement is explicit and auditable.

## 1. Guardrails (Required, Not Optional)

Every tool-use loop must define `MAX_TURNS` and `MAX_TOKENS_PER_RUN` as named constants before the first call. These are not suggestions — cost runaway is the primary production failure mode for agents. At $3–15 per million output tokens, an unguarded 100-turn loop can accumulate hundreds of dollars before anyone notices.

| Constant             | Recommended Default | Effect                                                                    |
| -------------------- | ------------------- | ------------------------------------------------------------------------- |
| `MAX_TURNS`          | 10                  | Aborts loop after N iterations — raise `RuntimeError` / `Error` on breach |
| `MAX_TOKENS_PER_RUN` | 4096                | Caps output tokens per call — set on every `messages.create()` call       |

**Why named constants?** Magic numbers inline (`range(10)`, `max_tokens=4096`) are invisible in code review. Named constants are grep-able, auditable, and communicate intent. `max_tokens` must be set explicitly on every `messages.create()` call — SDK defaults vary by version.

## 2. Streaming (FastAPI → Next.js)

Streaming matters when responses are long (>2s to first token) or UX requires progressive rendering. Stack: FastAPI `StreamingResponse` (server) → Next.js API route + Vercel AI SDK v6 `useChat` (client).

### Server: Python FastAPI

```python
# server.py — FastAPI SSE endpoint
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import anthropic, json

app = FastAPI()
client = anthropic.AsyncAnthropic()
MAX_TOKENS_PER_RUN = 4096  # REQUIRED

async def stream_agent_response(user_message: str):
    """Async generator yielding SSE-formatted text chunks."""
    async with client.messages.stream(
        model="claude-sonnet-4-6",
        max_tokens=MAX_TOKENS_PER_RUN,
        messages=[{"role": "user", "content": user_message}],
    ) as stream:
        async for text in stream.text_stream:
            yield f"data: {json.dumps({'text': text})}\n\n"
    yield "data: [DONE]\n\n"

@app.post("/api/chat")
async def chat(request: dict):
    return StreamingResponse(
        stream_agent_response(request["message"]),
        media_type="text/event-stream",
        headers={"Cache-Control": "no-cache", "Connection": "keep-alive"},
    )
```

### Client: Next.js API Route (Vercel AI SDK v6)

Proxy through a Next.js API route using `@ai-sdk/anthropic` — avoids CORS and keeps the API key server-side.

```typescript
// app/api/chat/route.ts
import { streamText, convertToModelMessages, UIMessage } from "ai";
import { createAnthropic } from "@ai-sdk/anthropic";

const anthropic = createAnthropic(); // reads ANTHROPIC_API_KEY from env
export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: anthropic("claude-sonnet-4-6"),
    messages: await convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse();
}
```

```typescript
// app/page.tsx — React client component
'use client';
import { useChat } from '@ai-sdk/react';
import { useState } from 'react';

export default function AgentChat() {
  const [input, setInput] = useState('');
  const { messages, sendMessage } = useChat(); // defaults to /api/chat
  return (
    <div>
      {messages.map((m) => (
        <div key={m.id}>
          <strong>{m.role}:</strong>{' '}
          {m.parts.map((p, i) => p.type === 'text' ? <span key={i}>{p.text}</span> : null)}
        </div>
      ))}
      <form onSubmit={(e) => { e.preventDefault(); sendMessage({ text: input }); setInput(''); }}>
        <input value={input} onChange={(e) => setInput(e.target.value)} />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

> **FastAPI direct:** `TextStreamChatTransport` in AI SDK v6 can point at a custom URL (like the FastAPI endpoint directly). However, the FastAPI endpoint must produce AI SDK-compatible SSE format, not raw Anthropic SSE. The Next.js API route proxy pattern is simpler and recommended.

## 3. Multi-Agent with LangGraph

LangGraph 1.x (stable since October 2025, zero breaking changes) — use `create_react_agent` from `langgraph.prebuilt`. TypeScript SDK (`@langchain/langgraph`) exists but Python is the primary ecosystem.

```python
# Python
from langchain_anthropic import ChatAnthropic
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.memory import InMemorySaver        # dev/test only
# from langgraph.checkpoint.postgres import PostgresSaver    # production

MAX_TURNS = 10  # REQUIRED: passed as recursion_limit

model = ChatAnthropic(model="claude-sonnet-4-6")

def fetch_url(url: str) -> str:
    """Fetch web content from a URL (first 2000 chars)."""
    import urllib.request
    with urllib.request.urlopen(url) as r:
        return r.read().decode()[:2000]

# Dev: InMemorySaver — state is lost on server restart
agent = create_react_agent(model, tools=[fetch_url], checkpointer=InMemorySaver())

# Production: PostgresSaver — state persists across restarts
# with PostgresSaver.from_conn_string("postgresql://...") as cp:
#     agent = create_react_agent(model, tools=[fetch_url], checkpointer=cp)

config = {"configurable": {"thread_id": "session-1"}, "recursion_limit": MAX_TURNS}
result = agent.invoke(
    {"messages": [{"role": "user", "content": "Fetch https://example.com and summarise it"}]},
    config,
)
print(result["messages"][-1].content)
```

| Situation                           | Pattern                             |
| ----------------------------------- | ----------------------------------- |
| Task fits in one context window     | Single agent + tools                |
| Requires parallelism                | Orchestrator + parallel subagents   |
| Exceeds context window              | Orchestrator + sequential subagents |
| Long-running with human checkpoints | LangGraph with `PostgresSaver`      |

`InMemorySaver` is for development only — state is lost on restart. Use `PostgresSaver` from `langgraph-checkpoint-postgres` in production.

## 4. Memory and MCP

### Episodic Memory via pgvector

Episodic memory stores agent conversation history as vector embeddings — agents recall past interactions rather than relying solely on in-context messages.

```python
# Python — store and recall agent episodes
import openai, pgvector

EMBEDDING_MODEL = "text-embedding-3-small"
openai_client = openai.OpenAI()

def store_episode(db, session_id: str, role: str, content: str) -> None:
    vector = openai_client.embeddings.create(model=EMBEDDING_MODEL, input=content).data[0].embedding
    db.execute(
        "INSERT INTO agent_episodes (session_id, role, content, embedding, embedding_model_version) "
        "VALUES (%s, %s, %s, %s, %s)",
        (session_id, role, content, pgvector.encode(vector), f"{EMBEDDING_MODEL}-v1"),
    )

def recall_episodes(db, query: str, top_k: int = 5) -> list[dict]:
    vector = pgvector.encode(
        openai_client.embeddings.create(model=EMBEDDING_MODEL, input=query).data[0].embedding
    )
    return db.fetchall(
        "SELECT role, content, 1 - (embedding <=> %s) AS score FROM agent_episodes "
        "WHERE embedding_model_version = %s ORDER BY embedding <=> %s LIMIT %s",
        (vector, f"{EMBEDDING_MODEL}-v1", vector, top_k),
    )
```

**Schema note:** `agent_episodes` requires an `embedding_model_version` column — same pattern as `document_chunks` in the RAG skill. Without it, embeddings from different model versions are silently compared, producing garbage recall scores. See the `rag-vector-search` skill for HNSW index setup, chunking strategies, and eval patterns.

### MCP (Model Context Protocol)

MCP is an open standard introduced by Anthropic in November 2024, now governed by the Linux Foundation / AAIF. It standardizes how AI models connect to external tools and data sources — defining a client-server protocol built on JSON-RPC 2.0 so any AI host (Claude, ChatGPT, Cursor) can invoke any MCP server without custom connectors. Where the tool-use patterns in this skill cover tools you build and host yourself, MCP defines how to expose those tools (or any existing service) as a reusable server that any compliant AI client can call. If you're building agents that need to connect to third-party services or share tool implementations across multiple AI systems, MCP is the integration layer to reach for.

## When NOT to Use Agents

> **Before reaching for an agent, check these criteria:**
>
> - **Single deterministic transformation** — no agent needed; a function is faster and cheaper
> - **Task has a known, fixed set of steps** — a workflow or pipeline is more predictable and debuggable
> - **Latency is the primary constraint** — each agent turn adds 300–2000ms; a 5-turn agent is 1–10s of overhead
> - **Accuracy must be >99%** — agent error rates compound (see Bag-of-Agents pitfall below); a single well-prompted call is more reliable
>
> If a capable single model can do the task, use one call. Agents are for tasks that require dynamic decision-making, external data retrieval, or multi-step reasoning that cannot be pre-planned.

## Gotchas

### Gotcha 1: Runaway Costs (Agent Loop Without Termination)

**Warning signs:**

- API cost spikes on dashboards with no corresponding feature launch
- Agent "spinning" — same or similar tool calls repeated across turns
- Simple queries taking many seconds or minutes to respond

**Why it happens:** Developers test with small prompts that terminate in 1–2 turns. The loop never runs long in development, so no one adds a guard.

```python
# BAD: no termination guard — will run until API error or budget exhaustion
while True:
    response = client.messages.create(model="claude-sonnet-4-6", max_tokens=4096, ...)
    if response.stop_reason == "end_turn":
        break
    # What if stop_reason is never "end_turn"?
    handle_tool_use(response)
```

```python
# GOOD: named constants, raises on breach — visible in code review
MAX_TURNS = 10
MAX_TOKENS_PER_RUN = 4096

for turn in range(MAX_TURNS):
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=MAX_TOKENS_PER_RUN,
        ...
    )
    if response.stop_reason == "end_turn":
        return extract_text(response)
    handle_tool_use(response)

raise RuntimeError(
    f"Exceeded MAX_TURNS ({MAX_TURNS}). Check tool logic for infinite loops."
)
```

### Gotcha 2: Untyped Sub-Agent Handoffs

**Warning signs:**

- Sub-agent returns answers that reference fields not in the original task
- Different runs of the same pipeline produce structurally different outputs
- Adding a new orchestrator output field breaks one sub-agent but not others

**Why it happens:** Multi-agent systems are prototyped with string handoffs because it's easy. When the output format changes (even a field rename), downstream agents break without raising an exception.

```typescript
// BAD: untyped string handoff — downstream agent can't validate input
const result = await orchestratorAgent(task); // returns string
const subResult = await subAgent(result); // what schema does subAgent expect?
```

```typescript
// GOOD: typed contract at the handoff boundary — throws on bad schema
interface SearchHandoff {
  query: string;
  maxResults: number;
  requiredFields: string[];
}

function validateHandoff(raw: unknown): SearchHandoff {
  const h = raw as SearchHandoff;
  if (typeof h.query !== "string")
    throw new Error("Invalid handoff: missing query");
  if (typeof h.maxResults !== "number")
    throw new Error("Invalid handoff: missing maxResults");
  return h;
}

const rawHandoff = await orchestratorAgent(task);
const handoff = validateHandoff(JSON.parse(rawHandoff)); // throws on bad schema
const subResult = await subAgent(handoff);
```

### Gotcha 3: Bag-of-Agents Error Multiplication

**Warning signs:**

- Final output quality is worse than a single-agent approach on the same task
- Early-stage agent errors appear in final outputs with greater confidence
- Removing an agent from the pipeline improves output quality

**Why it happens:** Developers treat agent accuracy as additive. It's multiplicative. A pipeline of 5 agents each 90% reliable has a combined reliability of 0.9^5 = 59%. Each agent that processes a flawed output amplifies the error — the final agent presents the hallucination as fact.

```python
# BAD: no output validation between agents — errors compound silently
result_1 = agent_1(user_input)   # 90% reliable
result_2 = agent_2(result_1)     # 90% × 90% = 81%
result_3 = agent_3(result_2)     # 73%
result_4 = agent_4(result_3)     # 66%
final    = agent_5(result_4)     # 59% — worse than a single capable model
```

```python
# GOOD: validate at each stage; abort early if output fails schema
result_1 = agent_1(user_input)
if not validate_stage(result_1, schema=SearchResultSchema):
    raise ValueError("Stage 1 output failed validation — aborting pipeline")

result_2 = agent_2(result_1)
if not validate_stage(result_2, schema=SummarySchema):
    raise ValueError("Stage 2 output failed validation — aborting pipeline")
# ... repeat for each stage
```

> **Rule of thumb:** If a single capable model can do the task, use one agent. Add agents only when tasks require genuine parallelism or exceed a single context window.

## Version Context

| Library                         | Version      | Notes                                                        |
| ------------------------------- | ------------ | ------------------------------------------------------------ |
| `@anthropic-ai/sdk`             | 0.37.x       | TypeScript — `tool_use` stop reason pattern                  |
| `anthropic`                     | 0.40.x       | Python equivalent                                            |
| `ai` (Vercel AI SDK)            | 6.x (6.0.97) | `useChat`, `streamText`, `convertToModelMessages`            |
| `@ai-sdk/anthropic`             | latest       | `createAnthropic()` — breaking change from `new Anthropic()` |
| `@ai-sdk/react`                 | 6.x          | `useChat` hook with `parts` API                              |
| `langgraph`                     | 1.x          | Stable since October 2025, zero breaking changes             |
| `langgraph-checkpoint-postgres` | 1.x          | `PostgresSaver` for production checkpointing                 |
| `langchain-anthropic`           | latest       | `ChatAnthropic` model provider for LangGraph                 |
| `fastapi`                       | 0.115+       | `StreamingResponse` + async generator for SSE                |

## See Also

- [RAG / Vector Search](../rag-vector-search/SKILL.md) — pgvector pattern (HNSW index, embedding pipeline, hybrid search)
- [Prompt Engineering](../prompt-engineering/SKILL.md) — system-prompt design for agent instruction structure
