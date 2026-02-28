---
name: ai-agent-basics
description: Master AI agent fundamentals - architectures, ReAct patterns, cognitive loops, and autonomous system design
sasmp_version: "1.3.0"
bonded_agent: 01-ai-agent-fundamentals
bond_type: PRIMARY_BOND
version: "2.0.0"
---

# AI Agent Basics

Build production-grade AI agents with modern architectures and patterns.

## When to Use This Skill

Invoke this skill when:
- Designing new AI agent systems
- Implementing ReAct or Plan-and-Execute patterns
- Building autonomous task-solving agents
- Integrating cognitive loops into applications

## Parameter Schema

| Parameter | Type | Required | Description | Default |
|-----------|------|----------|-------------|---------|
| `task` | string | Yes | What agent capability to build | - |
| `architecture` | enum | No | `single`, `multi`, `hybrid` | `single` |
| `framework` | enum | No | `langchain`, `langgraph`, `custom` | `langgraph` |
| `complexity` | enum | No | `basic`, `intermediate`, `advanced` | `intermediate` |

## Quick Start

```python
# Basic ReAct Agent
from langgraph.prebuilt import create_react_agent
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(model="claude-sonnet-4-20250514")
agent = create_react_agent(llm, tools=[search, calculator])
result = await agent.ainvoke({"messages": [("user", "What is 25 * 4?")]})
```

## Core Patterns

### 1. ReAct Agent
```python
# Thought → Action → Observation loop
graph = StateGraph(AgentState)
graph.add_node("think", reason_node)
graph.add_node("act", action_node)
graph.add_node("observe", observation_node)
```

### 2. Plan-and-Execute
```python
# Create plan → Execute steps → Verify
planner = create_planner(llm)
executor = create_executor(llm, tools)
```

### 3. Reflexion
```python
# Execute → Reflect → Improve
agent_with_reflection = add_reflection_layer(base_agent)
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Agent loops forever | Add max_iterations limit |
| Wrong tool selected | Improve tool descriptions |
| Context too large | Implement summarization |
| Slow responses | Use streaming |

## Best Practices

- Start with simple single-agent before multi-agent
- Always add circuit breakers (max iterations)
- Use verbose mode for debugging
- Implement human-in-the-loop for critical decisions

## Related Skills

- `llm-integration` - LLM API configuration
- `tool-calling` - Function calling implementation
- `agent-memory` - Memory systems

## References

- [LangGraph Docs](https://langchain-ai.github.io/langgraph/)
- [Anthropic Agent Patterns](https://docs.anthropic.com/)
