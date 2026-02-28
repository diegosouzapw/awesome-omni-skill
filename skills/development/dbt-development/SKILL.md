---
name: dbt Development
description: PROACTIVE skill - STOP and invoke BEFORE writing dbt SQL. Validates models against coding conventions for staging, integration, and warehouse layers. Covers naming, SQL structure, field conventions, testing, and documentation. CRITICAL - When about to write .sql files in models/, invoke this skill first, write second. Supports project-specific convention overrides and sqlfluff integration.
---

# dbt Development Skill

## Purpose

This skill automatically activates when working with dbt models to ensure adherence to coding conventions and best practices. It provides validation and recommendations for model structure, naming, SQL style, testing, and documentation.

## ⚠️ CRITICAL: Self-Detection Triggers

**STOP and invoke this skill BEFORE writing dbt SQL if you catch yourself:**
- About to write a `.sql` file in a `models/` directory
- Thinking "I'll just copy the pattern from that other model..."
- About to create staging, integration, or warehouse models
- Writing SQL with `ref()` or `source()` calls
- Creating a `schema.yml` entry for testing
- Thinking "this looks like the right structure..."

**Rule of thumb:** About to write or modify dbt SQL → Invoke this skill first, write second.

**Why this matters:** Without invoking the skill, you'll rely on pattern-matching from existing code (which may be inconsistent) instead of validated conventions. The skill loads project-specific conventions and provides structured validation.

## When This Skill Activates

### User-Triggered Activation

This skill should activate when users:
- **Create new dbt models:** "Create a staging model for users from Salesforce"
- **Review existing models:** "Review this dbt model for issues"
- **Refactor models:** "Refactor this integration model to follow best practices"
- **Work with .sql files in models/:** Any read/write operations on dbt model files
- **Ask about dbt conventions:** "What are the naming conventions for warehouse models?"
- **Request schema/test files:** "Add tests to this model"

**Keywords to watch for:**
- "dbt model", "staging", "integration", "warehouse", "intermediate"
- "refactor", "review", "validate", "check conventions"
- "stg_", "int_", "_dim", "_fct"
- "schema.yml", "tests", "dbt test"

### Self-Triggered Activation (Proactive)

**⚠️ STOP and invoke this skill BEFORE creating or modifying dbt SQL when:**
- You're about to write any `.sql` file in a `models/` directory
- You're about to suggest creating a model from scratch
- User asks to "write SQL" or "create a model" in a dbt project context
- You're copying patterns from existing models (this means conventions aren't loaded!)
- Working with files that match dbt patterns (stg_, int_, _dim, _fct)
- You're about to update or create `schema.yml` test entries

**Internal thoughts that MUST trigger skill invocation:**
- "I'll create a staging model for..." → **STOP** - Invoke skill first
- "Let me copy the pattern from that existing model..." → **STOP** - Conventions not loaded!
- "I'll write this transformation..." in dbt context → **STOP** - Check conventions first
- "This model structure looks right..." → **STOP** - Validate, don't assume
- "Let me add tests to schema.yml..." → **STOP** - Verify test patterns first

**Anti-pattern to avoid:**
Reading an existing model, copying its structure, and assuming it follows conventions. Existing code may be inconsistent. Always invoke this skill to load and validate against authoritative convention sources.

## Instructions

### 0. Load Convention Source

**Priority Order (2-tier system):**
1. **Project-specific conventions** (highest priority)
   - Check for `.dbt-conventions.md` in project root
   - Check for `dbt_coding_conventions.md` in project root
   - Check for `docs/dbt_conventions.md` in project

2. **PKM user conventions** (fallback)
   - Conventions: `/Users/olivierdupois/dev/PKM/4. 🛠️ Craft/Tools/dbt/dbt-conventions.md`
   - Testing: `/Users/olivierdupois/dev/PKM/4. 🛠️ Craft/Tools/dbt/dbt-testing.md`

