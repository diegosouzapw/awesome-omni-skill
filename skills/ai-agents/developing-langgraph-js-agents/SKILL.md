---
name: developing-langgraph-js-agents
description: "Build, audit, review, and update LangGraph.js agents. Use PROACTIVELY when working with LangGraph, @langchain/langgraph, agent graphs, state machines, or AI workflows in TypeScript/JavaScript. Covers creating new agents, adding features, debugging, testing, and optimizing. (user)"
---

<essential_principles>

## How LangGraph.js Agents Work

LangGraph decomposes agents into **discrete nodes** (functions) connected through **shared state**. Execution flows through a graph where nodes do work and edges determine what runs next.

### 1. State-First Design

State is the shared memory accessible to all nodes. Design state before nodes:

```typescript
import { Annotation, MessagesAnnotation } from "@langchain/langgraph";

const AgentState = Annotation.Root({
  ...MessagesAnnotation.spec,
  // Add custom fields with reducers
  context: Annotation<string[]>({
    reducer: (x, y) => x.concat(y),
    default: () => [],
  }),
});
```

**Critical rules:**
- Store raw data, not formatted text (format in nodes)
- Use reducers for fields that accumulate (messages, lists)
- Keep state minimal - only persist what's needed across steps

### 2. Nodes Do Work, Edges Route

```typescript
// Node: receives state, returns partial update
async function callModel(state: typeof AgentState.State) {
  const response = await model.invoke(state.messages);
  return { messages: [response] };
}

// Edge: determines next node
function shouldContinue(state: typeof AgentState.State) {
  const lastMessage = state.messages.at(-1);
  if (lastMessage?.tool_calls?.length) return "tools";
  return END;
}
```

### 3. Always Compile Before Use

```typescript
const graph = new StateGraph(AgentState)
  .addNode("agent", callModel)
  .addNode("tools", toolNode)
  .addEdge(START, "agent")
  .addConditionalEdges("agent", shouldContinue)
  .addEdge("tools", "agent")
  .compile(); // Required!
```

### 4. Checkpointers Enable Persistence

For conversation memory, human-in-the-loop, or fault tolerance:

```typescript
import { MemorySaver } from "@langchain/langgraph";

const graph = workflow.compile({
  checkpointer: new MemorySaver()
});

// Invoke with thread_id
await graph.invoke(input, {
  configurable: { thread_id: "user-123" }
});
```

</essential_principles>

<intake>

What would you like to do?

1. Build a new agent from scratch
2. Add a feature to an existing agent
3. Audit/review an agent's architecture
4. Debug an agent issue
5. Write tests for an agent
6. Optimize agent performance
7. Something else

**Wait for response, then read the matching workflow from `workflows/` and follow it.**

</intake>

<routing>

| Response | Workflow |
|----------|----------|
| 1, "new", "create", "build", "start", "scaffold" | `workflows/build-new-agent.md` |
| 2, "add", "feature", "implement", "extend" | `workflows/add-feature.md` |
| 3, "audit", "review", "check", "assess", "evaluate" | `workflows/audit-agent.md` |
| 4, "debug", "fix", "broken", "error", "bug", "issue" | `workflows/debug-agent.md` |
| 5, "test", "tests", "testing", "coverage" | `workflows/write-tests.md` |
| 6, "optimize", "performance", "slow", "fast", "improve" | `workflows/optimize-agent.md` |
| 7, other | Clarify intent, then select appropriate workflow |

</routing>

<verification_loop>

## After Every Change

```bash
# 1. TypeScript compiles?
npx tsc --noEmit

# 2. Tests pass?
npm test

# 3. Agent runs?
npx ts-node src/agent.ts
```

Report:
- "Build: ✓" or "Build: ✗ [error]"
- "Tests: X pass, Y fail"
- "Agent executed successfully" or "Runtime error: [details]"

</verification_loop>

<reference_index>

## Domain Knowledge

All in `references/`:

**LangChain Fundamentals:**
- langchain-fundamentals.md - Messages, chat models, structured output, retrieval, guardrails

**Architecture:**
- graph-api.md - StateGraph, nodes, edges, compilation
- functional-api.md - Tasks, entrypoints, when to use
- state-management.md - Annotations, reducers, state design

**Features:**
- tools.md - Creating and binding tools
- persistence.md - Checkpointers, memory, threads
- streaming.md - Real-time output modes
- interrupts.md - Human-in-the-loop patterns
- subgraphs.md - Composing multi-agent systems
- agent-chat-ui.md - Chat UI setup and integration
- agent-inbox.md - Inbox UI for interrupt management, ambient agents
- deployment.md - Local server, LangSmith Cloud, Studio, observability, time-travel

**Patterns:**
- common-patterns.md - ReAct, RAG, routing patterns
- multi-agent.md - Supervisor, hierarchical, network architectures
- agent-skills.md - Modular capabilities and skill loading
- anti-patterns.md - Common mistakes to avoid

</reference_index>

<workflows_index>

## Workflows

All in `workflows/`:

| File | Purpose |
|------|---------|
| build-new-agent.md | Create a new LangGraph.js agent from scratch |
| add-feature.md | Add capabilities to an existing agent |
| audit-agent.md | Review architecture and identify issues |
| debug-agent.md | Find and fix agent bugs |
| write-tests.md | Test nodes, graphs, and integrations |
| optimize-agent.md | Improve performance and reduce latency |

</workflows_index>

<templates_index>

## Templates

All in `templates/`:

| File | Purpose |
|------|---------|
| basic-agent.ts | Minimal ReAct agent scaffold |
| rag-agent.ts | Retrieval-augmented agent |
| multi-agent.ts | Multi-agent system with subgraphs |

</templates_index>

<external_docs>

## Official Documentation

For topics not fully covered here, consult:

- **LangGraph.js Docs**: https://docs.langchain.com/oss/javascript/langgraph/overview.md
- **API Reference**: https://langchain-ai.github.io/langgraphjs/reference/
- **GitHub Examples**: https://github.com/langchain-ai/langgraphjs/tree/main/examples
- **LangChain Tools**: https://docs.langchain.com/oss/javascript/langchain/tools.md

</external_docs>
