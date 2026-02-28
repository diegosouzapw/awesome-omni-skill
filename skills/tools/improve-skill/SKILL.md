---
name: improve-skill
description: Review and improve skills in capability files for agent documents. Use when improving toolReferences, instructions, installScript, implementationReference, and stage checklists.
---

# Improve Skills for Agent Documents

Review and improve skills in a capability file to produce excellent agent skill
documents. Focus on `toolReferences`, `instructions`, `installScript`,
`implementationReference`, and stage `readChecklist`/`confirmChecklist`
sections.

## When to Use

- Reviewing skills with `agent:` sections in capability files
- Improving tool references for better agent skill documents
- Splitting content across the three progressive disclosure layers
- Improving stage checklists (read-then-do and do-then-confirm)
- Ensuring generated SKILL.md files are useful and complete

## Context

Skills with `agent:` sections generate a skill directory with up to three files
following progressive disclosure:

- `SKILL.md` — Stage checklists + tools + `instructions` (always generated)
- `scripts/install.sh` — From `installScript` field (only when present)
- `references/REFERENCE.md` — From `implementationReference` field (only when
  present)

The template (`products/pathway/templates/skill.template.md`):

- Renders `toolReferences` as a **Required Tools** table automatically
- Renders `instructions` as an **Instructions** section in the SKILL.md body
- Links to `scripts/install.sh` when `installScript` exists
- Links to `references/REFERENCE.md` when `implementationReference` exists

### Progressive Disclosure Fields

| Field                     | Purpose                    | Output Target                         |
| ------------------------- | -------------------------- | ------------------------------------- |
| `toolReferences`          | Required tools (table)     | `<required_tools>` in SKILL.md        |
| `instructions`            | Main workflow guidance     | `## Instructions` section in SKILL.md |
| `installScript`           | Environment setup commands | `scripts/install.sh`                  |
| `implementationReference` | Code examples & reference  | `references/REFERENCE.md`             |

### Schema Fields for Skills

Every skill lives inside a capability YAML file. The complete set of fields for
a skill object:

| Field                     | Required | Type    | Description                                                                           |
| ------------------------- | -------- | ------- | ------------------------------------------------------------------------------------- |
| `id`                      | Yes      | string  | Unique identifier (`snake_case`)                                                      |
| `name`                    | Yes      | string  | Human-readable name                                                                   |
| `isHumanOnly`             | No       | boolean | If true, excluded from agent profiles                                                 |
| `human`                   | Yes      | object  | Human-specific content (`description`, `levelDescriptions`)                           |
| `agent`                   | No       | object  | Agent section (`name`, `description`, `useWhen`, `stages`)                            |
| `toolReferences`          | No       | array   | Required tools with `name`, `description`, `useWhen`, optional `url` and `simpleIcon` |
| `instructions`            | No       | string  | Workflow guidance rendered inline in SKILL.md body                                    |
| `installScript`           | No       | string  | Shell commands exported to `scripts/install.sh`                                       |
| `implementationReference` | No       | string  | Code examples exported to `references/REFERENCE.md`                                   |

**Field ordering in YAML:** Follow the schema order — `id`, `name`,
`isHumanOnly`, `human`, `agent`, `toolReferences`, `instructions`,
`installScript`, `implementationReference`.

**Validation rules** (enforced by `npx fit-map validate`):

- `instructions`, `installScript`, `implementationReference` must be strings
  (use YAML `|` block scalar)
- `implementationReference` must NOT contain `<onboarding_steps>` tags (extract
  to `installScript` instead)
- `toolReferences` entries require `name`, `description`, `useWhen`; `url` and
  `simpleIcon` are optional
- `agent.stages` must use valid stage IDs: `specify`, `plan`, `onboard`, `code`,
  `review`, `deploy`
- Each stage requires `focus` (string), `readChecklist` (array),
  `confirmChecklist` (array)

### Data File Locations

Skills live in capability YAML files in **two** locations that must stay in
sync:

- `data/capabilities/{id}.yaml` — Active installation data
- `products/map/examples/capabilities/{id}.yaml` — Canonical example data

When a capability exists in both locations, update both files. Check with:

```sh
diff data/capabilities/{id}.yaml products/map/examples/capabilities/{id}.yaml
```

