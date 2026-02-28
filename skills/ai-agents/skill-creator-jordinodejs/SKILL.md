---
name: skill-creator
description: Guide for creating effective skills in a Node.js/JavaScript environment. This skill should be used when users want to create a new skill (or update an existing skill) that extends Claude's capabilities with specialized knowledge, workflows, or tool integrations for modern JavaScript development.
license: Complete terms in LICENSE.txt
---

# Skill Creator - Node.js & JavaScript Edition

This skill provides guidance for creating effective skills tailored to Node.js and modern JavaScript ecosystems.

## About Skills

Skills are modular, self-contained packages that extend Claude's capabilities by providing
specialized knowledge, workflows, and tools. Think of them as "onboarding guides" for specific
domains or tasks—they transform Claude from a general-purpose agent into a specialized agent
equipped with procedural knowledge that no model can fully possess.

### What Skills Provide

1. Specialized workflows - Multi-step procedures for specific domains
2. Tool integrations - Instructions for working with specific file formats or APIs
3. Domain expertise - Company-specific knowledge, schemas, business logic
4. Bundled resources - Scripts, references, and assets for complex and repetitive tasks

## Core Principles

### Concise is Key

The context window is a public good. Skills share the context window with everything else Claude needs: system prompt, conversation history, other Skills' metadata, and the actual user request.

**Default assumption: Claude is already very smart.** Only add context Claude doesn't already have. Challenge each piece of information: "Does Claude really need this explanation?" and "Does this paragraph justify its token cost?"

Prefer concise examples over verbose explanations.

### Set Appropriate Degrees of Freedom

Match the level of specificity to the task's fragility and variability:

**High freedom (text-based instructions)**: Use when multiple approaches are valid, decisions depend on context, or heuristics guide the approach.

**Medium freedom (pseudocode or scripts with parameters)**: Use when a preferred pattern exists, some variation is acceptable, or configuration affects behavior.

**Low freedom (specific scripts, few parameters)**: Use when operations are fragile and error-prone, consistency is critical, or a specific sequence must be followed.

Think of Claude as exploring a path: a narrow bridge with cliffs needs specific guardrails (low freedom), while an open field allows many routes (high freedom).

### Anatomy of a Skill

Every skill consists of a required SKILL.md file and optional bundled resources:

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter metadata (required)
│   │   ├── name: (required)
│   │   └── description: (required)
│   └── Markdown instructions (required)
└── Bundled Resources (optional)
    ├── scripts/          - Executable code (Python/Bash/etc.)
    ├── references/       - Documentation intended to be loaded into context as needed
    └── assets/           - Files used in output (templates, icons, fonts, etc.)
```

#### SKILL.md (required)

Every SKILL.md consists of:

- **Frontmatter** (YAML): Contains `name` and `description` fields. These are the only fields that Claude reads to determine when the skill gets used, thus it is very important to be clear and comprehensive in describing what the skill is, and when it should be used.
- **Body** (Markdown): Instructions and guidance for using the skill. Only loaded AFTER the skill triggers (if at all).

#### Bundled Resources (optional)

##### Scripts (`scripts/`)

Executable JavaScript/TypeScript code for Node.js or browser tasks that require deterministic reliability or are repeatedly rewritten. Can be CLI utilities, utilities run with `pnpm dlx tsx`, or automation scripts.

- **When to include**: When the same code is being rewritten repeatedly or deterministic reliability is needed
- **Example**: `scripts/migrate-db.ts` for database migrations, `scripts/optimize-images.ts` for batch image optimization
- **File formats**: `.ts` (TypeScript), `.mts` (ES modules), `.js` files
- **Benefits**: Token efficient, deterministic, can be executed via `pnpm dlx tsx` without loading full context
- **Note**: Scripts may still need to be read by Claude for patching or environment-specific adjustments
- **Best practice**: Use TypeScript for type safety and IDE support when possible

##### References (`references/`)

Documentation and reference material intended to be loaded as needed into context to inform Claude's process and thinking.

- **When to include**: For documentation that Claude should reference while working
- **Examples**: `references/finance.md` for financial schemas, `references/mnda.md` for company NDA template, `references/policies.md` for company policies, `references/api_docs.md` for API specifications
- **Use cases**: Database schemas, API documentation, domain knowledge, company policies, detailed workflow guides
- **Benefits**: Keeps SKILL.md lean, loaded only when Claude determines it's needed
- **Best practice**: If files are large (>10k words), include grep search patterns in SKILL.md
- **Avoid duplication**: Information should live in either SKILL.md or references files, not both. Prefer references files for detailed information unless it's truly core to the skill—this keeps SKILL.md lean while making information discoverable without hogging the context window. Keep only essential procedural instructions and workflow guidance in SKILL.md; move detailed reference material, schemas, and examples to references files.

##### Assets (`assets/`)

Files not intended to be loaded into context, but rather used within the output Claude produces. Common in Node.js/JS projects: boilerplate code, templates, configuration files, and generated assets.

- **When to include**: When the skill needs files that will be used in the final output
- **Examples**: `assets/nextjs-template/` for Next.js boilerplate, `assets/tailwind-config.ts` for Tailwind configuration templates, `assets/auth-middleware.ts` for middleware patterns, `assets/db-schema.sql` for database templates
- **Use cases**: Project templates, configuration templates, boilerplate code, reusable middleware, database schemas, environment templates
- **Benefits**: Separates output resources from documentation, enables Claude to use files without loading them into context

#### What to Not Include in a Skill

A skill should only contain essential files that directly support its functionality. Do NOT create extraneous documentation or auxiliary files, including:

- README.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md
- etc.

The skill should only contain the information needed for an AI agent to do the job at hand. It should not contain auxilary context about the process that went into creating it, setup and testing procedures, user-facing documentation, etc. Creating additional documentation files just adds clutter and confusion.

### Progressive Disclosure Design Principle

Skills use a three-level loading system to manage context efficiently:

1. **Metadata (name + description)** - Always in context (~100 words)
2. **SKILL.md body** - When skill triggers (<5k words)
3. **Bundled resources** - As needed by Claude (Unlimited because scripts can be executed without reading into context window)

#### Progressive Disclosure Patterns

Keep SKILL.md body to the essentials and under 500 lines to minimize context bloat. Split content into separate files when approaching this limit. When splitting out content into other files, it is very important to reference them from SKILL.md and describe clearly when to read them, to ensure the reader of the skill knows they exist and when to use them.

**Key principle:** When a skill supports multiple variations, frameworks, or options, keep only the core workflow and selection guidance in SKILL.md. Move variant-specific details (patterns, examples, configuration) into separate reference files.

**Pattern 1: High-level guide with references**

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

**Pattern 2: Domain-specific organization**

For Skills with multiple domains, organize content by domain to avoid loading irrelevant context:

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

When a user asks about sales metrics, Claude only reads sales.md.

Similarly, for skills supporting multiple frameworks or variants, organize by variant:

```
cloud-deploy/
├── SKILL.md (workflow + provider selection)
└── references/
    ├── aws.md (AWS deployment patterns)
    ├── gcp.md (GCP deployment patterns)
    └── azure.md (Azure deployment patterns)
