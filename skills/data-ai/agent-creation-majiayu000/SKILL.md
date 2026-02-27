---
name: agent-creation
description: Systematically design and validate specialist agents with evidence-based prompting, tooling contracts, and adversarial evaluation.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Task, TodoWrite
model: sonnet
x-version: 3.2.0
x-category: foundry
x-vcl-compliance: v3.1.1
x-cognitive-frames:
  - HON
  - MOR
  - COM
  - CLS
  - EVD
  - ASP
  - SPC
---




### L1 Improvement
- Rebuilt the skill to follow the Skill Forge section cadence with explicit guardrails, validation hooks, and confidence ceilings.
- Added prompt-architect style constraint extraction, contract-first design, and adversarial testing steps to avoid shallow agent definitions.

## STANDARD OPERATING PROCEDURE

### Purpose
Create production-ready specialist agents with clear personas, tool wiring, evaluation scaffolding, and documentation that can be routed by orchestration layers.

### Trigger Conditions
- Positive: requests to create or refine an agent, multi-agent system design, agent prompt hardening, adding new tools to an agent.
- Negative/reroute: single-turn tasks better handled by micro-skill-creator, standalone prompt tuning handled by prompt-architect, or skill structure requests handled by skill-builder/skill-forge.

### Guardrails
- Use English-first outputs with explicit confidence ceilings following prompt-architect rules.
- Structure-first: deliver the agent spec (frontmatter + body), examples, and validation notes; avoid "created" confirmations without artifacts.
- Avoid duplication: check existing registry entries; prefer specialization over overlap.
- Run adversarial probes (boundary, failure, misuse) before considering the agent production-ready.
- Respect hooks latency targets from Skill Forge: pre_hook_target_ms 20/100 max; post_hook_target_ms 100/1000 max.

### Execution Phases
1. **Discovery**: Capture intent, domain, constraints (hard/soft/inferred), tools, and expected outputs.
2. **Design**: Define persona, responsibilities, tool permissions, memory strategy, and decision checkpoints.
3. **Prompt Construction**: Author the system prompt with role, inputs, outputs, style, and escalation rules; add few-shot patterns if helpful.
4. **Validation**: Create test cases (happy path + edge), run self-consistency checks, and document evaluation results with confidence ceilings.
5. **Delivery**: Package the agent spec, integration notes, and next-step risks; register metadata for agent-selector routing.

### Pattern Recognition
- Greenfield agent for new domain → emphasize constraint extraction and baseline tests.
- Refining underperforming agent → capture failure cases and add targeted guardrails.
- Multi-agent topology → design interaction contracts, escalation paths, and boundaries between agents.

### Advanced Techniques
- Use program-of-thought to decompose complex roles into capabilities and checkpoints.
- Apply contrastive prompting to defend against prompt drift and misuse.
- Capture evaluation harness steps so skill-forge or CI can rerun them.

### Common Anti-Patterns
- Shipping an agent without tool/contracts defined.
- Using generic personas without domain evidence or examples.
- Skipping adversarial tests or confidence ceilings.

### Practical Guidelines
- Prefer specific tool verbs over open-ended "help" instructions.
- Keep output formats deterministic (JSON or structured text) to ease downstream parsing.
- Record INFERRED constraints and seek confirmation before finalizing.

### Cross-Skill Coordination
- Upstream: prompt-architect for constraint extraction and clarity; skill-builder for directory scaffolding.
- Parallel: cognitive-lensing for alternative reasoning frames; meta-tools for tooling composition.
- Downstream: agent-selector for routing; recursive-improvement for iterative hardening.

### MCP Requirements
- Optional memory/vector MCP for loading prior agent outcomes; tag sessions with WHO=agent-creation-{session}, WHY=skill-execution.
- If external tools are added, document authentication and rate-limit assumptions in the agent spec.

### Input/Output Contracts
```yaml
inputs:
  task: string  # required description of the agent's job
  domain: string  # required domain or vertical
  tools: list[string]  # optional tools or MCP servers to integrate
  constraints: list[string]  # optional hard/soft/inferred constraints
outputs:
  agent_spec: file  # system prompt with frontmatter and body
  eval_notes: file  # tests, adversarial cases, and results
  integration: summary  # how to register and route the new agent
```

### Recursive Improvement
- Run recursive-improvement with failure cases and evaluation deltas; iterate until improvement delta < 2% or risks are documented.

### Examples
- Create a compliance-review agent with SOC2/GDPR guardrails and audit-ready outputs.
- Refine an existing API-tester agent to add contract testing and retry logic for flaky endpoints.

### Troubleshooting
- Ambiguous scope → rerun constraint extraction and confirm hard vs soft requirements.
- Overlapping agents → consult registry and propose consolidation or specialization.
- Tool misfires → add error-handling branches and safe defaults in the prompt body.

### Completion Verification
- [ ] Agent spec delivered with persona, contracts, and escalation rules.
- [ ] Tests and adversarial probes executed with recorded results.
- [ ] Confidence ceiling stated; memory/tags applied if MCP used.
- [ ] Registry metadata prepared for agent-selector.

Confidence: 0.70 (ceiling: inference 0.70) - SOP rewritten with Skill Forge structure and prompt-architect guardrails.
