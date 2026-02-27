---
name: creating-skills
description: Expert knowledge on creating Agent Skills for Claude Code. Use when designing or creating SKILL.md files, understanding Skill structure, or implementing progressive disclosure patterns.
allowed-tools: Read, Grep, Glob
---

# Creating Agent Skills

This Skill provides comprehensive knowledge about creating effective Agent Skills in Claude Code.

## What Are Agent Skills?

Agent Skills are modular capabilities that extend Claude's functionality through organized folders containing instructions, scripts, and resources. Skills are **model-invoked** - Claude autonomously decides when to use them based on context and the Skill's description.

**Critical Insight**: Skills are not just passive knowledge bases - they are formalized, "battle-tested" process workflows or "playbooks". They define precisely how to execute a complex, multi-step task in a reliable and repeatable way. Think of them as procedural scripts that guide the agent through a specific workflow, not just reference documentation.

## Foundational Principle: Context Window as a Public Good

The context window is a **shared resource** that your Skill competes for with:
- User's current request
- Conversation history
- Other Skills that might be loaded
- System instructions

### Key Insight: Claude is Already Very Smart

**Default assumption**: Claude already knows programming concepts, common APIs, design patterns, etc.

**Only add** what Claude doesn't already have:
- Domain-specific knowledge (your company's architecture)
- Team conventions (your naming standards)
- Special workflows (your deployment process)

### Example

❌ **Bad** - Over-explaining basics:
```markdown
# Python Testing Skill

## What is pytest?
pytest is a testing framework for Python. Tests are functions that start with `test_`.
To run: `pytest`. Tests can use assertions: `assert x == y`.

[... 200 lines explaining pytest basics ...]
```

Claude already knows pytest!

✅ **Good** - Only your specifics:
```markdown
# Python Testing Skill

## Our Testing Conventions

- Tests in `tests/` mirror `src/` structure
- Use `@pytest.fixture` for database connections (see tests/conftest.py)
- Run full suite: `pytest --cov=src --cov-report=html`
- Pre-commit hook requires 80% coverage

Our custom fixtures:
- `db_session`: Test database with rollback
- `api_client`: Authenticated test client
```

### Conciseness Guidelines

Every token must justify its context cost:
- Skip basics Claude knows
- Focus on your specifics
- Link to external docs for general info
- Progressive disclosure for details

## How Skills Load: Three-Tier Architecture

Understanding how Skills load is key to writing efficient ones.

### The Three Tiers

**Tier 1: Metadata (Always Loaded)**
- Only `name` and `description` from YAML frontmatter
- Loaded at startup for ALL skills
- Cost: 30-50 tokens per skill
- Purpose: Skill discovery - helps Claude decide which skills to invoke

**Tier 2: Full Content (On-Demand)**
- Complete SKILL.md file loaded when Claude determines relevance
- Cost: Full file token count (aim for < 500 lines)
- Purpose: Provides the actual instructions/process

**Tier 3: Referenced Files (Selective)**
- Additional bundled files (reference.md, examples.md) loaded only when needed
- Cost: Only when explicitly referenced or needed
- Purpose: Detailed documentation without bloating main file

### Why This Matters

This architecture enables having **100+ skills available** without context bloat:
- 100 skills × 40 tokens = 4,000 tokens (Tier 1 metadata)
- Only 1-2 skills fully loaded per task (Tier 2)
- Referenced files loaded sparingly (Tier 3)

**Implication**: You can create many focused skills without worrying about startup cost. The metadata tier is lightweight.

### Example Load Sequence

```
User: "Generate API documentation"

Tier 1: Claude scans all skill descriptions (100 × 40 = 4k tokens)
  → Identifies "api-documentation" skill as relevant

Tier 2: Loads api-documentation/SKILL.md (450 lines = ~3k tokens)
  → Instructions say: "See examples.md for REST patterns"

Tier 3: Loads api-documentation/examples.md (200 lines = ~1.5k tokens)
  → Only when Claude needs REST examples

Total: 8.5k tokens (vs 100k+ if all skills fully loaded)
```

## File Structure

### Basic Skill (Instruction-Based)
```
.claude/skills/skill-name/
└── SKILL.md
```

### Advanced Skill (Code-Based with Resources)
```
.claude/skills/skill-name/
├── SKILL.md              # Main instructions
├── reference.md          # Detailed documentation
├── examples.md           # Usage examples
├── scripts/
│   └── helper.py        # Utility scripts
└── templates/
    └── template.txt     # Code templates
```

### Standard Directory Organization

For skills with multiple resources, use this three-directory pattern:

```
.claude/skills/my-skill/
├── SKILL.md                      # Main instructions (< 500 lines)
├── scripts/                      # Executables (output consumed as tokens)
│   ├── analyze.py                # Python/Bash scripts
│   └── process.sh                # Only OUTPUT loads to context
├── references/                   # Documentation (loaded on-demand)
│   ├── patterns.md               # Extended documentation
│   └── examples.md               # Full content loaded when needed
└── assets/                       # Templates and binaries
    ├── template.json             # Referenced by path
    └── config.yaml               # NOT loaded to context
```

### Directory Purposes

**scripts/** - Executables where only OUTPUT consumes tokens
- Python, Bash, Node.js scripts
- Claude runs them, only result goes to context
- Use for: data processing, API calls, calculations

**references/** - Documentation loaded into context on-demand
- Markdown files with detailed information
- Loaded fully when referenced
- Use for: extended guides, detailed patterns, examples

**assets/** - Templates and binaries referenced by path
- JSON templates, config files, images
- Referenced but never loaded into context
- Use for: templates to copy, configuration files, binary data

### Token Optimization

```
scripts/helper.py (500 lines):
  → Runs, only output (5 lines) → ~50 tokens

references/guide.md (500 lines):
  → Loaded fully when needed → ~3,500 tokens

assets/template.json (500 lines):
  → Referenced by path only → 0 tokens
```

**Use scripts/** to minimize token consumption - only output matters!

## Portable Resource References ({baseDir} Variable)

Use the **{baseDir} variable** for portable references to bundled resources.

### The Problem

Hardcoded paths break across environments:

❌ **Bad**:
```markdown
Run the helper script: `python /Users/alice/.claude/skills/my-skill/scripts/helper.py`
```

This breaks for other users (Bob's path is different).

### The Solution

Use `{baseDir}` which resolves to the skill's installation directory:

✅ **Good**:
```markdown
Run the helper script: `python {baseDir}/scripts/helper.py`
```

Works for all users regardless of installation location.

### Usage Patterns

**Script Execution:**
```markdown
python {baseDir}/scripts/init.py
bash {baseDir}/scripts/setup.sh
```

**Reading Reference Files:**
```markdown
See detailed patterns: Read({baseDir}/references/patterns.md)
```

**Template References:**
```markdown
Use template at: {baseDir}/assets/template.json
```

### Directory Organization Example

```
.claude/skills/my-skill/
├── SKILL.md                      # Use {baseDir} for references
├── scripts/
│   ├── helper.py                 # {baseDir}/scripts/helper.py
│   └── process.sh                # {baseDir}/scripts/process.sh
├── references/
│   └── patterns.md               # {baseDir}/references/patterns.md
└── assets/
    └── template.json             # {baseDir}/assets/template.json
```

### Best Practice

**Always** use {baseDir} for bundled resources. **Never** hardcode absolute paths.

## SKILL.md Format

### Required Structure
```yaml
---
name: skill-name
description: What this Skill does and when to use it. Include trigger keywords.
---

# Skill Name

## Instructions
Step-by-step guidance for Claude.

## Examples
Concrete usage examples.
```

### YAML Frontmatter Fields

**Required**:
- `name` - Lowercase letters, numbers, hyphens only (max 64 characters)
- `description` - What it does + when to use it (max 1024 characters)

**Optional**:
- `allowed-tools` - Comma-separated list of tools (e.g., `Read, Grep, Glob`)
  - If omitted, inherits all tools from conversation
  - Use to restrict capabilities (e.g., read-only Skills)
- `model` - Request specific Claude model for this skill (October 2025)
  - Shorthand: `haiku`, `sonnet`, `opus`, `inherit` (default)
  - Specific version: `claude-3-5-sonnet-20241022`, `claude-opus-4-20250514`
  - Use when skill needs more capable model than session default
  - Example: Security reviews may request `opus` for deepest reasoning

**⚠️ Note on 'when_to_use' field:**

The `when_to_use` field is **not officially documented** and may be deprecated. While it appears in some codebases, official Anthropic documentation (October 2025) recommends putting all discovery information in the `description` field instead.

**Recommended approach:**
```yaml
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
# Put ALL discovery info in description, not separate when_to_use field
---
```

**Avoid:**
```yaml
---
name: pdf-processing
description: PDF processing tools
when_to_use: Use when working with PDFs  # Undocumented, may not work
---
```

Rely on detailed `description` for skill discovery.

## Writing Effective Descriptions

The `description` field is **critical** for Claude to discover when to use your Skill.

### Good Descriptions (Specific + Triggers)
```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

```yaml
description: FastAPI endpoint patterns, dependency injection, error handling. Use when creating or modifying API routes, implementing authentication, or handling HTTP requests.
```

```yaml
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when working with Excel files, spreadsheets, or analyzing tabular data in .xlsx format.
```

### Bad Descriptions (Too Vague)
```yaml
description: Helps with documents  # ❌ What documents? When to use?
description: For data              # ❌ What kind of data? What operations?
description: Coding assistance     # ❌ What language? What tasks?
```

### Key Elements
1. **What it does** - List main capabilities
2. **When to use** - Specific triggers and keywords
3. **Context clues** - Technology names, file types, operations

### Example with Model Field

```yaml
---
name: security-review
description: Deep security analysis of authentication systems, cryptography, and access control. Use when reviewing security, auditing code for vulnerabilities, or analyzing authentication flows.
allowed-tools: Read, Grep, Glob
model: opus  # Request most capable model for critical task
---

# Security Review Process

This review requires deep reasoning about security implications.
Claude Opus provides the most thorough analysis for critical security tasks.

## Your Process
1. Analyze authentication mechanisms
2. Review authorization patterns
3. Check for common vulnerabilities (SQL injection, XSS, CSRF)
4. Verify cryptographic implementations
5. Assess session management
6. Check for information disclosure risks
```

**When to specify model:**
- Complex reasoning tasks → `opus`
- Fast, simple tasks → `haiku`
- Default/balanced → omit field or `inherit`

## Instruction-Based vs Code-Based Skills

### Instruction-Based (Most Common)
**Use when**: Providing guidance, patterns, best practices, analysis

**Example**: Code review checklist, testing guidelines, style guide

**Note**: Even instruction-based skills benefit from structure. They should define a clear process or workflow, not just provide information.
```yaml
---
name: code-review-checklist
description: Code review best practices and security checks. Use when reviewing code, checking PRs, or analyzing code quality.
---

# Code Review Checklist

## Security
- Check for exposed secrets or API keys
- Verify input validation
- Review authentication/authorization

## Quality
- Functions are well-named and focused
- No code duplication
- Proper error handling
```

### Code-Based (For Generation/Automation)
**Use when**: Generating code, running scripts, automating tasks

**Example**: Endpoint scaffolding, test generation, migration creation

### Hybrid Model (Most Robust)
**Use when**: You need both flexibility and reliability

The most robust approach combines both instruction-based and code-based elements:
- **LLM (Claude)** handles the "fuzzy" parts - choosing the right approach, reasoning about requirements
- **Scripts (Python/Bash)** handle rigid orchestration - ensuring steps happen in the right order with proper error handling

**Why Hybrid Works**:
- Pure English prompts can be inconsistent for complex workflows
- Pure code lacks flexibility for context-dependent decisions
- Combining them gives you "flexible orchestration with reliable execution"

**Implementation Pattern**:
```yaml
---
name: deployment-workflow
description: Orchestrates deployment with pre-flight checks. Use when deploying to production.
allowed-tools: Read, Write, Bash
---

# Deployment Workflow

## Your Process
1. Read the deployment configuration
2. Run pre-flight checks: `python scripts/preflight.py`
3. Analyze the script output and decide if deployment should proceed
4. If safe, execute deployment: `python scripts/deploy.py --env production`
5. Run post-deployment verification: `python scripts/verify.py`
6. Report results to user

The skill instructs Claude to run specific scripts at each step, while Claude uses reasoning to interpret results and make go/no-go decisions.
```
```
.claude/skills/scaffold-endpoint/
├── SKILL.md           # When/how to use the generator
├── scaffold.py        # Python script that generates code
└── templates/         # Jinja2 templates
    ├── model.py.jinja2
    └── endpoint.py.jinja2
```

**SKILL.md for code-based**:
```yaml
---
name: scaffold-endpoint
description: Generates complete REST endpoint with model, schema, and tests. Use when creating new API resources or adding CRUD operations.
allowed-tools: Read, Write, Bash
---

# Endpoint Scaffolding

## Usage
Run the scaffold script: `python scripts/scaffold.py [resource-name]`

## What It Generates
- SQLAlchemy model
- Pydantic schemas
- CRUD operations
- API endpoints
- Test fixtures

## Requirements
- Project uses FastAPI + SQLAlchemy
- Database connection configured
```

## Freedom Levels: Balancing Specificity and Flexibility

How prescriptive should your instructions be? Use the **Freedom Levels Framework**.

### The Three Levels

**High Freedom** - Broad guidance for multi-approach tasks
- Multiple valid approaches exist
- Claude should choose based on context
- Provide principles, not steps

**Example (High Freedom):**
```markdown
## API Design Principles
- RESTful conventions
- Consistent error handling
- Clear resource naming
```

**Medium Freedom** - Preferred patterns with variations allowed
- Established patterns exist
- Some adaptation expected
- Show preferred approach, allow alternatives

**Example (Medium Freedom):**
```markdown
## Recommended REST Endpoint Pattern

Prefer this structure:
- GET /resources - List
- GET /resources/{id} - Retrieve
- POST /resources - Create

Adapt if domain requires variations (hierarchical resources, etc.)
```

**Low Freedom** - Strict step-by-step for fragile operations
- Operations are error-prone
- Specific order matters
- No room for interpretation

**Example (Low Freedom):**
```markdown
## Database Migration Process (EXACT ORDER)

1. ALWAYS create backup first: `pg_dump > backup.sql`
2. Run migration in transaction: `BEGIN; ... COMMIT;`
3. NEVER skip rollback script: Create before applying
4. Verify data integrity: Run checks before and after
```

### Decision Guide

```
Is the operation error-prone or fragile?
├─ Yes → Low Freedom (strict steps)
└─ No → Are there multiple valid approaches?
    ├─ Yes → High Freedom (broad guidance)
    └─ No → Medium Freedom (preferred pattern)
```

### Best Practices

- **Default to Medium** - Provide guidance, allow adaptation
- **Use Low sparingly** - Only for truly fragile operations
- **High for creative** - Architecture, design, problem-solving
- **Document why** - Explain the freedom level choice

## Progressive Disclosure

For complex Skills, use multiple files to avoid overwhelming Claude with information.

**Critical Best Practice**: Keep your main SKILL.md file under **500 lines** or **5,000 words**. This is not just a suggestion - it's a key architectural decision.

**Why Two Metrics?**
- 500 lines: Easy to count, clear limit
- 5,000 words: Alternative for dense content
- Either signals it's time to split content

If you exceed both limits, you're definitely over the threshold for progressive disclosure.

**Why 500 Lines / 5,000 Words?**
- Beyond these limits, you risk context overload
- Large skills defeat the entire purpose of modular skills (loading only what you need)
- Claude's context window is valuable - don't waste it on unnecessary details

**How to Implement Progressive Disclosure**:

**SKILL.md** - Overview and common usage (< 500 lines):
```markdown
# PDF Processing

## Quick Start
Basic text extraction pattern.

## Advanced Usage
See [reference.md](reference.md) for:
- Form filling
- Table extraction
- Merging documents
```

**reference.md** - Detailed documentation (loaded only when needed):
```markdown
# PDF Processing Reference

## Form Filling
Detailed instructions for PDF form filling...

## Table Extraction
Advanced table extraction patterns...
```

Claude reads additional files only when it needs them, keeping context efficient and focused.

## Tool Restrictions with allowed-tools

Use `allowed-tools` to limit capabilities for security or focus.

### Read-Only Skill
```yaml
---
name: safe-file-reader
description: Read and analyze files without modifications. Use when you need read-only access.
allowed-tools: Read, Grep, Glob
---
```

### Limited Scope Skill
```yaml
---
name: data-analyzer
description: Analyze data files and generate reports. Use for data analysis tasks.
allowed-tools: Read, Bash(python:*), Write
---
```

### No Restrictions (Inherits All)
```yaml
---
name: full-access-skill
description: Complete access to all tools.
# allowed-tools field omitted - inherits all tools
---
```

## Validation Rules

### Name Rules
- ✅ `creating-skills`, `code-reviewer`, `test-generator`
- ❌ `Creating Skills` (no spaces or capitals)
- ❌ `create_skills` (use hyphens, not underscores)
- ❌ `this-is-a-very-long-skill-name-that-exceeds-sixty-four-characters` (too long)

### Description Rules
- ✅ Include what + when + triggers
- ✅ Specific technologies/file types mentioned
- ✅ Under 1024 characters
- ❌ Vague "helps with X"
- ❌ Missing when-to-use guidance
- ❌ Too long/verbose

### YAML Rules
- ✅ Opening `---` on line 1
- ✅ Closing `---` before Markdown
- ✅ Valid YAML syntax (no tabs)
- ❌ Missing delimiters
- ❌ Invalid field names
- ❌ Tabs instead of spaces

## Best Practices

### 1. Keep Skills Focused
**Good** - One capability:
- "PDF form filling"
- "Excel data analysis"
- "Git commit messages"

**Bad** - Too broad:
- "Document processing" (split into PDF, Word, Excel Skills)
- "Data tools" (split by data type)
- "All coding tasks" (way too broad)

### 2. Use Clear Triggers
Include keywords users would naturally mention:
```yaml
# Good - Multiple triggers
description: Analyze Excel spreadsheets (.xlsx, .xls), create pivot tables, generate charts. Use when working with spreadsheets, tabular data, or Excel files.

# Bad - No triggers
description: Data analysis tool
```

### 3. Start Simple, Add Complexity
Begin with basic SKILL.md, add reference files only if needed:
1. Start: Single SKILL.md (~100 lines)
2. If approaching 500 lines: Split to SKILL.md + reference.md
3. If needs automation: Add scripts/ directory
4. If needs templates: Add templates/ directory

**Rule of Thumb**: If your SKILL.md file is growing beyond 500 lines, it's time to refactor into multiple files or split into separate, focused skills.

### 4. Provide Examples
Include concrete examples in SKILL.md:
```markdown
## Examples

### Basic Usage
```python
import pdfplumber
with pdfplumber.open("doc.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

### Advanced Usage
See [examples.md](examples.md) for table extraction and form filling.
```

### 5. Document Dependencies
If Skill requires packages, list them:
```markdown
## Requirements

Packages must be installed:
```bash
pip install pypdf pdfplumber pandas
```
```

## Common Pitfalls and Anti-Patterns

### Anti-Pattern 1: The Monolithic Skill

**Problem**: Creating massive SKILL.md files (1000+ or even 1500+ lines) that contain everything about a domain.

**Why It's Bad**:
- Defeats the entire purpose of modular skills
- You wanted to load only what you need, but now you load everything
- Wastes precious context window space
- Makes skills hard to maintain and update
- Claude has to parse through irrelevant information

**Real-World Example**:
A user created a `frontend-dev-guidelines` skill that grew to 1500+ lines, covering React, Vue, TypeScript, CSS, testing, and deployment. When Claude loaded this skill for a simple React question, it consumed massive context for Vue and other irrelevant content.

**Solution**:
- Keep SKILL.md under 500 lines
- Split into focused skills: `react-patterns`, `vue-patterns`, `typescript-guidelines`
- Use progressive disclosure with reference files
- Ask yourself: "Does Claude need ALL of this information for ANY use of this skill?"

### Anti-Pattern 2: Knowledge Dump Instead of Process

**Problem**: Creating skills that are just reference documentation without a clear workflow.

**Why It's Bad**:
- Skills should be "playbooks" or "processes", not encyclopedias
- Without a clear workflow, Claude doesn't know what to do with the information
- Makes skills less actionable and repeatable

**Example of Bad Skill**:
```markdown
# Python Best Practices

Here are all the Python best practices...
[500 lines of various tips and tricks with no structure]
```

**Example of Good Skill**:
```markdown
# Python Code Review Process

## Your Workflow
1. Read the changed files
2. Check for PEP 8 compliance using the checklist below
3. Verify error handling patterns
4. Check for security issues
5. Provide structured feedback in this format: [...]

## PEP 8 Checklist
[Specific, actionable items]

## Security Checklist
[Specific, actionable items]
```

### Anti-Pattern 3: Ignoring the Hybrid Model for Complex Workflows

**Problem**: Trying to orchestrate complex, multi-step workflows using only English prompts.

**Why It's Bad**:
- English-only orchestration can be inconsistent
- Complex state management is error-prone in pure prompts
- Hard to ensure steps happen in the correct order

**Solution**: Use the hybrid model - prompts for reasoning, scripts for orchestration.

## Common Patterns

### Analysis/Review Skills
```yaml
---
name: security-reviewer
description: Security vulnerability analysis for code. Use when reviewing security, checking authentication, or analyzing API endpoints.
allowed-tools: Read, Grep, Glob
---

# Security Review Checklist
- Authentication checks
- Input validation
- SQL injection risks
...
```

### Generation/Scaffolding Skills
```yaml
---
name: test-generator
description: Generate pytest test fixtures and test cases. Use when creating tests or adding test coverage.
allowed-tools: Read, Write, Bash
---

# Test Generation

Run: `python scripts/generate_test.py [module-name]`

Generates:
- Test fixtures
- Unit tests
- Integration tests
```

### Pattern/Convention Skills
```yaml
---
name: api-patterns
description: REST API design patterns and conventions. Use when creating API endpoints or designing REST interfaces.
allowed-tools: Read, Grep
---

# API Design Patterns

## Endpoint Structure
Standard CRUD pattern:
- GET /resources - List
- GET /resources/{id} - Retrieve
- POST /resources - Create
- PUT /resources/{id} - Update
- DELETE /resources/{id} - Delete
```

## Testing Skills

Test your Skills by asking questions that match the description:

**If description mentions "PDF files"**:
> "Can you help me extract text from this PDF?"

**If description mentions "API endpoints"**:
> "Create a new REST endpoint for products"

Claude should autonomously invoke your Skill based on the context.

## Troubleshooting

### Skill Not Invoked
**Problem**: Claude doesn't use the Skill

**Check**:
1. Is description specific enough? (Not "helps with data")
2. Does description include trigger keywords? (file types, operations)
3. Is YAML valid? (Check `---` delimiters)
4. Does file exist at correct path? (`.claude/skills/{name}/SKILL.md`)

### Skill Loaded But Not Helpful
**Problem**: Skill is used but doesn't provide useful guidance

**Check**:
1. Are instructions clear and actionable?
2. Are examples concrete? (Not "do X", but "use this code...")
3. Is Skill too broad? (Split into focused Skills)
4. Should it reference additional files? (Use progressive disclosure)

## Summary

**Essential Elements**:
1. Clear, focused purpose (one capability)
2. Specific description with triggers
3. Actionable instructions with examples
4. Valid YAML frontmatter
5. Appropriate tool restrictions

**Success Criteria**:
- Claude invokes Skill at the right time
- Instructions are clear and followable
- Examples are helpful and concrete
- Skill integrates well with others