```

When the user chooses AWS, Claude only reads aws.md.

**Pattern 3: Conditional details**

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

Claude reads REDLINING.md or OOXML.md only when the user needs those features.

**Important guidelines:**

- **Avoid deeply nested references** - Keep references one level deep from SKILL.md. All reference files should link directly from SKILL.md.
- **Structure longer reference files** - For files longer than 100 lines, include a table of contents at the top so Claude can see the full scope when previewing.

## Skill Creation Process

Skill creation involves these steps:

1. Understand the skill with concrete examples
2. Plan reusable skill contents (scripts, references, assets)
3. Initialize the skill (run init_skill.py)
4. Edit the skill (implement resources and write SKILL.md)
5. Package the skill (run package_skill.py)
6. Iterate based on real usage

Follow these steps in order, skipping only if there is a clear reason why they are not applicable.

### Step 1: Understanding the Skill with Concrete Examples

Skip this step only when the skill's usage patterns are already clearly understood. It remains valuable even when working with an existing skill.

To create an effective skill, clearly understand concrete examples of how the skill will be used. This understanding can come from either direct user examples or generated examples that are validated with user feedback.

For example, when building a `nextjs-api-skill`, relevant questions include:

- "What API patterns should this skill support? Route handlers, middleware, validation?"
- "Can you give some examples of how this skill would be used?"
- "I can imagine users asking for things like 'Create a protected API route' or 'Add input validation to this endpoint'. Are there other ways you imagine this skill being used?"
- "What would a user say that should trigger this skill?"

To avoid overwhelming users, avoid asking too many questions in a single message. Start with the most important questions and follow up as needed for better effectiveness.

Conclude this step when there is a clear sense of the functionality the skill should support.

### Step 2: Planning the Reusable Skill Contents

To turn concrete examples into an effective skill, analyze each example by:

1. Considering how to execute on the example from scratch
2. Identifying what scripts, references, and assets would be helpful when executing these workflows repeatedly

Example: When building a `database-migration-skill` to handle queries like "Create a migration for adding a users table," the analysis shows:

1. Writing migrations requires the same TypeScript boilerplate each time
2. A `scripts/generate-migration.ts` script and `assets/migration-template.ts` template would be helpful to store in the skill

Example: When designing an `api-development-skill` for queries like "Build a REST API endpoint" or "Add authentication to my API," the analysis shows:

1. API development requires the same middleware and routing patterns each time
2. `assets/auth-middleware.ts` and `assets/route-handler-template.ts` templates would be helpful to store in the skill

Example: When building a `next-js-optimization-skill` to handle queries like "What's slowing down my Next.js app?" the analysis shows:

1. Analyzing Next.js performance requires understanding the same patterns, configs, and metrics each time
2. A `references/performance-patterns.md` file documenting common bottlenecks would be helpful to store in the skill

To establish the skill's contents, analyze each concrete example to create a list of the reusable resources to include: scripts, references, and assets.

### Step 3: Initializing the Skill

At this point, it is time to actually create the skill.

Skip this step only if the skill being developed already exists, and iteration or packaging is needed. In this case, continue to the next step.

When creating a new skill from scratch, initialize it with the following structure:

```
skill-name/
├── SKILL.md (required)
├── scripts/
│   └── example.ts
├── references/
│   └── example.md
└── assets/
    └── template-example.ts