**Note:** The skill's supporting files (`conventions-reference.md`, `testing-reference.md`, `examples/`) are embedded reference documentation that guide validation logic, not convention sources.

**Detection:**
- Use `Glob` to search for convention files in project root
- If found, use `Read` to load project conventions
- If not found, use PKM conventions as fallback
- Note which source is being used in validation output

---

### 1. Identify Model Type and Context

When working with a dbt model, determine:

**Model Type:**
- **Staging** (`stg_`): First transformation layer, selects from sources
- **Integration** (`int_`): Combines multiple sources, enriches entities
- **Intermediate** (`int__<object>__<action>`): Subcomponent of integration
- **Warehouse - Dimension** (`_dim`): Mutable, noun-based entities
- **Warehouse - Fact** (`_fct`): Immutable, verb-based events

**Context Information:**
- File location in directory structure
- Source system (for staging models)
- Entity/object name
- Related models (refs)
- Expected materialization

**How to identify:**
- Check filename prefix/suffix
- Review directory structure (staging/, integration/, warehouse/)
- Read model content for `ref()` and `source()` calls
- Look at model configuration blocks

---

### 2. Validate Naming Conventions

**Check the following:**

**File and Model Naming:**
- ✓ All objects are singular (e.g., `user` not `users`)
- ✓ Staging: `stg_<source>__<object>.sql` (e.g., `stg_salesforce__user.sql`)
- ✓ Integration: `int__<object>.sql` (e.g., `int__user.sql`)
- ✓ Intermediate: `int__<object>__<action>.sql` (e.g., `int__user__unioned.sql`)
  - Actions should be past tense verbs (unioned, grouped, filtered, etc.)
- ✓ Warehouse dimensions: `<object>_dim.sql` or `<warehouse>_<object>_dim.sql`
  - Core warehouse has no prefix (e.g., `user_dim.sql`)
  - Other warehouses prefixed (e.g., `finance_revenue_dim.sql`)
- ✓ Warehouse facts: `<object>_fct.sql` or `<warehouse>_<object>_fct.sql`
  - Same prefix rules as dimensions

**Directory Structure:**
```
models/
├── staging/
│   └── <source_name>/
│       ├── stg_<source>.yml
│       └── stg_<source>__<object>.sql
├── integration/
│   ├── intermediate/
│   │   ├── intermediate.yml
│   │   └── int__<object>__<action>.sql
│   ├── int__<object>.sql
│   └── integration.yml
└── warehouse/
    └── <warehouse_name>/
        ├── <warehouse>.yml
        ├── <object>_dim.sql
        └── <object>_fct.sql
```

**Violations to Flag:**
- Plural object names
- Missing or incorrect prefixes/suffixes
- Non-standard directory structure
- Mismatched filename and directory location

---

### 3. Validate SQL Structure

**Required Structure:**

1. **All refs at top in CTEs:**
```sql
with

s_source_table as (
    select * from {{ ref('source_model') }}
),

s_another_source as (
    select * from {{ ref('another_model') }}
),
```

2. **CTE Naming:**
   - ✓ Prefix with `s_` for CTEs that select from refs/sources
   - ✓ Descriptive names for transformation CTEs (e.g., `filtered_events`, `aggregated_metrics`)
   - ✓ One logical unit of work per CTE

3. **Final CTE Pattern:**
```sql
final as (
    select
        -- fields here
    from s_source_table
    -- joins and where clauses
)

select * from final
```

4. **Configuration Block (if needed):**
```sql
{{
  config(
    materialized = 'table',
    sort = 'id',
    dist = 'id'
  )
}}
```

**Style Requirements:**
- ✓ 4-space indentation (not tabs)
- ✓ Lines no longer than 80 characters
- ✓ Lowercase field and function names
- ✓ Use `as` keyword for aliases
- ✓ Fields before aggregates/window functions
- ✓ Group by column name, not number
- ✓ Prefer `union all` to `union distinct`
- ✓ Explicit joins (`inner join`, `left join`, never just `join`)
- ✓ If joining 2+ tables, always prefix column names with table alias
- ✓ No table alias initialisms (use `customer`, not `c`)
- ✓ Comments for confusing CTEs

