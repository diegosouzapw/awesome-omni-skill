---
name: adr-creator
description: Create Architecture Decision Records (ADRs) documenting significant technical decisions for the FF Analytics platform. This skill should be used when making architectural choices, evaluating alternatives for data models or infrastructure, documenting trade-offs, or when the user asks "should we use X or Y approach?" Guides through the ADR creation workflow from context gathering to documentation.
---

# ADR Creator

Create Architecture Decision Records (ADRs) that document significant technical decisions for the Fantasy Football Analytics project following the established ADR format and workflow.

## When to Use This Skill

Use this skill proactively when:

- Making architectural decisions (data model, infrastructure, tooling)
- User asks "should we use X or Y?" or "what's the best approach for Z?"
- Evaluating alternatives with trade-offs
- Documenting decisions that will impact future development
- Creating records referenced by strategic-planner skill
- Superseding or updating previous ADRs

## ADR Purpose

ADRs serve as **permanent records of significant technical decisions**, capturing:

1. **Context** - Why the decision was needed
2. **Alternatives** - What options were considered (and why rejected)
3. **Decision** - What was chosen and how to implement it
4. **Consequences** - Trade-offs, benefits, and risks

**Benefits:**

- Future developers understand "why" not just "what"
- Prevents re-litigating settled decisions
- Documents trade-offs for later review
- Creates institutional memory

## ADR Categories

The FF Analytics project uses ADRs for:

1. **Data Model Decisions** (e.g., ADR-007: Separate fact tables, ADR-009: 2×2 stat model)
2. **Infrastructure Choices** (e.g., ADR-004: GitHub Actions, ADR-006: GCS storage)
3. **Data Quality Policies** (e.g., ADR-005: Server-side copy, ADR-008: Idempotency)
4. **Identity & Conformance** (e.g., ADR-010: mfl_id canonical, ADR-011: Team conformance, ADR-014: Pick identity)

## ADR Creation Workflow

### Step 1: Identify Decision Context

**Ask these questions:**

- What problem needs solving?
- What constraints exist (technical, business, time)?
- Is this a new decision or updating an existing ADR?
- Does this supersede a previous ADR?
- What's the scope (data model, infrastructure, process)?

**Decision Scope Examples:**

**HIGH - Needs ADR:**

- Choosing between DuckDB vs PostgreSQL for local development
- Deciding on SCD Type 1 vs Type 2 for player dimension
- Selecting canonical player ID (mfl_id vs gsis_id)
- Establishing batch vs streaming architecture

**LOW - Does NOT need ADR:**

- Naming a specific model or column
- Minor code refactoring
- Formatting/style choices covered by linting
- Tactical bug fixes

**Rule of thumb:** If the decision impacts multiple components or will be referenced in future work, create an ADR.

### Step 2: Research Alternatives

**Thoroughly evaluate options:**

1. **List all viable alternatives** (aim for 2-4 options)
2. **Research each option**:
   - Technical feasibility
   - Implementation complexity
   - Performance implications
   - Cost (development time, runtime, maintenance)
   - Compatibility with existing architecture
3. **Consult existing ADRs** for precedents or conflicts
4. **Prototype if needed** (for high-risk decisions)

**Example (from ADR-010):**

| Alternative | Pros | Cons | Verdict |
|-------------|------|------|---------|
| gsis_id canonical | NFL authoritative | NFL-specific, couples to nflverse | Rejected |
| mfl_id canonical | Platform-neutral, nflverse provides crosswalk | Indirect (via crosswalk) | **Chosen** |
| Generate synthetic ID | Complete control | No provider alignment, complex migration | Rejected |

### Step 3: Draft ADR

**Use the template** from `assets/adr_template.md`:

1. **Get next ADR number**: Run `scripts/get_next_adr_number.py`

```bash
cd .claude/skills/adr-creator
python scripts/get_next_adr_number.py  # Returns ADR-015, ADR-016, etc.
```

2. **Create file**: `docs/adr/ADR-{NNN}-{slug}.md`
   - **Number**: Zero-padded 3 digits (ADR-004, ADR-015)
   - **Slug**: Kebab-case summary (e.g., `mfl-id-canonical-identity`)

3. **Fill all sections completely**:

#### Context Section

- **Problem statement**: Clearly define what needs deciding
- **Current state**: Describe relevant existing architecture
- **Constraints**: Technical, business, or timeline limitations
- **Stakeholders**: Who is affected by this decision?

**Good context example:**
> "The platform integrates 19+ fantasy data providers, each using different player IDs. We need a canonical player_id for dim_player that enables identity resolution across all providers while remaining stable as providers are added/removed."

#### Decision Section

- **Be specific and actionable**: Not "use DuckDB" but "use DuckDB for local development with External Parquet materialization for large models"
- **Include technical details**: Schemas, configurations, implementation approach
- **Reference code/SQL if helpful**: Concrete examples clarify intent

**Good decision example (from ADR-010):**
> "Use nflverse's `mfl_id` as the canonical `player_id` throughout the dimensional model. Create `dim_player_id_xref` crosswalk seed table mapping mfl_id to 19 provider-specific IDs."

#### Consequences Section

- **Positive consequences**: What benefits does this provide?
- **Negative consequences**: What trade-offs or risks?
- **Mitigation strategies**: How to address the negatives?