## Core Principles

### Progressive Disclosure

Content follows three layers matching the Agent Skills specification:

1. **SKILL.md** (< 200 lines) — Stage checklists, instructions, tool table.
   Loaded when the skill is activated. Keep lean.
2. **scripts/install.sh** (10-30 lines) — Self-contained install script. Loaded
   on demand during onboarding.
3. **references/REFERENCE.md** (50-200 lines) — Code examples, patterns,
   pitfalls. Loaded on demand during implementation.

### Content Splitting Decision Tree

When reviewing content, decide where each piece belongs:

1. **Is it a tool name, URL, or "use when" description?** → `toolReferences`
2. **Is it a numbered workflow step or process guidance?** → `instructions`
3. **Is it a `pip install`, `npm install`, or `set -e` shell command?** →
   `installScript`
4. **Is it a code example, config snippet, or verification command?** →
   `implementationReference`
5. **Is it a pre-work check or quality gate?** → stage `readChecklist` /
   `confirmChecklist`

Some skills only need one or two of the optional fields. Not every skill needs
all three. A skill with no install steps and no code examples is fine with just
`instructions` or even nothing beyond the stage checklists.

### Outcome-Oriented Instructions

The `instructions` field (string, YAML `|` block scalar) provides **step-by-step
workflow guidance** that goes into the SKILL.md body. It should guide the reader
through the main workflow from start to finish.

Ask: _"If I follow these instructions, will I achieve the skill's stated
purpose?"_

**Size guide:** 10-50 lines. Use markdown headings (`## Step 1: ...`) to
structure the flow. No code blocks — those belong in `implementationReference`.

### Install Scripts

The `installScript` field (string, YAML `|` block scalar) contains **only shell
commands** for environment setup. Exported as `scripts/install.sh` with
executable permissions (mode 0o755).

- Plain bash commands, no markdown, no prose
- Start with `set -e` for error handling
- Pin dependency versions where practical
- End with verification commands (e.g. `command -v tool`,
  `python -c "import pkg"`)
- Do NOT include workflow steps, code examples, or explanatory text

**Size guide:** 5-30 lines.

### Implementation Reference

The `implementationReference` field (string, YAML `|` block scalar) contains
**code examples, patterns, and verification steps**. Exported as
`references/REFERENCE.md`.

- Must NOT duplicate tool information (already in `toolReferences`)
- Must NOT contain workflow steps (those go in `instructions`)
- Must NOT contain install commands (those go in `installScript`)
- Must NOT contain `<onboarding_steps>` tags (validation will reject them)
- Should include a **Verification** section with runnable commands
- Should include a **Common Pitfalls** section where relevant

**Size guide:** 30-200 lines. Use markdown headings to organize sections.

### Minimal Essential Tooling

Recommend only the **core tools required** to achieve the skill's outcome. A
focused set of 2-4 essential tools is more valuable than an exhaustive list.

Ask: _"Could someone complete this skill without this tool?"_ If yes, omit it.

### Checklist Manifesto Checklists

Stage checklists exist to **prevent critical failures**, not to be exhaustive
task lists. They should cause the agent to pause, reflect, and ask the user
where relevant.

- **readChecklist** (Read-Then-Do): Critical checks to read **before** starting
  the stage. These gate entry — don't start until every item is satisfied.
- **confirmChecklist** (Do-Then-Confirm): Critical checks to verify **before**
  handing off to the next stage. These gate exit — don't proceed until every
  item is confirmed.

Ask: _"If someone skipped this check, what could go wrong?"_ If the answer is
"something critical fails silently," it belongs in the checklist.

## Process

1. **Identify the capability** to review (ask if not specified)
2. **Read the capability file** from both `data/capabilities/{id}.yaml` and
   `products/map/examples/capabilities/{id}.yaml`
3. **For each skill with an `agent:` section**, review and improve
4. **Study the updated skill** by running `npx fit-pathway skill <name> --agent`
5. **Iterate** until the skill document is clear, complete, and well-structured
6. **Run validation**: `npx fit-map validate`

### Tool References Review

Aim for **2-4 essential tools** per skill. Check that `toolReferences`:

- **Include only tools essential** to achieve the skill's core outcome
- Include the primary tool used in `implementationReference` code samples
- Prefer open source tools and libraries over commercial offerings (exceptions:
  ubiquitous platforms like AWS, GitHub, Azure, GCP are fine)
- Have accurate, concise `description` fields
- Have specific `useWhen` guidance relevant to this skill (not generic)
- Include `url` for official documentation where available
- Include `icon` field where appropriate using Simple Icons names (use `task`,
  `python`, `typescript` as generic fallbacks)

**Exclude:**

- Nice-to-have tools that aren't central to the implementation
- Alternative tools (pick one, don't list options)
- Generic utilities (linters, formatters) unless skill-specific
- Tools mentioned only in passing

### Instructions Review

Check that `instructions`:

- **Follows a logical sequence** using markdown headings (`## Step 1: ...`)
- **Provides clear workflow guidance** from start to finish
- Does NOT contain code examples (those go in `implementationReference`)
- Does NOT contain install commands (those go in `installScript`)
- Does NOT contain tool tables (those go in `toolReferences`)
- Stays under ~50 lines for SKILL.md body budget
- Uses YAML `|` block scalar (not `>` which folds newlines)

### Install Script Review

Check that `installScript` (if present):

- Contains **only shell commands** (no markdown, no prose, no `#` section
  headers)
- Starts with `set -e` for error handling
- Pins dependency versions where practical (e.g. `pip install duckdb==0.10.0`)
- Includes verification commands (e.g. `command -v tool`,
  `python -c "import pkg"`)
- Does NOT contain code examples or workflow steps
- Uses YAML `|` block scalar

### Implementation Reference Review

Check that `implementationReference` (if present):

- **Shows complete, working code** (not fragments or pseudocode)
- **Does NOT contain** tool tables or "Technology Stack" sections
- **Does NOT contain** `<onboarding_steps>` tags (validation will reject them;
  extract to `installScript` instead)
- **Does NOT contain** step-by-step workflow guidance (use `instructions`
  instead)
- **Does NOT contain** `pip install` / `npm install` commands (use
  `installScript` instead)
- **Includes verification** so the reader knows when they've succeeded
- **Includes Common Pitfalls** section where relevant
- Uses YAML `|` block scalar

### Cross-Field Consistency Review

After reviewing individual fields, check cross-field consistency:

- Tools in `implementationReference` code samples appear in `toolReferences`
- Packages in `installScript` match imports in `implementationReference`
- `instructions` workflow references tools that exist in `toolReferences`
- Stage checklists reference tools/packages from the other fields
- No content is duplicated across fields

### Stage Checklists Review

Each stage (`specify`, `plan`, `onboard`, `code`, `review`, `deploy`) has a
`readChecklist` and `confirmChecklist`. Review all stages for the skill.

#### Writing Effective Checklists

**1. Force pause-and-reflect with ASK items**

Use `ASK the user` prefix for items that require information the agent cannot
infer. This forces the agent to stop and gather critical input before
proceeding.

Good examples by stage:

- **specify**:
  `ASK the user what business problem this should solve — get a concrete problem statement`
- **onboard**: `ASK the user for a valid GITHUB_TOKEN with Models access`
- **onboard**:
  `ASK the user for database credentials (connection string, API key)`

Bad: `Configure API keys` (vague, agent may skip or guess)

**2. Be explicit and verifiable**

Each item should be concrete enough that someone can unambiguously determine
whether it's done. Include specific commands, thresholds, or observable
outcomes.

Good: `python -c "import sklearn, pandas, mlflow"` succeeds Bad:
`ML frameworks installed`

Good: `Train/test gap within acceptable range (< 5%)` Bad: `No overfitting`

Good: `All credentials stored in .env file and .env is listed in .gitignore`
Bad: `Environment variables configured`

**3. Include quantified thresholds where possible**

Replace vague quality bars with concrete numbers:

- `gap < 5%` instead of "acceptable range"
- `p50 and p95 latency measured` instead of "latency checked"
- `at least 3× model size available for checkpoints` instead of "sufficient disk
  space"
- `similarity scores > 0.7 indicates good relevance` instead of "good relevance"

**4. Cover security and credentials explicitly**

Every `onboard` checklist should address credentials:

- readChecklist: `ASK the user for [specific credential]`
- confirmChecklist: `All credentials stored in .env — NEVER hardcoded in code`

**5. Cover domain-specific critical risks**

Each skill has unique failure modes. The checklists must explicitly guard
against them:

| Domain        | Critical Risks to Check                                    |
| ------------- | ---------------------------------------------------------- |
| ML models     | Data leakage, overfitting, bias/fairness, class imbalance  |
| RAG systems   | Hallucination, chunk quality, source citation, cost/query  |
| Fine-tuning   | Catastrophic forgetting, VRAM limits, licensing, eval loss |
| Observability | Tracing gaps, missing spans, dashboard verification        |
| Deployment    | Rollback procedures, monitoring, alerting                  |

**6. Match checklist items to stage purpose**

- **specify**: Gather requirements from user, define success criteria
- **plan**: Technical decisions, architecture, evaluation methodology
- **onboard**: Environment setup, credentials, tool installation, data access
- **code**: Implementation, testing, experiment tracking
- **review**: Validation against criteria, edge cases, bias checks
- **deploy**: Production readiness, monitoring, rollback, documentation

#### Checklist Anti-Patterns

| Anti-Pattern                    | Fix                                                |
| ------------------------------- | -------------------------------------------------- |
| Vague items ("check quality")   | Be specific ("retrieval precision@5 > 0.8")        |
| Missing user prompts in onboard | Add `ASK the user for` credential/config items     |
| No verification commands        | Add runnable commands that prove success           |
| Duplicating focus text          | Checklists complement focus, don't repeat it       |
| Too few items (< 4)             | Cover all critical failure modes for the stage     |
| Too many items (> 10)           | Keep only items where skipping causes real failure |
| Generic across skills           | Tailor to the skill's unique domain risks          |
| No security checks              | Add credential storage and .gitignore verification |
| Missing rollback/recovery       | Deploy stage should always cover rollback          |

### Common Problems to Fix (All Sections)

| Problem                       | Fix                                               |
| ----------------------------- | ------------------------------------------------- |
| Too many tools                | Keep only 2-4 essential tools for core outcome    |
| Mixed content in one field    | Split across instructions/installScript/reference |
| `<onboarding_steps>` tags     | Extract to `installScript` field                  |
| Workflow steps in reference   | Move to `instructions` field                      |
| Install commands in reference | Move to `installScript` field                     |
| Tool lists in reference       | Remove (already in `toolReferences`)              |
| Code without context          | Add prose explaining what each section achieves   |
| SKILL.md over 200 lines       | Move code examples to `implementationReference`   |
| Vague checklists              | Add ASK items, commands, and thresholds           |
| Missing ASK items             | Onboard and specify stages need user prompts      |
| No credential checks          | Add .env and .gitignore verification              |

### Good Structure Pattern

````yaml
# In capability YAML file — skill with all three optional fields
- id: data_integration
  name: Data Integration
  human:
    description: ...
    levelDescriptions: ...
  agent:
    name: data-integration
    description: Build robust data pipelines
    useWhen: Working with data ingestion or ETL
    stages:
      onboard:
        focus: Set up data tools and verify data access
        readChecklist:
          - ASK the user for database connection string
          - Install pipeline tools — `pip install duckdb polars`
        confirmChecklist:
          - All packages installed — `python -c "import duckdb, polars"` succeeds
          - All credentials stored in .env — NEVER hardcoded in code
      code:
        focus: Build and test the pipeline
        readChecklist:
          - Read instructions section for workflow steps
          - Read references/REFERENCE.md for code patterns
        confirmChecklist:
          - Pipeline produces expected output shape
          - Error handling covers connection failures
  toolReferences:
    - name: DuckDB
      url: https://duckdb.org/docs
      description: In-process SQL analytics engine
      useWhen: Querying local files or building analytical pipelines
    - name: Polars
      url: https://docs.pola.rs
      description: Fast DataFrame library
      useWhen: Transforming tabular data with lazy evaluation
  instructions: |
    ## Step 1: Explore the Source Data
    Profile the source data to understand schema, volume, and quality.

    ## Step 2: Build the Pipeline
    Create extraction, transformation, and loading stages.

    ## Step 3: Add Validation
    Define data quality expectations and verify output.
  installScript: |
    set -e
    pip install duckdb polars great-expectations
    python -c "import duckdb, polars, great_expectations"
  implementationReference: |
    ## Client Setup
    ```python
    import duckdb
    conn = duckdb.connect(":memory:")
    df = conn.execute("SELECT * FROM 'data.parquet'").pl()
    ```

    ## Verification
    ```python
    assert df.shape[0] > 0, "Pipeline produced no rows"
    assert set(df.columns) >= {"id", "value"}, "Missing required columns"
    ```

    ## Common Pitfalls
    - Forgetting to close DuckDB connections in error paths
    - Not handling NULL values in join keys
````

```yaml
# Skill with instructions only — no install or reference needed
- id: problem_discovery
  name: Problem Discovery
  human: ...
  agent:
    name: problem-discovery
    description: Discover and validate user problems
    useWhen: Starting new feature work or investigating user pain points
    stages: ...
  instructions: |
    ## Discovery Process
    Interview stakeholders and gather evidence before proposing solutions.

    ## Validation
    Confirm the problem exists with data before building anything.
```

### Good Checklist Pattern

```yaml
stages:
  onboard:
    focus: |
      Set up the development environment...
    readChecklist:
      - ASK the user for [specific credential or config needed]
      - ASK the user to confirm [prerequisite environment detail]
      - Install [specific packages] — `pip install pkg1 pkg2`
      - Configure [specific tool] with [specific settings]
      - Verify [specific data/service] is accessible
    confirmChecklist:
      - All packages installed — `python -c "import pkg1, pkg2"` succeeds
      - [Service] running — `curl localhost:port` responds
      - [Credential] configured — `tool whoami` succeeds
      - All credentials stored in .env — NEVER hardcoded in code
      - Dependencies pinned in requirements.txt
```

## Output

1. Summarize issues found
2. Apply fixes directly to the capability file (both locations if applicable)
3. Run `npx fit-map validate` to verify changes

## Memory

> **Keep this section up to date.** After every review or tweak to the skill
> improvement process, add a dated note here. This prevents repeating past
> mistakes and preserves decisions made during reviews. When completing a
> review, check this section first and update it with any new decisions.

### Protected Tools

The following tools have been explicitly identified as critical and **must NOT
be removed** during skill reviews, even if they appear to exceed the 2-4 tool
guideline:

- **GitHub Models** (in `retrieval_augmentation`) — Critical for developer
  experience. Provides free LLM inference via GITHUB_TOKEN with zero cost. Do
  not remove in favour of generic OpenAI references.
- **Playwright** (in `code_quality`) — Critical for end-to-end testing. The
  primary E2E testing framework for verifying full application behavior across
  browsers.
- **Rancher Desktop** (in `full_stack_development`, `devops`) — Critical for
  container runtime on macOS. Provides Docker socket at `/var/run/docker.sock`
  with no licensing concerns. Replaces Colima (which has docker.sock mount
  issues with Supabase).
- **uv** (all Python skills) — Critical for Python package management. Replaces
  pip with 10-50x faster installs, built-in lockfiles, and Python version
  management. Do not revert to pip.
- **mise** (skills with runtime requirements) — Critical for runtime version
  pinning. Each skill provides a mise fragment in implementationReference. Do
  not remove or replace with nvm/pyenv/asdf.
- **just** (skills with dev workflows) — Critical for project-level task
  automation. Each skill provides justfile recipe fragments in
  implementationReference. Do not remove.
- **AWS Step Functions** (in `cloud_platforms`) — Critical for modern serverless
  practices. Essential for orchestrating multi-step workflows with error
  handling, retries, and state management.

### Review Log

- **2026-02-10**: Restored GitHub Models, Playwright, Colima, and Step Functions
  after initial review incorrectly removed them as "nice-to-have". Added this
  Memory section to prevent recurrence.
- **2026-02-11**: DX overhaul — Replaced Colima with Rancher Desktop, pip with
  uv, added mise and just. All Python skills use `uv sync` instead of
  `pip install`. Skills provide composable Docker Compose, justfile, and mise
  fragments in implementationReference for Tier 3 project-level assembly.
