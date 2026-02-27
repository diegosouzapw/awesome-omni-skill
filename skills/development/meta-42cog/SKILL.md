---
name: meta-42cog
description: This skill should be used when initializing a new project with Cognitive Agile methodology. It automatically generates real.md (reality constraints) and cog.md (cognitive model) by scanning the project directory and identifying key patterns.
---

# Meta Skill - Cognitive Agile Foundation Generator

## Overview

This is the **meta skill** for Cognitive Agile methodology (认知敏捷法). It generates the two foundational documents that all other skills depend on:

1. **real.md** - Reality Constraints Document
2. **cog.md** - Cognitive Model Document

These documents follow the RCSW workflow:
```
Real → Cog → Spec → Work
```

## When to Use This Skill

- Starting a new project with Cognitive Agile methodology
- When real.md or cog.md is missing and another skill needs them
- When project structure has significantly changed and foundation documents need regeneration
- When onboarding a project to Cognitive Agile workflow

## Core Principles

This skill follows the **Supreme Principle** of Cognitive Agile:

> **Accelerate the Hybrid Intelligence Loop** — All designs should be evaluated by whether they accelerate the human-AI collaboration cycle.

### Four Specific Principles

1. **Independent Operation**: Let AI work autonomously
2. **Focus on Output**: Care about final results, not intermediate steps
3. **Consider Exceptions**: Handle cases AI might not anticipate
4. **Continuous Reflection**: Generate more skills from experience

## Process

### Phase 1: Project Scanning

Scan the project directory to identify:

**Technical Stack Detection:**
- `package.json` → Node.js/JavaScript project, dependencies
- `requirements.txt` / `pyproject.toml` → Python project
- `Cargo.toml` → Rust project
- `go.mod` → Go project
- `*.csproj` → .NET project

**Framework Detection:**
- Next.js, React, Vue, Angular (frontend)
- Express, FastAPI, Django, Rails (backend)
- Drizzle, Prisma, TypeORM (ORM)

**Database Detection:**
- Schema files, migration files
- Connection strings in environment files

**Sensitive Data Patterns:**
- User credentials (passwords, tokens)
- API keys
- Personal information (email, phone, address)
- Payment information

### Phase 2: Generate real.md

> **Recommendation**: Most projects should use the **simple format**. Only use the detailed format when you need to distinguish required/optional constraints or provide detailed explanations.

**Template Files**:
- Simple format (recommended): [real-template-simple.md](./template/real-template-simple.md)
- Detailed format: [real-template-detailed.md](./template/real-template-detailed.md)

**Example Files**:
- Simple format examples: [real-example-simple.md](./example/real-example-simple.md)
- Detailed format example: [real-example-detailed.md](./example/real-example-detailed.md)

<real-md-template>

**Format**: Markdown + XML semantic closure tags

**Structure**:
```markdown
# [Project Name] - Reality Constraints Document

<meta>
  <document-id>[project]-real</document-id>
  <version>1.0.0</version>
  <project>[Project Name]</project>
  <type>Reality Constraints</type>
  <created>[Date]</created>
</meta>

## Document Purpose

[Brief description of what this document defines]

<constraints>

## Required Constraints (Maximum 4)

<constraint required="true" id="C1">
<title>[Constraint Title]</title>
<description>[What must be done or avoided]</description>
<rationale>[Why this constraint exists]</rationale>
<violation-consequence>[What happens if violated]</violation-consequence>
</constraint>

[... up to 4 required constraints]

## Optional Constraints (Maximum 3)

<constraint required="false" id="C5">
<title>[Constraint Title]</title>
<description>[What should be done or avoided]</description>
<rationale>[Why this is recommended]</rationale>
</constraint>

[... up to 3 optional constraints]

</constraints>

## Technical Environment

<environment>
<stack>
  [Technology stack details]
</stack>
</environment>

## Constraint Checklist

[Checkboxes for verification]
```

</real-md-template>

**Constraint Identification Guidelines:**

| Priority | Type | Examples |
|----------|------|----------|
| Required | Security | Password hashing, API key encryption, data ownership validation |
| Required | Data Integrity | First-user-admin rule, unique constraints |
| Required | Compliance | GDPR, data residency, audit logging |
| Optional | UX Simplification | Avatar generation, file type limits |
| Optional | Performance | Caching rules, rate limits |

**Key Rule**: Focus on constraints that **AI might not anticipate** but would cause **real-world damage** if violated.

### Phase 3: Generate cog.md

> **Recommendation**: Most projects should use the **simple format**. Only use the detailed format when you need more detailed attribute definitions and complex relationships.

