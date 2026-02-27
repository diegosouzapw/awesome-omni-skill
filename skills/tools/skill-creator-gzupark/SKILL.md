---
name: skill-creator
description: Guide for creating effective skills. Use when users want to create or update a skill that extends Claude's capabilities with specialized knowledge, workflows, or tool integrations.
license: Complete terms in LICENSE.txt
---

# Skill Creator

This skill provides guidance for creating effective skills.

## About Skills

Skills are modular, self-contained packages that extend Claude's capabilities
by providing specialized knowledge, workflows, and tools.
Think of them as "onboarding guides" for specific domains or tasks -
they transform Claude from a general-purpose agent into a specialized agent
equipped with procedural knowledge that no model can fully possess.

### What Skills Provide

1. Specialized workflows - Multi-step procedures for specific domains
2. Tool integrations - Instructions for working with file formats or APIs
3. Domain expertise - Company-specific knowledge, schemas, business logic
4. Bundled resources - Scripts, references, and assets for repetitive tasks

## Core Principles

### Concise is Key

The context window is a public good.
Skills share the context window with everything else Claude needs:
system prompt, conversation history, other Skills' metadata, and user request.

**Default assumption: Claude is already very smart.**
Only add context Claude doesn't already have.
Challenge each piece of information:
"Does Claude really need this?" and "Does this justify its token cost?"

Prefer concise examples over verbose explanations.

### Set Appropriate Degrees of Freedom

Match specificity to the task's fragility and variability:

**High freedom (text-based instructions)**: Use when multiple approaches are
valid, decisions depend on context, or heuristics guide the approach.

**Medium freedom (pseudocode or scripts with parameters)**: Use when a
preferred pattern exists, some variation is acceptable, or config affects behavior.

**Low freedom (specific scripts, few parameters)**: Use when operations are
fragile and error-prone, consistency is critical, or specific sequence is needed.

Think of Claude as exploring a path: a narrow bridge needs specific guardrails
(low freedom), while an open field allows many routes (high freedom).

### Anatomy of a Skill

Every skill consists of a required SKILL.md file and optional bundled resources:

```text
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter metadata (required)
│   │   ├── name: (required)
│   │   └── description: (required)
│   └── Markdown instructions (required)
└── Bundled Resources (optional)
    ├── scripts/          - Executable code (Python/Bash/etc.)
    ├── references/       - Documentation to be loaded as needed
    └── assets/           - Files used in output (templates, etc.)
```

#### SKILL.md (required)

Every SKILL.md consists of:

- **Frontmatter** (YAML): Contains `name` and `description` fields.
  These are the only fields that Claude reads to determine when the skill gets
  used, so be clear about what the skill is and when it should be used.
- **Body** (Markdown): Instructions and guidance for using the skill.
  Only loaded AFTER the skill triggers (if at all).

#### Bundled Resources (optional)

**Scripts (`scripts/`)**: Executable code (Python/Bash/etc.) for tasks
that require deterministic reliability or are repeatedly rewritten.

- **When to include**: When code is being rewritten repeatedly or reliability needed
- **Example**: `scripts/rotate_pdf.py` for PDF rotation tasks
- **Benefits**: Token efficient, deterministic, executed without loading context
- **Note**: Scripts may still need to be read for patching or env adjustments

**References (`references/`)**: Documentation and reference material
intended to be loaded as needed to inform Claude's process and thinking.

- **When to include**: For documentation that Claude should reference while working
- **Examples**: `references/finance.md` for financial schemas, `references/mnda.md`
  for company NDA template, `references/api_docs.md` for API specifications
- **Use cases**: Database schemas, API documentation, domain knowledge, policies
- **Benefits**: Keeps SKILL.md lean, loaded only when Claude determines it's needed
- **Best practice**: If files are large (>10k words), include grep patterns
- **Avoid duplication**: Information should live in either SKILL.md or references,
  not both. Prefer references for detailed info unless truly core to the skill.

**Assets (`assets/`)**: Files not intended to be loaded into context,
but rather used within the output Claude produces.

