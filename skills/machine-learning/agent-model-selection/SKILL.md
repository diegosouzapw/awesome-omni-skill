---
name: agent-model-selection
description: Guidelines for selecting appropriate AI model (Sonnet vs Haiku) based on task complexity, ensuring cost efficiency while maintaining quality. Use when assigning work.
---

# Agent Model Selection

## Instructions

### Core decision

**Sonnet:** Complex reasoning, architecture, security (2+ criteria)
**Haiku:** Defined rules, repetitive tasks, simple commands (~95% cheaper)

### Selection criteria

**Use Sonnet if 2+ apply:**
1. Logical reasoning and trade-off analysis
2. Architecture/design decisions
3. Semantic/intent analysis
4. Problem diagnosis and strategy
5. Multi-component interaction
6. Security/performance analysis

**Use Haiku if dominant:**
1. Following defined rules/templates
2. Repetitive mechanical tasks
3. Command execution and collection
4. Simple CRUD operations
5. Format validation

### Decision flowchart

```
Architecture/design? → YES → Sonnet
Multiple options? → YES → Sonnet
Security/performance? → YES → Sonnet
Defined rules only? → YES → Haiku
Detailed guide? → YES → Haiku
Large delegated? → YES → Sonnet
Simple commands? → YES → Haiku
Default: Sonnet (quality first)
```

## Example

```markdown
Task: Add validation logic
→ Analysis: Complex rules + security + error handling
→ Decision: Sonnet (3 criteria met)

Task: Add tags to files
→ Analysis: Template exists, repetitive
→ Decision: Haiku (rule-following)
```

---

**For detailed criteria, see [reference.md](reference.md)**
**For more examples, see [examples.md](examples.md)**
