---
name: project-domain
description: Capture subject matter expertise and domain knowledge that will inform all product decisions - the ubiquitous language, domain rules, and principles from the field
disable-model-invocation: false
---

---
description: Capture subject matter expertise and domain knowledge that will inform all product decisions - the ubiquitous language, domain rules, and principles from the field.
---

The user input to you can be provided directly by the agent or as a command argument - you **MUST** consider it before proceeding with the prompt (if not empty).

User input:

$ARGUMENTS

Create or update the domain model that captures subject matter expertise for this project.

## Why Domain Knowledge Matters

Many products operate in specialized domains (fitness, healthcare, finance, education, etc.) where:
- **Terminology matters**: Using wrong terms confuses users and violates industry norms
- **Domain rules exist**: Physical, scientific, or regulatory constraints that can't be ignored
- **Expertise is required**: Decisions must be informed by domain knowledge, not guesses
- **User trust depends on accuracy**: Getting domain details wrong destroys credibility

This command captures that knowledge **early** so it informs all downstream decisions.

## Project Identification

First, determine the project:
- Parse arguments for project identifier
- If not provided, ask: "Which project should I create/update domain model for?"
- List available projects in `specs/projects/`

Set `PROJECT_DIR` = `specs/projects/[PROJECT_ID]`

## Context Loading

Load existing project artifacts:
```
REQUIRED:
- {PROJECT_DIR}/project.md → Project vision, domain type, users

OPTIONAL (if exist):
- {PROJECT_DIR}/domain-model.md → Existing domain model (update mode)
- {PROJECT_DIR}/project-import.md → Brownfield import notes
- {PROJECT_DIR}/project-landscape-overview.md → Brownfield code scan
```

**Detect mode**:
- If `domain-model.md` exists: **UPDATE mode** - enhance existing model
- If not: **CREATE mode** - build from scratch

## Just-In-Time Research

**Reference**: Follow the just-in-time research pattern (`.cursor/skills/just-in-time-research/SKILL.md`)

### Research Strategy

For domain model creation, research is **critical**. Identify knowledge gaps and conduct research:

**1. Domain Terminology**:
- Decision: What are the canonical terms in [domain]?
- Web Search: "[domain] glossary", "[domain] terminology standards"
- Deep Research (if specialized): Industry-specific ontologies, taxonomies

**2. Domain Principles**:
- Decision: What scientific/industry principles govern [domain]?
- Web Search: "[domain] best practices", "[domain] principles", "[field] science"
- Deep Research (if needed): Academic research, industry standards

**3. Domain Rules & Constraints**:
- Decision: What rules must always hold true in [domain]?
- Web Search: "[domain] safety guidelines", "[domain] regulations"
- Deep Research (if safety-critical): Regulatory requirements, certification standards

**4. Domain Entities**:
- Decision: What are the core concepts/entities in [domain]?
- Web Search: "[domain] data model", "[domain] ontology"
- Context7 (if applicable): Domain-specific libraries with established models

### Research Execution

Use available MCP tools in this priority order:
1. **Perplexity MCP** (preferred): `perplexity_search`, `perplexity_ask`, `perplexity_research`
2. **Web Search** (fallback): `web_search` tool
3. **User Input**: Ask user for SME knowledge they have

**Embed all findings** directly into the domain model sections with source citations.

## Interactive Domain Elicitation

### Step 1: Domain Identification

Ask the user:

> "Let's capture the domain expertise for your project. I'll ask you questions to understand the specialized knowledge that should inform this product.
>
> **What domain/field does this product operate in?**
> (e.g., Exercise Science, Healthcare, Financial Services, Education, Agriculture, etc.)
>
> **What makes this domain specialized?** Why does someone need expertise to build this correctly?"

Wait for response. Use the domain identification to guide research.

### Step 2: Terminology Elicitation