**Violations to Flag:**
- `ref()` or `source()` calls outside of top CTEs
- Missing final CTE
- Improper indentation or line length
- Uppercase SQL keywords or functions
- Implicit joins or missing join qualifiers
- Hard-to-understand table aliases

---

### 4. Validate Field Naming and Ordering

**Field Naming Conventions:**

**Primary Keys:**
- ✓ Named `<object>_pk` (e.g., `user_pk`, `transaction_pk`)
- ✓ Generated using `dbt_utils.surrogate_key()`
- ✓ Never look up PKs in separate queries

**Foreign Keys:**
- ✓ Named `<referenced_object>_fk` (e.g., `user_fk`, `transaction_fk`)
- ✓ Generated using `dbt_utils.surrogate_key()`

**Natural Keys:**
- ✓ Source system identifiers: `<descriptive_name>_natural_key`
- ✓ Example: `salesforce_user_natural_key`, `stripe_customer_natural_key`

**Timestamps:**
- ✓ Named `<event>_ts` (e.g., `created_ts`, `updated_ts`, `order_placed_ts`)
- ✓ Always in UTC timezone
- ✓ If different timezone, add suffix: `created_ts_ct`, `created_ts_pt`

**Booleans:**
- ✓ Prefixed with `is_` or `has_` (e.g., `is_active`, `has_subscription`)

**Prices/Revenue:**
- ✓ In decimal currency (e.g., 19.99 for $19.99)
- ✓ If stored in cents, add suffix: `price_in_cents`

**Common Fields:**
- ✓ Prefix with entity name (e.g., `customer_name`, `carrier_name`, not just `name`)

**General Rules:**
- ✓ All names in `snake_case`
- ✓ Use business terminology, not source terminology
- ✓ Avoid SQL reserved words
- ✓ Consistency across models (same field names for same concepts)

**Field Ordering (Staging/Base Models):**
1. Keys (pk, fks, natural keys)
2. Dates and timestamps
3. Attributes (dimensions/slicing fields)
4. Metrics (measures/aggregatable values)
5. Metadata (insert_ts, updated_ts, etc.)

Within each category, sort alphabetically.

**Violations to Flag:**
- Inconsistent naming patterns
- Missing _pk or _fk suffixes
- Timestamps without _ts suffix
- Booleans without is_/has_ prefix
- Reserved words as column names
- Incorrect field ordering

---

### 5. Validate Model Configuration

**Configuration Rules:**

**Warehouse Models:**
- ✓ Always materialized as `table`
- ✓ Consider sort/dist keys for performance

**Other Layers:**
- ✓ Prefer `view` or ephemeral (CTE) materialization
- ✓ Use `table` only if performance requires it

**Configuration Placement:**
- ✓ Model-specific config in the model file (in config() block)
- ✓ Directory-wide config in `dbt_project.yml`

**Example:**
```sql
{{
  config(
    materialized = 'table',
    sort = 'user_pk',
    dist = 'user_pk'
  )
}}
```

**Violations to Flag:**
- Warehouse models not materialized as tables
- Unnecessary table materializations in staging/integration
- Config that should be in dbt_project.yml but is in model

---

### 6. Validate Testing Coverage

**Minimum Testing Requirements:**

**Every Model:**
- ✓ Has a corresponding entry in a `schema.yml` file
- ✓ Primary key has `unique` and `not_null` tests
- ✓ Integration models with multiple sources: use `dbt_utils.unique_combination_of_columns`

**Schema.yml Location:**
- ✓ Every subdirectory should contain a `.yml` file
- ✓ Filename typically matches directory (e.g., `stg_salesforce.yml`, `integration.yml`)

