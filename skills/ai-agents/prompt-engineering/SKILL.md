---
name: "prompt-engineering"
description: 'Write effective prompts for AI coding agents. Use when crafting system prompts, implementing chain-of-thought reasoning, building few-shot examples, adding guardrails, configuring tool use, or designing agentic prompt patterns. Covers CoT, few-shot, guardrails, and function calling.'
metadata:
  author: "AgentX"
  version: "1.0.0"
  created: "2025-01-15"
  updated: "2025-01-15"
compatibility:
  frameworks: ["agentx", "copilot", "openai", "anthropic"]
---

# Prompt Engineering

> **Purpose**: Write effective prompts for AI coding agents and workflows.  
> **Scope**: System prompts, reasoning patterns, guardrails, tool use, agentic workflows.

---

## When to Use This Skill

- Crafting system prompts for AI agents
- Implementing chain-of-thought reasoning
- Building few-shot prompt examples
- Adding content guardrails and safety filters
- Configuring tool/function calling patterns

## Prerequisites

- Understanding of LLM capabilities and limitations
- Access to an AI model endpoint

## Decision Tree

```
Writing a prompt?
├─ Simple, well-known task?
│   └─ Zero-shot (just instructions)
├─ Need specific output format?
│   └─ Few-shot (2-3 examples of input → output)
├─ Complex reasoning required?
│   ├─ Step-by-step? → Chain-of-thought ("think step by step")
│   └─ Multi-perspective? → Self-consistency (sample multiple paths)
├─ Agent / tool-use scenario?
│   ├─ Define tool schemas clearly
│   ├─ Add guardrails (what NOT to do)
│   └─ Include error recovery instructions
├─ System prompt for coding agent?
│   ├─ Role + constraints + format + examples
│   └─ Keep under 4K tokens for efficiency
└─ Prompt too long?
    └─ Progressive disclosure: load details on demand
```

## Quick Reference

| Pattern | When to Use | Token Cost |
|---------|-------------|------------|
| **Zero-Shot** | Simple tasks, well-known domains | Low |
| **Few-Shot** | Consistent output format needed | Medium |
| **Chain-of-Thought** | Multi-step reasoning, debugging | Medium |
| **ReAct** | Tool use, agentic workflows | High |
| **Reflection** | Self-correction, quality improvement | High |

---

## System Prompts

### Structure

Every system prompt should have four parts:

```
1. ROLE       → Who the AI is
2. CONTEXT    → What it knows about the situation
3. TASK       → What it should do
4. CONSTRAINTS → What it must NOT do
```

### Good Example

```text
You are a senior Python engineer reviewing pull requests.

CONTEXT:
- Project uses FastAPI + SQLAlchemy + pytest
- Code follows PEP 8 and uses type hints
- Test coverage target: 80%+

TASK:
Review the code changes and provide:
1. Security issues (critical)
2. Bug risks (high)
3. Style improvements (low)

CONSTRAINTS:
- Do NOT rewrite code, only point out issues
- Do NOT suggest changes outside the diff
- Rate each issue: critical / high / medium / low
```

### Anti-Patterns

| Don't | Do Instead |
|-------|------------|
| "Be helpful" | "You are a Python code reviewer" |
| "Do your best" | "List exactly 3 issues per file" |
| "Be careful" | "NEVER execute DELETE queries" |
| Long paragraphs | Bullet points and numbered lists |
| Vague instructions | Specific output format with examples |
| Inline prompt strings in code | Load from `prompts/{agent}.md` file |
| Inline output templates in code | Load from `templates/{name}.md` file |

---

## File-Based Prompt Management

> **RULE**: ALWAYS store prompts in separate files. NEVER embed multi-line prompts or output templates as string literals in code.

### Directory Convention

```
project/
  prompts/                    # System & agent prompts
    assistant.md              # One file per agent/role
    code-reviewer.md
    researcher.md
  templates/                  # Output format templates
    review-report.md          # Structured output templates
    analysis-summary.md
  config/
    models.yaml               # Model configuration
```

### Prompt File Format

