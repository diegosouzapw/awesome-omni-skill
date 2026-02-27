---
name: multi-agent-patterns
description: Supervisor, swarm, and hierarchical multi-agent architectures with context isolation patterns.
agents: [morgan, atlas]
triggers: [multi-agent, orchestration, supervisor, swarm, coordination]
---

# Multi-Agent Architecture Patterns

Multi-agent architectures distribute work across multiple language model instances, each with its own context window. When designed well, this distribution enables capabilities beyond single-agent limits. When designed poorly, it introduces coordination overhead that negates benefits. The critical insight is that sub-agents exist primarily to isolate context, not to anthropomorphize role division.

## When to Activate

Activate this skill when:
- Single-agent context limits constrain task complexity
- Tasks decompose naturally into parallel subtasks
- Different subtasks require different tool sets or system prompts
- Building systems that must handle multiple domains simultaneously
- Scaling agent capabilities beyond single-context limits
- Designing production agent systems with multiple specialized components

## Core Concepts

Multi-agent systems address single-agent context limitations through distribution. Three dominant patterns exist: supervisor/orchestrator for centralized control, peer-to-peer/swarm for flexible handoffs, and hierarchical for layered abstraction. The critical design principle is context isolation—sub-agents exist primarily to partition context rather than to simulate organizational roles.

Effective multi-agent systems require explicit coordination protocols, consensus mechanisms that avoid sycophancy, and careful attention to failure modes including bottlenecks, divergence, and error propagation.

## Architectural Patterns

### Pattern 1: Supervisor/Orchestrator

The supervisor pattern places a central agent in control, delegating to specialists and synthesizing results. The supervisor maintains global state and trajectory, decomposes user objectives into subtasks, and routes to appropriate workers.

```
User Query -> Supervisor -> [Specialist, Specialist, Specialist] -> Aggregation -> Final Output
```

When to use: Complex tasks with clear decomposition, tasks requiring coordination across domains, tasks where human oversight is important.

Advantages: Strict control over workflow, easier to implement human-in-the-loop interventions, ensures adherence to predefined plans.

Disadvantages: Supervisor context becomes bottleneck, supervisor failures cascade to all workers, "telephone game" problem where supervisors paraphrase sub-agent responses incorrectly.

**The Telephone Game Problem and Solution**

Benchmarks found supervisor architectures initially performed 50% worse than optimized versions due to the "telephone game" problem where supervisors paraphrase sub-agent responses incorrectly, losing fidelity.

The fix: implement a `forward_message` tool allowing sub-agents to pass responses directly to users:

```python
def forward_message(message: str, to_user: bool = True):
    """
    Forward sub-agent response directly to user without supervisor synthesis.

    Use when:
    - Sub-agent response is final and complete
    - Supervisor synthesis would lose important details
    - Response format must be preserved exactly
    """
    if to_user:
        return {"type": "direct_response", "content": message}
    return {"type": "supervisor_input", "content": message}
```

### Pattern 2: Peer-to-Peer/Swarm

The peer-to-peer pattern removes central control, allowing agents to communicate directly based on predefined protocols. Any agent can transfer control to any other through explicit handoff mechanisms.

```python
def transfer_to_agent_b():
    return agent_b  # Handoff via function return

agent_a = Agent(
    name="Agent A",
    functions=[transfer_to_agent_b]
)
```

When to use: Tasks requiring flexible exploration, tasks where rigid planning is counterproductive, tasks with emergent requirements that defy upfront decomposition.

Advantages: No single point of failure, scales effectively for breadth-first exploration, enables emergent problem-solving behaviors.

Disadvantages: Coordination complexity increases with agent count, risk of divergence without central state keeper, requires robust convergence constraints.

### Pattern 3: Hierarchical

Hierarchical structures organize agents into layers of abstraction: strategic, planning, and execution layers. Strategy layer agents define goals and constraints; planning layer agents break goals into actionable plans; execution layer agents perform atomic tasks.

```
Strategy Layer (Goal Definition) -> Planning Layer (Task Decomposition) -> Execution Layer (Atomic Tasks)
```

When to use: Large-scale projects with clear hierarchical structure, enterprise workflows with management layers, tasks requiring both high-level planning and detailed execution.

## Context Isolation as Design Principle

The primary purpose of multi-agent architectures is context isolation. Each sub-agent operates in a clean context window focused on its subtask without carrying accumulated context from other subtasks.

**Isolation Mechanisms**

- Full context delegation: For complex tasks where the sub-agent needs complete understanding
- Instruction passing: For simple, well-defined subtasks
- File system memory: For complex tasks requiring shared state

## Failure Modes and Mitigations

**Failure: Supervisor Bottleneck**
The supervisor accumulates context from all workers, becoming susceptible to saturation and degradation.
Mitigation: Implement output schema constraints so workers return only distilled summaries.

**Failure: Coordination Overhead**
Agent communication consumes tokens and introduces latency.
Mitigation: Minimize communication through clear handoff protocols. Batch results where possible.

**Failure: Divergence**
Agents pursuing different goals without central coordination can drift from intended objectives.
Mitigation: Define clear objective boundaries. Implement convergence checks. Use time-to-live limits.

**Failure: Error Propagation**
Errors in one agent's output propagate to downstream agents.
Mitigation: Validate agent outputs before passing to consumers. Implement retry logic with circuit breakers.

## Guidelines

1. Design for context isolation as the primary benefit of multi-agent systems
2. Choose architecture pattern based on coordination needs, not organizational metaphor
3. Implement explicit handoff protocols with state passing
4. Use weighted voting or debate protocols for consensus
5. Monitor for supervisor bottlenecks and implement checkpointing
6. Validate outputs before passing between agents
7. Set time-to-live limits to prevent infinite loops
8. Test failure scenarios explicitly
