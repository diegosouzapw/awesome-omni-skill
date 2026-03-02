---
name: architecture-decisioning
description: 'Guides the selection and documentation of architectural decisions using ADR patterns.'
metadata:
  id: ce.skill.architecture-decisioning
  tags: [architecture, planning, context-min]
  inputs:
    files: [ARCHITECTURE.md]
    concepts: [tradeoffs]
    tools: [toolset:read]
  outputs:
    artifacts: []
    files: []
    actions: [propose-decision]
  dependsOn:
    artifacts: [ce.doc.architecture]
    files: [ARCHITECTURE.md]
  related:
    artifacts: [ce.prompt.propose-adr]
    files: []
---

# Architecture Decisioning Skill

When a significant design choice must be made, use this skill to weigh options and document the
decision clearly.

## Steps

1. **State the context.** Summarise the problem or requirement driving the decision. Reference
   relevant sections of `ARCHITECTURE.md` and other docs that impose constraints.

2. **Identify options.** List the viable alternatives. For each, describe the approach,
   including technologies, patterns and how it satisfies the requirements.

3. **Evaluate trade‑offs.** Compare the options against criteria such as complexity, performance,
   scalability, security, maintainability and alignment with existing architecture. Note
   pros and cons.

4. **Recommend a decision.** Select the option that best meets the criteria. Explain why it is
   preferred and address why other options were rejected.

5. **Document the decision.** Create or update an Architecture Decision Record (ADR) in a
   dedicated directory (e.g. `docs/adr/`). Include context, decision, consequences and links
   to relevant discussions. Ensure the ADR is referenced in `ARCHITECTURE.md`.

6. **Communicate and review.** Share the ADR with stakeholders for feedback. Incorporate
   suggestions and finalise. Make sure the decision is reflected in subsequent plans and
   implementations.

Thoroughly documented decisions foster transparency and ease future maintenance or revisiting
when requirements change.