**Template Files**:
- Simple format (recommended): [cog-template-simple.md](./template/cog-template-simple.md)
- Detailed format: [cog-template-detailed.md](./template/cog-template-detailed.md)

**Example Files**:
- Simple format example: [cog-example-simple.md](./example/cog-example-simple.md)
- Detailed format example: [cog-example-detailed.md](./example/cog-example-detailed.md)

<cog-md-template>

**Format**: Markdown + XML semantic closure tags

**Core Framework**: **Agents + Information + Context**

**Structure**:
```markdown
# [Project Name] - Cognitive Model Document

<meta>
  <document-id>[project]-cog</document-id>
  <version>1.0.0</version>
  <project>[Project Name]</project>
  <type>Cognitive Model</type>
  <created>[Date]</created>
  <depends>real.md</depends>
</meta>

## Document Purpose

[Brief description based on "Agents + Information + Context" framework]

---

## 1. Agents

<agents>

### 1.1 Human Agents

<agent type="human" id="A1">
<name>[Agent Name]</name>
<identifier>[How to uniquely identify - UUID, email, etc.]</identifier>
<classification>
  <by-[criterion]>[Category 1] | [Category 2]</by-[criterion]>
</classification>
<capabilities>[What they can do]</capabilities>
<goals>[What they want to achieve]</goals>
</agent>

### 1.2 AI Agents

<agent type="ai" id="A2">
<name>[AI Agent Name]</name>
<identifier>[How to identify - provider + model]</identifier>
<classification>
  <by-[criterion]>[Categories]</by-[criterion]>
</classification>
<interaction-pattern>[Input/Output patterns]</interaction-pattern>
</agent>

</agents>

---

## 2. Information

<information>

### 2.1 Core Entities

<entity id="E1">
<name>[Entity Name]</name>
<unique-code>[How to uniquely identify]</unique-code>
<classification>
  <by-[criterion]>[Categories]</by-[criterion]>
</classification>
<attributes>[Key attributes]</attributes>
<relations>[Relationships: 1:1, 1:N, N:N]</relations>
</entity>

### 2.2 Information Flow

<information-flow>
<flow id="F1" name="[Flow Name]">
  [Agent] → [Action] → [System] → [Response] → [Agent]
</flow>
</information-flow>

</information>

---

## 3. Context

<context>

### 3.1 Application Context
[Web app, mobile app, CLI tool, etc.]

### 3.2 Technical Context
[Architecture, protocols, security measures]

### 3.3 User Experience Context
[Emotional goals, interaction style]

</context>

---

## 4. Weight Matrix

<weights>
[Importance weights for entities and interactions]
</weights>

---

## 5. Verification Checklist

[Checkboxes for validation]
```

</cog-md-template>

**Entity Identification Guidelines:**

For each entity, define:
1. **Unique Code**: How AI can locate and identify it (UUID, slug, compound key)
2. **Classification**: Human-defined categories (AI tends to classify arbitrarily)

### Phase 4: Validation

After generating both documents:

1. **Cross-reference Check**:
   - All entities in cog.md should respect constraints in real.md
   - Security-sensitive entities should have corresponding constraints

2. **Completeness Check**:
   - real.md: 4-7 constraints total
   - cog.md: All major entities identified with unique codes and classifications

3. **Format Check**:
   - XML tags are properly closed
   - Markdown structure is clean
   - No code fences around constraint/entity definitions

## Output

Generate two files in the project's cognitive agile directory:

```
.42cog/           (or project-specific location)
├── real.md       # Reality Constraints
└── cog.md        # Cognitive Model
```

## Quality Checklist

- [ ] real.md has 4 required constraints maximum
- [ ] real.md has 3 optional constraints maximum
- [ ] All constraints focus on AI-unaware, real-world-damaging issues
- [ ] cog.md follows "Agents + Information + Context" framework
- [ ] All entities have unique codes defined
- [ ] All entities have human-defined classifications
- [ ] XML semantic closure tags are properly used
- [ ] Documents are concise (AI context window is limited)

## Integration with Other Skills

| Skill | Relationship |
|-------|--------------|
| All 01-11 skills | Output: real.md and cog.md are prerequisites |
| product-requirements | Depends on cog.md for entity understanding |
| database-design | Depends on cog.md for entity relationships |
| coding | Depends on real.md for security constraints |

## Trigger Conditions

This skill is automatically invoked when:
1. Any other skill detects missing real.md or cog.md
2. User explicitly requests foundation document generation
3. Project is being initialized with Cognitive Agile methodology
