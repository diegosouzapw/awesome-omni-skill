---
name: agent-workflow-patterns
description: AI agent workflow patterns including ReAct agents, multi-agent systems, loop control, tool orchestration, and autonomous agent architectures. Use when building AI agents, implementing workflows, creating autonomous systems, or when user mentions agents, workflows, ReAct, multi-step reasoning, loop control, agent orchestration, or autonomous AI.
allowed-tools: Read, Write, Bash, Grep, Glob
---

# Agent Workflow Patterns

**Purpose:** Provide production-ready agent architectures, workflow patterns, and loop control strategies for building autonomous AI systems with Vercel AI SDK.

**Activation Triggers:**
- Building autonomous AI agents
- Implementing multi-step reasoning
- Creating agent workflows
- Tool orchestration and coordination
- Loop control and iteration management
- Multi-agent system architectures
- ReAct (Reasoning + Acting) patterns

**Key Resources:**
- `templates/react-agent.ts` - ReAct agent pattern
- `templates/multi-agent-system.ts` - Multiple specialized agents
- `templates/workflow-orchestrator.ts` - Workflow coordination
- `templates/loop-control.ts` - Iteration and safeguards
- `templates/tool-coordinator.ts` - Tool orchestration
- `scripts/validate-agent.sh` - Validate agent configuration
- `examples/` - Production agent implementations (RAG agent, SQL agent, etc.)

## Core Agent Patterns

### 1. ReAct Agent (Reasoning + Acting)

**When to use:** Complex problem-solving requiring iterative thought and action

**Template:** `templates/react-agent.ts`

**Pattern:**
```typescript
async function reactAgent(task: string, maxIterations: number = 5) {
  const tools = { /* tool definitions */ }
  let iteration = 0

  while (iteration < maxIterations) {
    // Reasoning step
    const thought = await generateText({
      model: openai('gpt-4o')
      messages: [
        { role: 'system', content: 'Think step-by-step...' }
        { role: 'user', content: task }
      ]
    })

    // Acting step (tool calls)
    const action = await generateText({
      model: openai('gpt-4o')
      tools
      toolChoice: 'auto'
      messages: [/* ... */]
    })

    // Check if task complete
    if (isComplete(action)) break
    iteration++
  }

  return result
}
```

**Best for:** Research, analysis, complex planning

### 2. Multi-Agent System

**When to use:** Complex domains requiring specialized expertise

**Template:** `templates/multi-agent-system.ts`

**Pattern:**
- Coordinator agent routes tasks
- Specialist agents handle specific domains
- Result aggregation and synthesis

**Best for:** Multi-domain problems, parallel task execution

### 3. Workflow Orchestration

**When to use:** Pre-defined sequences of steps

**Template:** `templates/workflow-orchestrator.ts`

**Pattern:**
- Define workflow steps
- Execute sequentially with error handling
- State management between steps
- Conditional branching

**Best for:** Structured processes, pipelines

## Loop Control Strategies

### 1. Iteration Limits

```typescript
const config = {
  maxIterations: 10
  onMaxIterations: 'return-last' | 'throw-error'
}
```

**Prevents:** Infinite loops

### 2. Cost Limits

```typescript
const config = {
  maxTokens: 10000
  onMaxTokens: 'graceful-stop'
}
```

**Prevents:** Runaway costs

### 3. Time Limits

```typescript
const config = {
  maxDuration: 30000, // 30 seconds
  onTimeout: 'return-partial'
}
```

**Prevents:** Long-running operations

### 4. Quality Gates

```typescript
const config = {
  stopCondition: (result) => result.confidence > 0.9
}
```

**Ensures:** Quality outputs

## Tool Orchestration

### Sequential Tool Execution

```typescript
const tools = {
  search: tool({ /* ... */ })
  analyze: tool({ /* ... */ })
  summarize: tool({ /* ... */ })
}

// AI decides order and usage
const result = await generateText({
  model: openai('gpt-4o')
  tools
  maxToolRoundtrips: 5
})
```

### Parallel Tool Execution

```typescript
const results = await Promise.all([
  callTool('search', { query: 'topic1' })
  callTool('search', { query: 'topic2' })
  callTool('search', { query: 'topic3' })
])
```

## Agent State Management

```typescript
interface AgentState {
  conversation: Message[]
  context: Record<string, any>
  toolResults: ToolResult[]
  iteration: number
}

class StatefulAgent {
  private state: AgentState

  async execute(task: string) {
    while (!this.isComplete()) {
      await this.step()
      this.updateState()
    }
    return this.state
  }
}
```

## Production Best Practices

### 1. Error Recovery

```typescript
try {
  result = await agent.execute(task)
} catch (error) {
  if (error.code === 'MAX_ITERATIONS') {
    return agent.getBestSoFar()
  }
  throw error
}
```

### 2. Monitoring

```typescript
agent.on('iteration', ({ count, result }) => {
  metrics.record('agent.iteration', { count })
})
```

### 3. Safeguards

- Rate limiting
- Input validation
- Output sanitization
- Cost tracking

## Common Agent Architectures

### 1. RAG Agent

**Example:** `examples/rag-agent.ts`

Retrieves information and answers questions

### 2. SQL Agent

**Example:** `examples/sql-agent.ts`

Queries databases using natural language

### 3. Research Agent

**Example:** `examples/research-agent.ts`

Gathers and synthesizes information

### 4. Code Agent

**Example:** `examples/code-agent.ts`

Writes and debugs code

## Resources

**Templates:**
- `react-agent.ts` - ReAct pattern implementation
- `multi-agent-system.ts` - Multi-agent coordination
- `workflow-orchestrator.ts` - Workflow execution
- `loop-control.ts` - Iteration safeguards
- `tool-coordinator.ts` - Tool orchestration

**Scripts:**
- `validate-agent.sh` - Agent config validation

**Examples:**
- `rag-agent.ts` - Complete RAG agent
- `sql-agent.ts` - Natural language SQL
- `research-agent.ts` - Information gathering
- `code-agent.ts` - Code generation

---

**SDK Version:** Vercel AI SDK 5+
**Agent Frameworks:** Built-in tools, MCP integration

**Best Practice:** Start simple (single tool), add complexity as needed