**Example:**
```yaml
version: 2

models:
  - name: stg_salesforce__user
    description: Salesforce user records
    columns:
      - name: user_pk
        description: Unique identifier for user
        tests:
          - unique
          - not_null

      - name: email
        description: User email address
        tests:
          - not_null
```

**Additional Tests:**
- ✓ `relationships` tests for foreign keys
- ✓ `accepted_values` for enums/status fields
- ✓ `not_null_where` for conditional requirements
- ✓ Custom data tests in `tests/` directory for KPI validation

**Violations to Flag:**
- Missing schema.yml file
- Models without test coverage
- Primary keys without unique/not_null tests
- Missing relationships tests on foreign keys

---

### 7. Validate Documentation Coverage

**Documentation Requirements:**

**Staging Models:**
- ✓ 100% documented (all models and columns)
- ✓ Clear descriptions for all fields
- ✓ Business terminology explained

**Warehouse Models:**
- ✓ 100% documented (all models and columns)
- ✓ End-user focused descriptions

**Integration/Intermediate:**
- ✓ Document as needed for clarity
- ✓ Explain any special cases or complex logic

**Doc Blocks:**
- ✓ Use `{% docs %}` blocks for shared documentation
- ✓ Reference doc blocks to maintain consistency
- ✓ Store in `models/docs/` directory

**Example:**
```yaml
version: 2

models:
  - name: user_dim
    description: |
      User dimension containing customer profile information.
      Updated nightly from Salesforce and Stripe sources.
    columns:
      - name: user_pk
        description: "{{ doc('user_pk') }}"
```

**Violations to Flag:**
- Staging/warehouse models without descriptions
- Missing column documentation
- Vague or unhelpful descriptions

---

### 8. Run sqlfluff Validation (if available)

**Check for sqlfluff:**
```bash
which sqlfluff
```

**If available:**
1. Check for `.sqlfluff` config in project root
2. Run: `sqlfluff lint <model_file> --dialect <dialect>`
3. Include sqlfluff violations in validation output
4. Note: sqlfluff enforces many style conventions automatically

**If not available:**
- Note in output: "sqlfluff not detected - recommend installing for automated linting"
- Provide manual validation of style conventions

---

### 9. Output Validation Report

Structure your validation feedback as:

```
## dbt Model Validation Report

**Model:** `<model_name>.sql`
**Type:** <staging/integration/warehouse-dim/warehouse-fct>
**Convention Source:** <project-specific / RA defaults>

### Summary
- ✓ X checks passed
- ⚠️ Y issues found (N critical, M important, P nice-to-have)

### Naming Conventions
[✓/⚠️] **File naming:** <details>
[✓/⚠️] **Field naming:** <details>

### SQL Structure
[✓/⚠️] **CTE structure:** <details>
[✓/⚠️] **Style compliance:** <details>
[✓/⚠️] **Field ordering:** <details>

### Configuration
[✓/⚠️] **Materialization:** <details>
[✓/⚠️] **Performance settings:** <details>

### Testing
[✓/⚠️] **Schema.yml exists:** <details>
[✓/⚠️] **Primary key tests:** <details>
[✓/⚠️] **Foreign key tests:** <details>

### Documentation
[✓/⚠️] **Model description:** <details>
[✓/⚠️] **Column descriptions:** <details>

### sqlfluff
[✓/⚠️/N/A] **Linter results:** <details>

---

## Recommendations

### Critical Issues (must fix)
1. <issue description>
   - **Location:** <file:line or section>
   - **Current:** `<current code>`
   - **Should be:** `<correct pattern>`
   - **Reason:** <why this matters>

### Important Issues (should fix)
<same format>

### Nice-to-have Improvements
<same format>

---

## Examples

See `skills/dbt-development/examples/` for reference implementations:
- `staging-model-example.sql` - Compliant staging model
- `integration-model-example.sql` - Compliant integration model
- `warehouse-model-example.sql` - Compliant warehouse model
- `schema-example.yml` - Proper testing setup
```