- **When to include**: When the skill needs files for the final output
- **Examples**: `assets/logo.png` for brand assets, `assets/slides.pptx` for
  templates, `assets/frontend-template/` for boilerplate, `assets/font.ttf`
- **Use cases**: Templates, images, icons, boilerplate code, fonts, samples
- **Benefits**: Separates output resources from docs, use files without context

#### What to Not Include in a Skill

A skill should only contain essential files that directly support its
functionality. Do NOT create extraneous documentation or auxiliary files:

- README.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md
- etc.

The skill should only contain information needed for an AI agent to do the job.
Do not include auxiliary context, setup procedures, or user-facing documentation.

### Progressive Disclosure Design Principle

Skills use a three-level loading system to manage context efficiently:

1. **Metadata (name + description)** - Always in context (~100 words)
2. **SKILL.md body** - When skill triggers (<5k words)
3. **Bundled resources** - As needed by Claude (scripts can execute without context)

#### Progressive Disclosure Patterns

Keep SKILL.md body under 500 lines to minimize context bloat.
Split content into separate files when approaching this limit.
Reference them from SKILL.md and describe when to read them.

**Key principle:** When a skill supports multiple variations or frameworks,
keep only core workflow and selection guidance in SKILL.md.
Move variant-specific details into separate reference files.

##### Pattern 1: High-level guide with references

```markdown
# PDF Processing

## Quick start

Extract text with pdfplumber:
[code example]

## Advanced features

- **Form filling**: See [FORMS.md](FORMS.md) for complete guide
- **API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
- **Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
```

Claude loads FORMS.md, REFERENCE.md, or EXAMPLES.md only when needed.

##### Pattern 2: Domain-specific organization

For Skills with multiple domains, organize content by domain
to avoid loading irrelevant context:

```text
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

When a user asks about sales metrics, Claude only reads sales.md.

Similarly, for skills supporting multiple frameworks, organize by variant:

```text
cloud-deploy/
├── SKILL.md (workflow + provider selection)
└── references/
    ├── aws.md (AWS deployment patterns)
    ├── gcp.md (GCP deployment patterns)
    └── azure.md (Azure deployment patterns)
```

When the user chooses AWS, Claude only reads aws.md.

##### Pattern 3: Conditional details

Show basic content, link to advanced content:

```markdown
# DOCX Processing

## Creating documents

Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents

For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

Claude reads REDLINING.md or OOXML.md only when user needs those features.

**Important guidelines:**

- **Avoid deeply nested references** - Keep references one level deep from
  SKILL.md. All reference files should link directly from SKILL.md.
- **Structure longer reference files** - For files longer than 100 lines,
  include a TOC at the top so Claude can see the full scope when previewing.

## Skill Creation Process

Skill creation involves these steps:

1. **Understanding** - Understand the skill with concrete examples
2. **Planning** - Plan reusable skill contents (scripts, references, assets)
3. **Analysis** - Apply analysis frameworks (complex skills only)
4. **Initialization** - Initialize the skill (run init_skill.py)
5. **Implementation** - Edit the skill (implement resources and write SKILL.md)
6. **Packaging** - Package the skill (run package_skill.py)
7. **Iteration** - Iterate based on real usage

Follow these steps in order, skipping only if clearly not applicable.

**Complex skills** (with external integrations or requiring careful design)
should apply the full process including Step 3. Simple skills may skip Analysis.

### Step 1: Understanding the Skill with Concrete Examples

Skip this step only when the skill's usage patterns are already clearly
understood. It remains valuable even when working with an existing skill.

To create an effective skill, clearly understand concrete examples of how
the skill will be used. This can come from direct user examples or generated
examples validated with user feedback.

For example, when building an image-editor skill, relevant questions include:

- "What functionality should the image-editor skill support?"
- "Can you give some examples of how this skill would be used?"
- "I can imagine users asking to 'Remove the red-eye' or 'Rotate this image'.
  Are there other ways you imagine this skill being used?"
- "What would a user say that should trigger this skill?"

To avoid overwhelming users, avoid asking too many questions in a single message.
Start with the most important questions and follow up as needed.

