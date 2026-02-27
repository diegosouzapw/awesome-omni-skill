---
name: Define Product
description: Create user-centric product definitions by analyzing the repository, inferring product goals and user needs, clarifying ambiguities through structured questioning, and generating comprehensive product.md documentation. Use when the user mentions "product definition", "define product", "product strategy", or needs to document user personas, pain points, and user journeys.
---

# Define Product Skill

@.claude/shared-imports/CoD_Σ.md
@.claude/shared-imports/project-intel-mjs-guide.md
@.claude/shared-imports/memory-utils.md

## Workflow Context

**SDD Phase**: Phase 1 (Foundation)
**Command**: `/define-product`
**Prerequisites**: Repository with code or documentation
**Creates**: `memory/product.md`
**Successor**: Phase 2 - `/generate-constitution` → `memory/constitution.md`

### Phase Chain
```
Phase 1: /define-product → memory/product.md (YOU ARE HERE)
Phase 2: /generate-constitution → memory/constitution.md
Phase 3: /feature → specs/$FEATURE/spec.md
Phase 4: /clarify (if needed) → updated spec.md
Phase 5: /plan → plan.md + research.md + data-model.md
Phase 6: /tasks (auto) → tasks.md
Phase 7: /audit (auto) → audit-report.md
Phase 8: /implement → code + tests + verification
```

**$FEATURE format**: `NNN-feature-name` (e.g., `001-therapy-app`)

---

## Overview

This skill generates user-centric product definitions (`product.md`) by combining intelligence-first repository analysis with structured user clarification.

**Critical Boundary**: product.md must be PURELY user-centric with NO technical implementation details.

---

## Workflow

### Step 1: Intelligence Gathering

Run intelligence queries to understand the codebase:

```bash
# Project overview
project-intel.mjs --overview --json

# Search for product signals
project-intel.mjs --search "README" --json
project-intel.mjs --docs "product" --json
```

Analyze:
- README files and documentation
- Package metadata (package.json, cargo.toml, etc.)
- Code structure patterns (auth, org management, social features)
- UI patterns (large fonts, high contrast → accessibility needs)

### Step 2: Infer Product Characteristics

Based on intelligence, infer:

**Product Type**:
- B2B signals: SSO, org/team management, audit logs, RBAC, subscription billing
- B2C signals: Social auth, personal profiles, gamification, push notifications

**Primary Users**:
- Developers (API docs, SDKs, developer tools)
- Marketing teams (campaigns, analytics, email)
- Elderly/accessibility (large fonts, simplified UI, reminders)
- Executives (dashboards, reports, ROI metrics)

**Core Problem**:
- Data aggregation (scattered info across tools)
- Workflow automation (manual, repetitive tasks)
- Accessibility (complex tasks for users with specific needs)
- Communication (team collaboration pain points)

### Step 3: Clarify Ambiguities (Max 5 Questions)

If signals are conflicting or unclear, use `AskUserQuestion` tool:

**Question Priority**:
1. Product type (if B2B vs B2C ambiguous)
2. Primary user persona (who matters most?)
3. Core problem being solved
4. Key differentiator ("our thing")
5. Most critical pain point

**Question Format**:
```python
AskUserQuestion(questions=[{
  "question": "Is this product primarily B2B or B2C?",
  "header": "Product Type",
  "multiSelect": False,
  "options": [
    {
      "label": "B2B (Business-to-Business)",
      "description": f"Enterprise/team product. Evidence: {b2b_signals}"
    },
    {
      "label": "B2C (Business-to-Consumer)",
      "description": f"Consumer-facing. Evidence: {b2c_signals}"
    }
  ]
}])
```

### Step 4: Define 3 Personas (Jobs-to-be-Done)

For each persona, gather:

**Demographics**: Age range, location, tech savviness, accessibility needs

**Pain Points** (JTBD Framework):
```markdown
**Pain #: [Short title]**
- **Pain**: [Specific frustration - be concrete]
- **Why it hurts**: [Impact: time lost, money wasted, stress caused]
- **Current workaround**: [How they cope today - tools, hacks, manual work]
- **Frequency**: [Daily, weekly, monthly]
```

**Pain Resolution Mapping**:
```
Pain 1 → [Our Solution Feature] → [Measurable Outcome]
```

### Step 5: Map 2-3 User Journeys

Using CoD^Σ notation (from @.claude/shared-imports/CoD_Σ.md):

**Standard Journey**:
```
Awareness ≫ Interest ≫ Research → Decision ≫ Onboarding → First Use ≫ First Value ∘ Habit
```

For each step, define:
- User action or system response
- Information/guidance needed
- Pain point addressed
- Success indicator

### Step 6: Generate product.md

Use template at `@.claude/templates/product.md`.

