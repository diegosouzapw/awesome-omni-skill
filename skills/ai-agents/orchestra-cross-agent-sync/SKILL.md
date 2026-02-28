---
name: orchestra-cross-agent-sync
description: Coordinate consistent behavior across heterogeneous agents (Claude, Copilot, Codex, Cursor, Windsurf, Gemini, Antigravity) using a shared source-of-truth model.
---

# Objective
Maintain parity of intent and task state across multiple agent ecosystems.

# Use This Skill When
- Creating or updating agent-specific instruction files in parallel.
- Mapping one registry/task model to multiple tool formats.
- Planning cross-agent handoff and fallback workflows.

# Procedure
1. Start from canonical project intent (phases, constraints, task state).
2. Render or author each agent file in its native style, not a generic format.
3. Keep skill and agent role naming consistent across ecosystems.
4. Verify that active tasks and conventions match after each update.
5. Record unsupported capabilities and provide explicit manual fallback.

# Guardrails
- Do not let one platform drift from canonical phase definitions.
- Do not assume feature parity for subagents/skills/config formats.
- Keep migration notes explicit where docs are incomplete.

# Done Criteria
- All supported platforms receive aligned instructions.
- Platform-specific limits are documented and handled safely.
- Cross-agent handoff remains deterministic and auditable.