Conclude this step when there is a clear sense of the functionality needed.

### Step 2: Planning the Reusable Skill Contents

To turn concrete examples into an effective skill, analyze each example by:

1. Considering how to execute the example from scratch
2. Identifying helpful scripts, references, and assets for repeated workflows

Example: When building a `pdf-editor` skill to handle "Help me rotate this PDF":

1. Rotating a PDF requires re-writing the same code each time
2. A `scripts/rotate_pdf.py` script would be helpful to store in the skill

Example: When designing a `frontend-webapp-builder` skill
for "Build me a todo app" or "Build me a dashboard":

1. Writing a frontend webapp requires the same boilerplate each time
2. An `assets/hello-world/` template with boilerplate would be helpful

Example: When building a `big-query` skill for "How many users logged in today?":

1. Querying BigQuery requires re-discovering table schemas each time
2. A `references/schema.md` file documenting schemas would be helpful

To establish the skill's contents, analyze each concrete example
to create a list of the reusable resources: scripts, references, and assets.

### Step 3: Analysis (Complex Skills Only)

For complex skills (external integrations, significant design decisions),
apply systematic analysis:

**6-Lens Framework**: Apply 6 mental models to validate design decisions:

- First Principles, Pre-Mortem, Constraint Analysis, Pareto, Root Cause, Systems
- See [analysis-lenses.md](references/analysis-lenses.md) for detailed guidance

**Regression Questioning**: Iteratively question until no new insights emerge:

- 7 categories: Scope, Users, Technical, Quality, Integration, Evolution, Risks
- Termination: 3 consecutive empty rounds or max 7 rounds
- See [regression-questioning.md](references/regression-questioning.md)

Skip this step for simple skills with clear requirements and minimal decisions.

### Step 4: Initializing the Skill

At this point, it is time to actually create the skill.

Skip this step only if the skill already exists and needs iteration or packaging.
In this case, continue to the next step.

When creating a new skill from scratch, always run the `init_skill.py` script:

```bash
./run.sh scripts/init_skill.py <skill-name> --path <output-directory> [--mode simple|complex]
```

Options:

- `--mode simple` (default): Basic skill structure
- `--mode complex`: For skills with external integrations

The script:

- Creates the skill directory at the specified path
- Generates a SKILL.md template with proper frontmatter and TODO markers
- Creates example resource directories: `scripts/`, `references/`, and `assets/`
- Adds example files in each directory that can be customized or deleted
- Creates `run.sh` wrapper script for executing Python scripts

After initialization, customize or remove the generated SKILL.md and examples.

**Running scripts:** Use `./run.sh` to execute Python scripts:

```bash
./run.sh scripts/example.py [args...]
```

`run.sh` uses `uv` if available, or falls back to a shared venv.

### Step 5: Edit the Skill

When editing the skill, remember it's created for another Claude to use.
Include information that would be beneficial and non-obvious.
Consider what procedural knowledge, domain-specific details, or reusable assets
would help another Claude instance execute these tasks more effectively.

#### Learn Proven Design Patterns

Consult these helpful guides based on your skill's needs:

- **Multi-step processes**: See [references/workflows.md](references/workflows.md)
  for sequential workflows and conditional logic
- **Specific output formats**: See [references/output-patterns.md](references/output-patterns.md)
  for template and example patterns
- **Script patterns**: See [references/script-patterns.md](references/script-patterns.md)
  for standard Python patterns (Result dataclass, Exit codes, etc.)
- **Self-review before packaging**: See [references/synthesis-checklist.md](references/synthesis-checklist.md)
  for the 12-point quality checklist

These files contain established best practices for effective skill design.

#### Start with Reusable Skill Contents

To begin, start with the reusable resources identified above: `scripts/`,
`references/`, and `assets/` files.
Note that this step may require user input.
For example, when implementing a `brand-guidelines` skill,
the user may need to provide brand assets or templates.

Added scripts must be tested by actually running them to ensure no bugs.
If there are many similar scripts, only a representative sample needs testing.

Any example files not needed for the skill should be deleted.
The init script creates example files to demonstrate structure.