Be honest about trade-offs. Example:
> **Negative:** Requires join to crosswalk table for provider ID lookup
> **Mitigation:** Crosswalk is small (~5K rows), joins are fast; SCD Type 1 so no history bloat

### Step 4: Review & Refine

**Self-review checklist:**

- [ ] Context is clear and complete (someone unfamiliar with project understands the problem)
- [ ] Decision is specific and actionable (implementation team knows what to build)
- [ ] Alternatives are documented with rationale for rejection
- [ ] Consequences are realistic (not overly optimistic)
- [ ] References are complete (links to specs, related ADRs, external docs)
- [ ] Status is correct (Draft for new, Accepted after implementation, Superseded if replaced)

**Verification:**

Does this ADR answer:

1. **Why** did we make this decision? (Context)
2. **What** are we doing? (Decision)
3. **What else** did we consider? (Alternatives in Context)
4. **What** are the implications? (Consequences)

### Step 5: File & Update Index

**Filing:**

1. **Save ADR**: `docs/adr/ADR-{NNN}-{slug}.md`
2. **Update index**: Add entry to `docs/adr/README.md`
3. **Link from specs**: Reference ADR from SPEC-1 or other specs (if applicable)
4. **Link related ADRs**: Update superseded ADRs or related decisions

**Example index entry:**

```markdown
- [ADR-015: DuckDB External Parquet for Large Marts](ADR-015-duckdb-external-parquet.md) - **Accepted** (2024-11-08)
```

**Superseding previous ADRs:**

When ADR supersedes an older decision:

1. **Update old ADR status**: Change to "Superseded by ADR-{NNN}"
2. **Reference in new ADR**: Add "Supersedes: ADR-{XXX}" to frontmatter
3. **Explain why**: In Context section, explain why original decision no longer applies

## ADR Lifecycle

ADRs progress through statuses:

1. **Draft** - Initial proposal, under discussion
2. **Accepted** - Decision approved and being implemented
3. **Superseded** - Replaced by newer ADR (include ADR number)
4. **Deprecated** - No longer relevant (explain why)

**Update status as decisions evolve:**

- Draft → Accepted when implementation begins
- Accepted → Superseded when replaced by newer decision
- Accepted → Deprecated if approach is abandoned (with explanation)

## Best Practices

### Be Specific

**Bad:** "Use DuckDB for better performance"
**Good:** "Use DuckDB with External Parquet materialization for mart models >1M rows to reduce dev.duckdb file size and improve query performance via columnar scans"

### Document "Why Not" for Rejected Alternatives

Don't just state the decision - explain why alternatives were rejected:

**Example:**
> **Alternative: PostgreSQL**
>
> - ✅ Mature, widely used, excellent ecosystem
> - ❌ Requires separate server process (complexity for local dev)
> - ❌ No native Parquet support (needs external tools)
> - **Verdict:** Rejected due to local development complexity

### Include Concrete Examples

When possible, show actual code/SQL/schemas:

```sql
-- dim_player_id_xref (crosswalk seed)
CREATE TABLE dim_player_id_xref (
    player_id VARCHAR PRIMARY KEY,  -- mfl_id (canonical)
    gsis_id VARCHAR,                -- NFLverse
    sleeper_id INTEGER,             -- Sleeper
    ...
)
```

### Link Liberally

Reference:

- Related ADRs (supersedes, complements, conflicts)
- Specs (SPEC-1, feature specs)
- External resources (documentation, blog posts, papers)
- Code locations (when decision manifests in specific files)

### Keep Them Immutable (Mostly)

Once accepted, ADRs should rarely change. If decision evolves:

- **Small clarifications:** Edit in place (note date of edit)
- **Major changes:** Create new ADR that supersedes the old one

## Integration with Other Skills

- **strategic-planner**: Specs reference ADRs for architectural decisions
- **sprint-planner**: Sprint tasks implement decisions from ADRs
- **dbt-model-builder**: Model designs follow patterns established in ADRs (e.g., ADR-009: 2×2 model)

**Workflow:**

1. **strategic-planner** creates SPEC-1 (high-level architecture)
2. **adr-creator** documents key technical decisions from spec
3. **sprint-planner** breaks ADR implementations into tasks
4. Implementation references ADRs for guidance

## Resources

### assets/

- `adr_template.md` - Complete ADR template with all sections

### references/

- `example_adr.md` - Real ADR from project (ADR-010: mfl_id canonical identity)

### scripts/

- `get_next_adr_number.py` - Returns next available ADR number
- `create_adr.py` - Interactive ADR creation helper (use this to create new ADRs)

## Common Patterns

### Data Model ADRs

**Pattern:** Dimensional modeling decisions (facts, dimensions, grain, SCDs)

**Examples:**

- ADR-007: Separate fact tables per measure type
- ADR-009: 2×2 stat model (actuals vs projections, real-world vs fantasy)
- ADR-010: mfl_id canonical player identity
- ADR-014: Pick identity resolution

**Typical structure:**

- Context: Data model requirements, query patterns
- Decision: Specific schema design with SQL examples
- Consequences: Query complexity, join performance, storage

### Infrastructure ADRs

**Pattern:** Tooling and platform choices

**Examples:**

- ADR-004: GitHub Actions for Sheets sync
- ADR-006: GCS for cloud storage

**Typical structure:**

- Context: Infrastructure requirements, constraints
- Decision: Tool selection with configuration details
- Consequences: Cost, maintenance, learning curve