```markdown
<!-- prompts/code-reviewer.md -->
<!-- Purpose: System prompt for code review agent -->
<!-- Model: gpt-5.1 | Max tokens: ~1500 -->

You are a senior Python engineer reviewing pull requests.

## Context
- Project uses FastAPI + SQLAlchemy + pytest
- Code follows PEP 8 and uses type hints
- Test coverage target: 80%+

## Task
Review the code changes and provide:
1. Security issues (critical)
2. Bug risks (high)
3. Style improvements (low)

## Constraints
- Do NOT rewrite code, only point out issues
- Do NOT suggest changes outside the diff
- Rate each issue: critical / high / medium / low
```

### Loading Pattern

```python
from pathlib import Path

# Load prompt from file
prompt = Path("prompts/code-reviewer.md").read_text(encoding="utf-8")

# Load output template and combine
template = Path("templates/review-report.md").read_text(encoding="utf-8")
full_prompt = f"{prompt}\n\n## Output Format\n{template}"
```

### Rules

- **MUST** store all prompts ≥2 lines in `prompts/` as `.md` files
- **MUST** store output format templates in `templates/` as `.md` files
- **MUST NOT** embed prompt text as multi-line strings in Python/C#/TS code
- **SHOULD** use Markdown format (readable, supports headers/lists)
- **SHOULD** name files after the agent role: `prompts/{role}.md`
- **SHOULD** include a comment header: purpose, target model, token estimate
- **MAY** use `{variable}` placeholders for runtime injection

### Why Separate Files?

| Benefit | Explanation |
|---------|-------------|
| **Version control** | Git diffs show exactly what changed in a prompt |
| **Non-dev editing** | PMs and prompt engineers edit without touching code |
| **A/B testing** | Swap prompt files without code changes |
| **Reuse** | Share prompts across agents, languages, and tests |
| **Separation of concerns** | Logic (code) vs. content (prompts) stay independent |

---

## Rules
1. All endpoints return ActionResult<T>
2. Use [Authorize] on all non-public endpoints
3. Validate input with FluentValidation
4. Return Problem() for errors (RFC 7807)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Prompt too long (>2000 words) | Split into system prompt + user prompt |
| No output format specified | Add "Respond in this format: ..." |
| Contradictory instructions | Review and remove conflicts |
| Assuming AI remembers context | Repeat key constraints in each message |
| Over-constraining | Allow flexibility for edge cases |
| No examples for complex formats | Add 2-3 few-shot examples |
| Mixing multiple tasks | One prompt = one task |

---

## Evaluation Checklist

Rate your prompt before using it:

- [ ] **Clear role**: Does the AI know who it is?
- [ ] **Specific task**: Is the desired output unambiguous?
- [ ] **Output format**: Will responses be consistent?
- [ ] **Constraints**: Are boundaries and safety rules defined?
- [ ] **Examples**: Are few-shot examples provided where needed?
- [ ] **Reasoning**: Is chain-of-thought requested for complex tasks?
- [ ] **Verification**: Does the prompt include self-check steps?
- [ ] **Stored externally**: Is the prompt in `prompts/` (not inline in code)?
- [ ] **Template separated**: Is the output template in `templates/` (not inline)?

---

## Resources

- [OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)
- [Anthropic Prompt Engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering)
- [Google Prompt Engineering](https://ai.google.dev/docs/prompt_best_practices)
- [AgentX Agent Definitions](../../../../.github/agents/)
- [AgentX Instruction Files](../../../../.github/instructions/)

---

**Related**: [AI Agent Development](../ai-agent-development/SKILL.md) for building agents • [Skills.md](../../../../Skills.md) for all skills

**Last Updated**: February 7, 2026


## Scripts

| Script | Purpose | Usage |
|--------|---------|-------|
| [`scaffold-prompt.py`](scripts/scaffold-prompt.py) | Generate structured prompt template (ROLE/CONTEXT/TASK/CONSTRAINTS) | `python scripts/scaffold-prompt.py --name code-reviewer [--pattern cot] [--with-examples 3]` |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Prompt too long / context exceeded | Reduce few-shot examples or split into sub-prompts |
| Model ignores instructions | Move critical rules to top of system prompt with explicit constraints |
| Inconsistent outputs | Add structured output format requirements and examples |

## References

- [Cot And Few Shot](references/cot-and-few-shot.md)
- [Guardrails And Tool Use](references/guardrails-and-tool-use.md)
- [Agentic Patterns](references/agentic-patterns.md)