> "Now let's establish the **canonical terminology** for this domain.
>
> **What are the most important terms/concepts users and the product must use?**
> For each term, provide:
> - The precise definition
> - Any synonyms we should AVOID (to maintain consistency)
>
> I'll also research standard terminology in [domain] to ensure we're aligned with industry norms."

Conduct terminology research and merge with user input.

### Step 3: Core Concepts & Entities

> "What are the **core concepts or entities** in this domain?
>
> Think about:
> - What 'things' does the user work with? (e.g., Workouts, Exercises, Muscle Groups)
> - How do these relate to each other?
> - What attributes define each concept?
>
> I'll help structure these into a domain model."

Build entity definitions and relationships.

### Step 4: Domain Rules & Invariants

> "What **rules must always be true** in this domain?
>
> Examples:
> - 'A workout cannot have zero exercises'
> - 'Rest periods must be positive numbers'
> - 'A muscle group cannot be trained on consecutive days' (if that's a principle you follow)
>
> These become validation rules and system invariants."

### Step 5: Domain Principles

> "What **scientific or industry principles** should guide this product?
>
> For a fitness app, this might include:
> - Progressive overload principle
> - Muscle recovery requirements
> - Volume-intensity relationship
>
> For each principle, I'll research the evidence base and document how it applies to our product."

Conduct principle research and document with sources.

### Step 6: Constraints & Safety

> "What is **impossible, unsafe, or inadvisable** in this domain?
>
> - Physical constraints (can't do negative reps)
> - Safety constraints (max heart rate limits)
> - Regulatory constraints (medical advice disclaimers)
>
> These become hard boundaries the product must respect."

## Domain Model Generation

After elicitation and research, generate `{PROJECT_DIR}/domain-model.md` using:
- Template: `.speck/templates/project/domain-model-template.md`
- User-provided domain expertise
- Research findings with citations
- Structured entity relationships

### Quality Checks

Before finalizing, verify:

**Completeness**:
- [ ] All user-provided terms captured in glossary
- [ ] Core entities have attributes and relationships
- [ ] Key invariants documented
- [ ] At least 2-3 foundational principles with sources
- [ ] Constraints clearly stated

**Consistency**:
- [ ] Terminology matches project.md and other artifacts
- [ ] No conflicting rules
- [ ] Entity relationships are bidirectional

**Actionability**:
- [ ] Glossary terms can be used in code (valid identifiers)
- [ ] Rules are specific enough to validate
- [ ] Principles provide clear guidance

## Output

Create/update: `{PROJECT_DIR}/domain-model.md`

**Summary to user**:
```
✅ Domain model created/updated for [PROJECT_NAME]

Captured:
- [X] terms in glossary
- [X] core entities with relationships
- [X] domain invariants
- [X] foundational principles
- [X] constraints

Research sources:
- [List key sources used]

This domain model will be referenced by:
- /project-plan (epic structure informed by domain)
- /epic-specify (feature scope uses domain terms)
- /story-specify (user stories use glossary)
- /story-plan (data model validates against domain)
- /story-validate (checks domain invariants)

Next steps:
1. Review domain-model.md for accuracy
2. Add any missing domain expertise
3. Continue with /project-context or /project-plan
```

## Integration Notes

**For brownfield projects**:
- Extract domain terminology from existing codebase (variable names, comments, UI text)
- Identify implicit domain rules in existing validation logic
- Document the "as-is" domain model before proposing changes

**For recipe-based projects**:
- Some recipes may have domain considerations (e.g., SaaS billing → subscription domain)
- Layer product-specific domain on top of recipe patterns

## When to Run This Command

**Recommended position in flow**:
```
/project-specify → /project-clarify → /project-domain → /project-context → /project-architecture → /project-plan
```

**Why here?**
- After specify/clarify: We know what we're building
- Before context: Domain constraints inform technical constraints
- Before architecture: Domain entities inform data architecture
- Before plan: Domain structure informs epic organization

**Can skip if**:
- Product has no specialized domain (generic CRUD app)
- Domain is purely technical (developer tools)
- Team already has domain documented elsewhere (link to it)