```

Create the SKILL.md with proper frontmatter and TODO placeholders:

```yaml
---
name: your-skill-name
description: Brief description of what this skill does and when to use it for Node.js/JavaScript development.
license: MIT or other appropriate license
---

# Your Skill Name

[Implementation details...]
```

Then customize or remove generated example files as needed.

### Step 4: Edit the Skill

When editing the (newly-generated or existing) skill, remember that the skill is being created for another instance of Claude to use. Include information that would be beneficial and non-obvious to Claude. Consider what procedural knowledge, domain-specific details, or reusable assets would help another Claude instance execute these tasks more effectively.

#### Learn Proven Design Patterns

Consult these helpful guides based on your skill's needs:

- **Multi-step processes**: See references/workflows.md for sequential workflows and conditional logic
- **Specific output formats or quality standards**: See references/output-patterns.md for template and example patterns

These files contain established best practices for effective skill design.

#### Start with Reusable Skill Contents

To begin implementation, start with the reusable resources identified above: `scripts/`, `references/`, and `assets/` files. Note that this step may require user input. For example, when implementing a `typescript-patterns` skill, the user may need to provide code examples to store in `assets/`, or documentation to store in `references/`.

Added scripts must be tested by actually running them to ensure there are no bugs and that the output matches what is expected. For example:

- Test TypeScript scripts with `pnpm dlx tsx scripts/your-script.ts`
- Verify generated code compiles with `tsc --noEmit`
- Run any bundled utilities to ensure they work

If there are many similar scripts, only a representative sample needs to be tested to ensure confidence that they all work while balancing time to completion.

Any example files and directories not needed for the skill should be deleted.

#### Update SKILL.md

**Writing Guidelines:** Always use imperative/infinitive form.

##### Frontmatter

Write the YAML frontmatter with `name` and `description`:

- `name`: The skill name
- `description`: This is the primary triggering mechanism for your skill, and helps Claude understand when to use the skill.
  - Include both what the Skill does and specific triggers/contexts for when to use it.
  - Include all "when to use" information here - Not in the body. The body is only loaded after triggering, so "When to Use This Skill" sections in the body are not helpful to Claude.
  - Example description for a `docx` skill: "Comprehensive document creation, editing, and analysis with support for tracked changes, comments, formatting preservation, and text extraction. Use when Claude needs to work with professional documents (.docx files) for: (1) Creating new documents, (2) Modifying or editing content, (3) Working with tracked changes, (4) Adding comments, or any other document tasks"

Do not include any other fields in YAML frontmatter.

##### Body

Write instructions for using the skill and its bundled resources.

### Step 5: Packaging a Skill

Once development of the skill is complete, it should be organized into a distributable directory structure. For Node.js/JavaScript skills in this environment:

1. Ensure all TypeScript files are properly typed
2. Add a `package.json` with dependencies if scripts have external requirements (e.g., `import axios from 'axios'`)
3. Document how to run scripts (e.g., `pnpm dlx tsx scripts/your-script.ts`)
4. Organize assets with clear folder structure (templates, config examples, etc.)
5. Create clear references documentation with examples

Directory structure example:

```
my-skill/
├── SKILL.md (required)
├── package.json (if needed for script dependencies)
├── scripts/
│   ├── setup.ts
│   └── migrate.ts
├── references/
│   ├── api-patterns.md
│   └── best-practices.md
└── assets/
    ├── templates/
    │   ├── route-handler.ts
    │   └── middleware.ts
    └── config/
        └── tsconfig.example.json
```

The skill is ready to share when:
- SKILL.md is complete and properly formatted with frontmatter
- All scripts are tested and working
- References are well-documented with examples
- Assets are organized logically

### Step 6: Iterate

After testing the skill, users may request improvements. Often this happens right after using the skill, with fresh context of how the skill performed.

**Iteration workflow for Node.js/JavaScript skills:**

1. Use the skill on real tasks (e.g., creating API routes, optimizing performance, refactoring code)
2. Notice struggles or inefficiencies
3. Test edge cases and improve script robustness
4. Identify how SKILL.md or bundled resources should be updated
5. Update documentation with new patterns discovered
6. Add reference examples for common use cases
7. Re-test and validate all changes

**Common improvements:**
- Add more TypeScript examples to assets when new patterns emerge
- Expand references with edge cases and troubleshooting
- Refactor scripts to handle more scenarios
- Add environment-specific guidance for different setups (local dev, CI/CD, Vercel, etc.)
