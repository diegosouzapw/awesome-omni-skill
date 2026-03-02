---
name: architecting-innovation-agents
description: Propose multi-agent and workflow architectures for CustomGPT.ai Labs projects, showing how Claude Code, CustomGPT.ai, and supporting tools interact to deliver the desired business outcome.
---

# Architecting Innovation Agents

You turn an Innovation PRD into a **high‑level agent and system architecture**
suitable for a design review.

## When to Use

Use this skill when the user:

- Needs a technical approach for an Innovation project.
- Is deciding between simple RAG vs. multi‑agent workflows.
- Wants to understand how CustomGPT.ai, Claude Code, and other services
  should work together.

## Inputs

Expect:

- The project PRD or equivalent description.
- Any explicit technical constraints (hosting, auth model, data residency,
  must‑use components).
- Notes on existing components (CustomGPT.ai chat widget, AI call center,
  CRMs, data warehouses, etc.).

## Architecture Output

Produce a Markdown document with:

1. **Overview** – one short paragraph summarizing the architecture choice.
2. **Agents and Components** – a numbered list where each item has:
   - Name and role.
   - Responsibilities.
   - Inputs and outputs.
3. **Data & Control Flow** – step‑by‑step description of how a typical
   request flows through the system.
4. **Context & Memory** – how RAG sources, metadata, and history are loaded
   and updated.
5. **Safety & Compliance** – where security, policy enforcement, and human
   overrides sit in the flow.
6. **Implementation Notes** – what should be implemented via CustomGPT.ai
   config, Claude Code automation, or traditional backend code.

If the user asks, also include a simple ASCII or Mermaid diagram of the flow.

## Guidelines

- Prefer the simplest architecture that can support the experiment or V0
  within **2–4 weeks** of effort.
- Make tradeoffs explicit (quality vs. latency, flexibility vs. complexity).
- Call out assumptions that engineering must validate.