---

## 10. Creating New Models

When creating a new dbt model from scratch:

**Step-by-step Process:**

1. **Determine Model Type**
   - Ask user if not clear: staging/integration/warehouse?
   - What source system (for staging)?
   - What entity/object?

2. **Generate File Structure**
   - Correct filename following conventions
   - Proper directory location
   - Configuration block if needed

3. **Build SQL Structure**
   - Refs/sources in CTEs at top
   - Transformation CTEs for logic
   - Final CTE
   - Select from final

4. **Apply Field Conventions**
   - Generate _pk using surrogate_key
   - Name foreign keys with _fk suffix
   - Timestamp fields with _ts suffix
   - Proper field ordering

5. **Create/Update schema.yml**
   - Model description
   - Column descriptions
   - Minimum tests (unique/not_null on pk)

6. **Validate Against Conventions**
   - Run through validation checklist
   - Provide preview before writing

---

## 11. Supporting References

**In This Skill Directory:**
- `conventions-reference.md` - Quick reference for naming, style, structure
- `testing-reference.md` - Test requirements and transformation layers
- `examples/staging-model-example.sql` - Staging model template
- `examples/integration-model-example.sql` - Integration model template
- `examples/warehouse-model-example.sql` - Warehouse model template
- `examples/schema-example.yml` - Testing and documentation example

**Convention Sources (2-tier system):**
- Project-specific: `.dbt-conventions.md` (if exists in project)
- PKM user conventions: `/Users/olivierdupois/dev/PKM/4. 🛠️ Craft/Tools/dbt/dbt-conventions.md` and `dbt-testing.md`

---

## 12. Important Guidelines

**Always Validate When:**
- Creating new dbt models
- Reviewing changes to existing models
- User asks for dbt guidance
- Working with .sql files in models/ directory
- Refactoring or cleaning up code

**Validation Mode (Not Auto-fix):**
- Provide clear, actionable feedback
- Show correct patterns with examples
- Explain WHY conventions matter
- Offer to make specific changes if user approves
- Never silently modify without explaining

**Project Awareness:**
- Always check for project-specific conventions first
- Note which convention source is being used
- Respect project overrides while suggesting RA best practices

**Priority Levels:**
- **Critical:** Breaks functionality, violates core principles, missing required tests
- **Important:** Inconsistent with conventions, maintainability issues, missing documentation
- **Nice-to-have:** Style preferences, minor optimizations, enhanced documentation

---

## 13. Examples of Activation

**Example 1: Creating a Staging Model**
```
User: "Create a staging model for Hubspot contacts"

Actions:
1. Activate dbt Development skill
2. Load convention source (project or RA defaults)
3. Determine: staging model, Hubspot source, contact object
4. Generate: stg_hubspot__contact.sql with proper structure
5. Create schema.yml entry with tests
6. Validate against all conventions
7. Present model for review
```

**Example 2: Reviewing Existing Model**
```
User: "Review this dbt model" [provides file]

Actions:
1. Activate dbt Development skill
2. Load convention source
3. Identify model type from filename/content
4. Run through validation checklist (naming, structure, fields, tests, docs)
5. Check sqlfluff if available
6. Generate validation report with recommendations
```

**Example 3: Refactoring**
```
User: "This integration model needs refactoring to match conventions"

Actions:
1. Activate dbt Development skill
2. Load conventions
3. Analyze current model structure
4. Identify violations
5. Provide detailed refactoring plan with before/after examples
6. Offer to apply changes section by section with user approval
```

---

## 14. Skill Deactivation

Do NOT activate this skill when:
- Working with non-dbt SQL (raw queries, database migrations, etc.)
- User explicitly says "ignore conventions" or "quick prototype"
- Files outside models/ directory (analyses, macros have different conventions)
- User is asking about dbt Cloud, dbt Core installation, or infrastructure (not model development)