Include:
- Product overview and value proposition
- 3 personas with demographics, psychographics, pain points
- Pain-to-resolution mapping
- 2-3 user journeys with CoD^Σ notation
- Journey-to-pain mapping
- "Our Thing" (key differentiator)
- North Star Metric

### Step 7: Validate

**CRITICAL - Check for violations**:

❌ **MUST NOT Contain**:
- Tech stack: "React", "Python", "PostgreSQL", "AWS"
- Architecture: "Microservices", "REST API", "GraphQL"
- Frameworks: "Next.js", "FastAPI", "Django"
- Infrastructure: "Kubernetes", "Docker", "Lambda"

✓ **MUST Contain**:
- 3 personas with complete JTBD pain points
- Pain-to-resolution mapping with measurable outcomes
- 2-3 user journeys with CoD^Σ notation
- "Our Thing" clearly articulated
- North Star Metric (quantifiable user outcome)

**Key Anti-Patterns**:

| ❌ Wrong (Technical) | ✓ Right (User-Centric) |
|---------------------|------------------------|
| "We'll use React for fast UI" | "Users need responsive, lag-free interface" |
| "PostgreSQL for data integrity" | "Users need reliable, accurate data they can trust" |
| "Microservices for scale" | "Users need instant search across millions of items" |
| "OAuth 2.0 authentication" | "Users need to log in with company credentials" |

---

## Example

See complete B2B SaaS example:
- [examples/b2b-saas-product.md](examples/b2b-saas-product.md)

This shows:
- Full persona definitions with JTBD pain points
- Complete pain-to-resolution mapping
- User journeys with CoD^Σ notation
- NO technical decisions

---

## Next Steps

After product.md is created, use `/generate-constitution` to derive technical principles FROM the user needs documented here.

---

## Key Reminders

1. **Intelligence FIRST** - Use `@.claude/shared-imports/project-intel-mjs-guide.md` patterns
2. **User-Centric ONLY** - NO tech stack, architecture, or implementation
3. **Evidence-Based** - Every claim traces to intelligence query or user input
4. **CoD^Σ Journeys** - Use `@.claude/shared-imports/CoD_Σ.md` notation
5. **Validate Boundary** - Verify no technical decisions leaked in

---

## Prerequisites

Before using this skill:
- ✅ Project repository exists with some code or documentation
- ✅ project-intel.mjs exists and is executable
- ✅ PROJECT_INDEX.json exists (run `/index` if missing)
- ✅ AskUserQuestion tool available (for clarification)
- ⚠️ Optional: README.md or docs/ with product information
- ⚠️ Optional: Existing product.md to enhance (will be rewritten)

## Dependencies

**Depends On**:
- project-intel.mjs - For intelligence-first codebase analysis
- @.claude/shared-imports/CoD_Σ.md - For user journey notation
- @.claude/shared-imports/project-intel-mjs-guide.md - For intelligence query patterns
- AskUserQuestion tool - For structured user clarification

**Integrates With**:
- **generate-constitution skill** - Uses product.md to derive technical constitution
- **/define-product command** - User-facing command that invokes this skill

**Tool Dependencies**:
- project-intel.mjs (codebase signals analysis)
- AskUserQuestion tool (max 5 questions for clarification)
- Read tool (to analyze existing docs)
- Write tool (to create product.md)

## Next Steps

After product.md creation completes, typical progression:

**Constitution Derivation Flow**:
```
define-product (creates product.md)
    ↓ (manual invocation)
generate-constitution (user runs /generate-constitution)
    ↓
constitution.md created (technical principles FROM user needs)
    ↓
Ready for feature development with constitutional guidance
```

**Direct to Feature Development** (if constitution exists):
```
define-product (creates/updates product.md)
    ↓
User has clearer product understanding
    ↓
specify-feature skill (references product.md personas and pain points)
```

**User Action Required**:
- Review product.md for accuracy and completeness
- Answer clarification questions (max 5) if ambiguities detected
- Run `/generate-constitution` to derive technical constitution from product definition
- Share product.md with team for alignment

**Outputs Modified**:
- `memory/product.md` - User-centric product definition in memory/ documentation
- NO technical documents created by this skill

**Commands**:
- **/generate-constitution** - After product.md created, derive technical constitution
- **/define-product** - User-facing command to create/update product definition

## Failure Modes

### Common Failures & Solutions

**1. Technical implementation leaks into product.md**
- **Symptom**: product.md mentions "React", "API", "database", "microservices"
- **Solution**: Remove ALL technical terms, rephrase in user-centric language
- **Enforcement**: Validation step (Step 7) MUST catch these violations
- **Prevention**: Constantly check "Would a non-technical user understand this?"

**2. Personas lack concrete pain points**
- **Symptom**: Personas have vague descriptions like "wants efficiency", "needs reliability"
- **Solution**: Use JTBD framework with specific pain-why-workaround-frequency
- **Pattern**: "Sarah spends 4 hours/week copying data between spreadsheets (pain), causing billing errors (why), using manual copy-paste (workaround), every Friday (frequency)"
- **Enforcement**: Each persona MUST have 3-5 concrete pain points with all 4 fields