#### Update SKILL.md

**Writing Guidelines:** Always use imperative/infinitive form.

##### Frontmatter

Write the YAML frontmatter with required and optional fields:

**Required fields:**

- `name`: The skill name (hyphen-case)
- `description`: Primary trigger. Include what the skill does AND when to use it.

**Optional fields:**

- `metadata`: Additional skill configuration
  - `model` (recommended): Preferred model (e.g., `claude-sonnet-4-20250514`)
  - `allowed-tools`: List of tools the skill may use (optional)
- `context`: Set to `fork` to run in a forked sub-agent context
- `agent`: Agent type when `context: fork` (e.g., `Explore`, `Plan`, custom)
- `hooks`: Hooks scoped to this skill (PreToolUse, PostToolUse, Stop)

Example:

```yaml
---
name: docx-editor
description: Document editing with tracked changes. Use for .docx operations.
metadata:
  model: claude-sonnet-4-20250514
  allowed-tools:
    - Read
    - Write
    - Bash
---
```

Keep `description` comprehensive - it's the primary trigger.
The body only loads after triggering.

##### Body

Write instructions for using the skill and its bundled resources.

### Step 6: Packaging a Skill

Once development is complete, package the skill for distribution:

```bash
./run.sh scripts/package_skill.py <path/to/skill-folder> [output-dir] [--force]
```

The packaging script performs:

1. **Validation** - Checks frontmatter, naming, structure, Python syntax
2. **Skill Discovery** - Warns if similar skills exist (≥50% similarity)
3. **Evolution Scoring** - Calculates future-readiness score (recommended: 7+)
4. **Packaging** - Creates .skill file (zip format)

**Evolution Score Components** (see [references/evolution-scoring.md](references/evolution-scoring.md)):

- Extension points (2+): +2 points | No hardcoded versions: +2 points
- Design rationale (WHY): +2 points | Anti-patterns section: +2 points
- Base score: +2 points | Max: 10

Scores below 7 show warnings with suggestions but don't block packaging.
Use `--force` to skip confirmation prompts.

### Step 7: Iterate

After testing the skill, users may request improvements.
Often this happens right after using with fresh context of performance.

**Iteration workflow:**

1. Use the skill on real tasks
2. Notice struggles or inefficiencies
3. Identify how SKILL.md or bundled resources should be updated
4. Implement changes and test again

## Triggers

This skill activates when users want to:

- Create a new skill from scratch
- Update or improve an existing skill
- Package a skill for distribution
- Validate a skill's structure and quality
- Understand skill design best practices

## Anti-Patterns

Avoid these common mistakes when creating skills:

- **Overloading SKILL.md**: Too much detail in main file instead of references/
- **Missing triggers**: Not documenting when the skill should activate
- **Hardcoding versions**: Specifying exact versions instead of defaults
- **Ignoring evolution**: Building skills tightly coupled to implementations
- **Skipping validation**: Packaging without running quick_validate.py first
- **Duplicate functionality**: Creating skills that overlap with existing ones

## Extension Points

1. **Reference documents**: Add new reference files in `references/` for
   domain-specific guidance (e.g., `references/react-patterns.md`)
2. **Validation rules**: Extend `quick_validate.py` with custom checks
3. **Template customization**: Modify `init_skill.py` SKILL_TEMPLATE
4. **Scoring criteria**: Adjust weights in `score_evolution.py`

## Design Rationale

**Why 7-step workflow?** Skills require both understanding (Steps 1-3) and
implementation (Steps 4-7). The Analysis step was added for complex skills
with external integrations, where upfront design prevents costly rework.

**Why Evolution Scoring?** Skills become technical debt when tightly coupled to
specific versions or lack extensibility. The scoring system encourages
future-proof design without blocking packaging - warnings inform rather than enforce.

**Why Progressive Disclosure?** Context windows are limited. Loading all skill
content upfront wastes tokens. The three-tier system ensures efficient usage.

**Why validation patterns?** Consistent validation enables automation.
Standard patterns (ValidationResult, ExitCode) make scripts predictable.