**3. No intelligence queries before assumptions**
- **Symptom**: product.md created without running project-intel.mjs queries
- **Solution**: ALWAYS run intelligence queries first (Step 1)
- **Article I**: Intelligence-First Principle requires intel queries before assumptions
- **Prevention**: Skill workflow enforces Step 1 before Step 2

**4. More than 5 clarification questions asked**
- **Symptom**: Asking 10+ questions, overwhelming the user
- **Solution**: Prioritize top 5 questions by impact (product type, primary persona, core problem)
- **Enforcement**: Max 5 questions per iteration (skill constraint)
- **Prevention**: Use intelligence queries to reduce ambiguity before asking

**5. User journeys missing CoD^Σ notation**
- **Symptom**: User journeys described in plain text without operators
- **Solution**: Use CoD^Σ operators: ≫ (transformation), → (delegation), ∘ (sequential)
- **Template**: "Awareness ≫ Interest → Decision ≫ Onboarding ∘ First Value"
- **Article II**: Evidence-Based Reasoning requires structured notation

**6. Pain-to-resolution mapping missing measurable outcomes**
- **Symptom**: "Pain 1 → Our Feature" without quantifiable result
- **Solution**: Add measurable outcome: "Pain 1 → Our Feature → Saves 4 hours/week"
- **Pattern**: Every pain must map to feature AND measurable outcome
- **Enforcement**: Validation step checks all pains have resolutions

**7. No North Star Metric defined**
- **Symptom**: product.md lacks single quantifiable user outcome metric
- **Solution**: Define ONE metric that indicates user success
- **Examples**: "Active weekly users", "Time saved per user", "Tasks completed"
- **Not North Star**: Revenue, signups (business metrics, not user outcomes)

**8. "Our Thing" (differentiator) is generic**
- **Symptom**: "Best", "fastest", "easiest" without specificity
- **Solution**: Identify unique capability from intelligence analysis
- **Pattern**: "Only product that [specific capability] for [specific user]"
- **Prevention**: Reference codebase signals that show unique features

**9. Personas are too similar or too broad**
- **Symptom**: All 3 personas have overlapping pain points
- **Solution**: Ensure distinct personas with unique needs
- **Pattern**: Different demographics, different pain points, different outcomes
- **Enforcement**: Each persona should have 2-3 unique pain points

**10. User journey steps lack success indicators**
- **Symptom**: Journey shows steps but not how to know they're complete
- **Solution**: Add success indicator to each step
- **Example**: "Onboarding → [user creates first project] ← Success: project exists"

**11. Product overview is feature list, not value proposition**
- **Symptom**: "Features: X, Y, Z" instead of user outcome
- **Solution**: Lead with user problem solved, then how
- **Pattern**: "For [persona] who [pain], [product] is a [category] that [outcome]. Unlike [alternatives], [our thing]."

**12. Intelligence signals ignored or cherry-picked**
- **Symptom**: Conclusion contradicts codebase signals
- **Solution**: Reconcile through AskUserQuestion if signals conflict
- **Article II**: Evidence-Based Reasoning requires CoD^Σ traces from intelligence
- **Prevention**: Document all intelligence findings, trace decisions

## Related Skills & Commands

**Direct Integration**:
- **generate-constitution skill** - Uses product.md to derive technical constitution (successor)
- **/define-product command** - User-facing command that invokes this skill

**Workflow Context**:
- Position: **Foundation** of product development (before constitution, before features)
- Triggers: User mentions "define product", "product strategy", "user personas"
- Output: product.md with user-centric definition (NO technical details)

**Constitution Chain**:
```
User Context (this skill: product.md)
    ↓
Technical Principles (generate-constitution: constitution.md)
    ↓
Feature Specifications (specify-feature: spec.md)
    ↓
Implementation Plans (create-implementation-plan: plan.md)
```

**Core Principle**: Product definition is PURELY user-centric. Technical decisions come AFTER user needs are documented.

**Quality Gates**:
- **Critical Boundary**: NO technical implementation details in product.md
- **JTBD Framework**: All personas have concrete pain points with 4 fields
- **Max 5 Questions**: Clarification limited to 5 questions per iteration
- **CoD^Σ Notation**: User journeys use structured operators

**Integrations**:
- **specify-feature skill** - References product.md personas and pain points in spec.md
- **Intelligence queries** - Uses project-intel.mjs for codebase signal analysis
- **AskUserQuestion tool** - For structured clarification dialogue

**Workflow Recommendation**: Run this skill FIRST in a new project to establish user-centric foundation before any technical decisions.

---

**Version:** 1.1
**Last Updated:** 2025-01-19
**Owner:** Claude Code Intelligence Toolkit
