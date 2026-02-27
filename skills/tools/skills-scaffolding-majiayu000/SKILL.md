---
name: skills-scaffolding
description: Guide for creating effective Claude Code skills. This skill should be used when users want to create a new skill (or update an existing skill) that extends Claude's capabilities with specialized knowledge, workflows, or tool integrations.
---

# Skills Scaffold Skill

This skill provides guidance for creating effective Claude Code skills.

## About Skills

Skills are modular, self-contained packages that extend Claude's capabilities by providing specialized knowledge, workflows, and tools. Think of them as "onboarding guides" for specific domains or tasks—they transform Claude from a general-purpose agent into a specialized agent equipped with procedural knowledge that no model can fully possess.

### What Skills Provide

1. Specialized workflows - Multi-step procedures for specific domains
2. Tool integrations - Instructions for working with specific file formats or APIs
3. Domain expertise - Company-specific knowledge, schemas, business logic
4. Bundled resources - Scripts, references, and assets for complex and repetitive tasks

## When to Use This Skill

This skill should be used when:
- Creating a new Claude Code skill from scratch
- Updating an existing skill to improve effectiveness
- Structuring skill directories and files
- Writing YAML frontmatter for skills
- Organizing bundled resources (scripts, references, assets)
- Understanding progressive disclosure patterns
- Determining if a skill is the right solution vs a slash command

## Skills vs Commands

### Skills
- **Purpose:** Provide just-in-time expert guidance for specialized, repeatable tasks
- **Activation:** Automatically loaded when Claude detects relevant context
- **Token Efficiency:** Uses progressive disclosure - only metadata loaded initially
- **Scope:** Domain-specific procedural knowledge (e.g., brand guidelines, work item creation)
- **Location:** `skills/{skill-name}/SKILL.md` directory
- **Best For:** Ongoing expertise that applies across multiple sessions

### Slash Commands
- **Purpose:** Execute one-time setup or configuration tasks
- **Activation:** Explicitly invoked by user with `/command-name`
- **Token Efficiency:** Full command loaded when invoked
- **Scope:** Interactive workflows with user input (e.g., initialization, configuration)
- **Location:** `commands/{command-name}.md` file
- **Best For:** Setup tasks, scaffolding, project initialization

**Decision Guideline:** Use a skill for providing ongoing expertise or guidelines. Use a command for one-time interactive setup.

## Anatomy of a Skill

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
    ├── references/       - Documentation loaded into context as needed
    └── assets/           - Files used in output (templates, icons, fonts, etc.)
```

**Critical Requirements:**
- The skill directory must contain a `SKILL.md` file (case-sensitive)
- The `SKILL.md` file must begin with YAML frontmatter containing `name` and `description`
- Bundled resources are optional but should be organized by type

### SKILL.md (required)

**Metadata Quality:** The `name` and `description` in YAML frontmatter determine when Claude will use the skill. Be specific about what the skill does and when to use it. Use third-person form (e.g., "This skill should be used when..." instead of "Use this skill when...").

### Bundled Resources (optional)

#### Scripts (`scripts/`)

Executable code (Python/Bash/etc.) for tasks that require deterministic reliability or are repeatedly rewritten.

- **When to include**: When the same code is being rewritten repeatedly or deterministic reliability is needed
- **Example**: `scripts/rotate_pdf.py` for PDF rotation tasks
- **Benefits**: Token efficient, deterministic, may be executed without loading into context
- **Note**: Scripts may still need to be read by Claude for patching or environment-specific adjustments

#### References (`references/`)

Documentation and reference material intended to be loaded as needed into context to inform Claude's process and thinking.

- **When to include**: For documentation that Claude should reference while working
- **Examples**: `references/finance.md` for financial schemas, `references/api_docs.md` for API specifications, `references/policies.md` for company policies
- **Use cases**: Database schemas, API documentation, domain knowledge, company policies, detailed workflow guides
- **Benefits**: Keeps SKILL.md lean, loaded only when Claude determines it's needed
- **Best practice**: If files are large (>10k words), include grep search patterns in SKILL.md
- **Avoid duplication**: Information should live in either SKILL.md or references files, not both. Prefer references files for detailed information unless it's truly core to the skill—this keeps SKILL.md lean while making information discoverable without hogging the context window. Keep only essential procedural instructions and workflow guidance in SKILL.md; move detailed reference material, schemas, and examples to references files.

#### Assets (`assets/`)

Files not intended to be loaded into context, but rather used within the output Claude produces.

- **When to include**: When the skill needs files that will be used in the final output
- **Examples**: `assets/logo.png` for brand assets, `assets/slides.pptx` for PowerPoint templates, `assets/frontend-template/` for HTML/React boilerplate, `assets/font.ttf` for typography
- **Use cases**: Templates, images, icons, boilerplate code, fonts, sample documents that get copied or modified
- **Benefits**: Separates output resources from documentation, enables Claude to use files without loading them into context

## YAML Frontmatter Specifications

The `SKILL.md` file must begin with YAML frontmatter containing required and optional fields:

### Required Fields

```yaml
---
name: Skill Name Here
description: Clear explanation of what this skill does and when to use it
---
```

**Field Specifications:**

- **name** (required, max 64 characters)
  - Human-friendly identifier for the skill
  - Use title case (e.g., "Azure DevOps Work Items")
  - Should clearly indicate the skill's purpose
  - Examples: "Brand Guidelines", "PDF Creator", "Excel Analyzer"

- **description** (required, max 200 characters)
  - **CRITICAL:** Claude uses this to determine when to invoke the skill
  - Should clearly state what the skill does AND when to use it
  - Include relevant keywords that Claude can match against user requests
  - Use third-person form (e.g., "This skill should be used when..." instead of "Use this skill when...")
  - Be specific about the context where this skill applies

**Good Description Examples:**
```yaml
description: Guide for creating and managing Azure DevOps work items (Features, User Stories, Tasks) using proper hierarchy, naming conventions, and formatting. This skill should be used when creating ADO work items.
```

```yaml
description: Guide for applying company brand guidelines to documents. This skill should be used when ensuring consistent colors, fonts, logos, and tone across marketing materials.
```

**Bad Description Examples:**
```yaml
description: Helps with Azure DevOps  # Too vague, doesn't specify what it helps with
```

```yaml
description: Work items  # Too short, no context, no third-person form
```

### Optional Fields

```yaml
---
name: Skill Name
description: What it does and when to use it
dependencies: python>=3.8, pandas>=1.5.0
---
```

- **version** (optional)
  - Track iterations using semantic versioning (e.g., 1.0.0)
  - Useful for maintaining skill history

- **dependencies** (optional)
  - Required software packages or tools
  - Examples: `python>=3.8, pandas>=1.5.0`, `node>=18.0.0`
  - Note: For API skills, dependencies must be pre-installed in the container

## Progressive Disclosure Design Principle

Skills use a three-level loading system to manage context efficiently:

1. **Metadata (name + description)** - Always in context (~100 words)
2. **SKILL.md body** - When skill triggers (<5k words)
3. **Bundled resources** - As needed by Claude (Unlimited*)

*Unlimited because scripts can be executed without reading into context window.

**Design Principle:** Structure the skill so that SKILL.md body provides everything needed for common cases, with bundled resources (references/, scripts/, assets/) for detailed specifications, deterministic operations, and output files.

## Writing Effective Skill Content

**Writing Style:** Write the entire skill using **imperative/infinitive form** (verb-first instructions), not second person. Use objective, instructional language (e.g., "To accomplish X, do Y" rather than "You should do X" or "If you need to do X"). This maintains consistency and clarity for AI consumption.

The markdown content in SKILL.md should answer these key questions:

1. **What is the purpose of the skill?** (a few sentences)
2. **When should the skill be used?** (specific use cases)
3. **How should Claude use the skill in practice?** (procedural instructions)

### Recommended Structure

```markdown
# Skill Name

This skill provides [brief description of capability].

## When to Use This Skill

This skill should be used when:
- [Specific use case 1]
- [Specific use case 2]
- [Specific use case 3]

## Core Guidelines

### Primary Topic

To accomplish [task], follow these steps:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Key Rules:**
- [Rule 1 with rationale]
- [Rule 2 with rationale]

### Using Bundled Resources

**Scripts:** To [perform deterministic task], execute `scripts/script_name.py`

**References:** For detailed [specifications/schemas/documentation], read `references/reference_name.md`

**Assets:** To [create output], use the template at `assets/template_name.ext`

## Examples

**Good approach:**
```
[Example of correct approach]
```

**Bad approach:**
```
[Example of incorrect approach]
```

## Best Practices

1. **Practice Name:** Why this matters and how to apply it
2. **Practice Name:** Why this matters and how to apply it
```

## Naming Conventions

### Skill Directory Names
- Use kebab-case (lowercase with hyphens)
- Be descriptive and specific
- Examples: `ado-work-items`, `brand-guidelines`, `pdf-creator`

### Skill Names (in YAML)
- Use title case
- Be clear and descriptive
- Examples: "Azure DevOps Work Items", "Brand Guidelines", "PDF Creator"

## Supporting Files

### When to Use References Files
Create files in the `references/` directory when:
- You have detailed specifications that aren't needed for common cases
- The core SKILL.md would become too long (>5k words)
- You need to separate high-level guidance from detailed reference
- You have documentation that Claude should read only when needed

**Reference from SKILL.md:**
```markdown
For detailed field specifications, read references/field_specs.md.
```

### When to Use Scripts
Include executable scripts when:
- The task requires deterministic operations (sorting, parsing, calculations)
- Code provides more reliability than LLM generation
- You want to bundle pre-tested utilities with the skill

**Script Organization:**
```
skills/my-skill/
  SKILL.md
  scripts/
    parse_data.py      # Python utilities
    format_output.js   # JavaScript utilities
```

**Reference from SKILL.md:**
```markdown
To parse the data, use the provided Python script at scripts/parse_data.py.
```

## Skill Creation Process

To create a skill, follow the "Skill Creation Process" in order, skipping steps only if there is a clear reason why they are not applicable.

### Step 1: Understanding the Skill with Concrete Examples

Skip this step only when the skill's usage patterns are already clearly understood. It remains valuable even when working with an existing skill.

To create an effective skill, clearly understand concrete examples of how the skill will be used. This understanding can come from either direct user examples or generated examples that are validated with user feedback.

For example, when building an image-editor skill, relevant questions include:

- "What functionality should the image-editor skill support? Editing, rotating, anything else?"
- "Can you give some examples of how this skill would be used?"
- "I can imagine users asking for things like 'Remove the red-eye from this image' or 'Rotate this image'. Are there other ways you imagine this skill being used?"
- "What would a user say that should trigger this skill?"

To avoid overwhelming users, avoid asking too many questions in a single message. Start with the most important questions and follow up as needed for better effectiveness.

Conclude this step when there is a clear sense of the functionality the skill should support.

### Step 2: Planning the Reusable Skill Contents

To turn concrete examples into an effective skill, analyze each example by:

1. Considering how to execute on the example from scratch
2. Identifying what scripts, references, and assets would be helpful when executing these workflows repeatedly

**Example:** When building a `pdf-editor` skill to handle queries like "Help me rotate this PDF," the analysis shows:
1. Rotating a PDF requires re-writing the same code each time
2. A `scripts/rotate_pdf.py` script would be helpful to store in the skill

**Example:** When designing a `frontend-webapp-builder` skill for queries like "Build me a todo app" or "Build me a dashboard to track my steps," the analysis shows:
1. Writing a frontend webapp requires the same boilerplate HTML/React each time
2. An `assets/hello-world/` template containing the boilerplate HTML/React project files would be helpful to store in the skill

**Example:** When building a `big-query` skill to handle queries like "How many users have logged in today?" the analysis shows:
1. Querying BigQuery requires re-discovering the table schemas and relationships each time
2. A `references/schema.md` file documenting the table schemas would be helpful to store in the skill

To establish the skill's contents, analyze each concrete example to create a list of the reusable resources to include: scripts, references, and assets.

### Step 3: Create the Skill Directory

Create the skill directory structure:

```bash
mkdir -p skills/{skill-name}
mkdir -p skills/{skill-name}/scripts    # If needed
mkdir -p skills/{skill-name}/references # If needed
mkdir -p skills/{skill-name}/assets     # If needed
```

**Plugin Context:** When creating a skill within a Claude Code plugin, place it in `plugins/{plugin-name}/skills/{skill-name}/`

### Step 4: Edit the Skill

When editing the (newly-generated or existing) skill, remember that the skill is being created for another instance of Claude to use. Focus on including information that would be beneficial and non-obvious to Claude. Consider what procedural knowledge, domain-specific details, or reusable assets would help another Claude instance execute these tasks more effectively.

#### Start with Reusable Skill Contents

To begin implementation, start with the reusable resources identified in Step 2: `scripts/`, `references/`, and `assets/` files. Note that this step may require user input. For example, when implementing a `brand-guidelines` skill, the user may need to provide brand assets or templates to store in `assets/`, or documentation to store in `references/`.

#### Create SKILL.md

**Writing Style:** Write the entire skill using **imperative/infinitive form** (verb-first instructions), not second person. Use objective, instructional language.

To complete SKILL.md, answer the following questions:

1. What is the purpose of the skill, in a few sentences?
2. When should the skill be used?
3. In practice, how should Claude use the skill? All reusable skill contents developed above should be referenced so that Claude knows how to use them.

### Step 5: Test the Skill

After creating the skill, test it with realistic scenarios:

**Validation checklist:**
- Is the description specific enough for Claude to know when to load it?
- Does the SKILL.md body provide everything needed for common cases?
- Are bundled resources properly referenced in SKILL.md?
- Are examples clear and actionable?
- Is the skill focused on one domain/task, or too broad?

**Plugin Context:** If creating a skill within a Claude Code plugin, update plugin documentation to mention the skill.

### Step 6: Iterate

After testing the skill, users may request improvements. Often this happens right after using the skill, with fresh context of how the skill performed.

**Iteration workflow:**
1. Use the skill on real tasks
2. Notice struggles or inefficiencies
3. Identify how SKILL.md or bundled resources should be updated
4. Implement changes and test again

## Common Patterns

### Pattern: Domain Expertise Skill

**Use Case:** Providing expert knowledge for a specific domain (e.g., ADO work items, brand guidelines)

**Structure:**
```yaml
---
name: Domain Name Expert
description: Expert guidance for [specific task] using [domain-specific concepts and rules]
---

# Domain Name Expert Skill

This skill provides expert guidance on [domain].

## When to Use This Skill

Invoke this skill when:
- Working with [domain-specific tools]
- Creating [domain-specific artifacts]
- Following [domain-specific processes]

## Core Concepts

[Domain knowledge organized by topic]

## Best Practices

[Domain-specific best practices]
```

### Pattern: Document Processing Skill

**Use Case:** Working with specific file formats (PDF, Excel, Word)

**Structure:**
```yaml
---
name: Format Name Processor
description: Create, read, and manipulate [format] files with proper formatting and structure
dependencies: [required packages]
---

# Format Name Processor Skill

This skill helps you work with [format] files.

## When to Use This Skill

Invoke this skill when:
- Creating new [format] documents
- Reading data from [format] files
- Modifying existing [format] documents

## File Format Specifications

[Format-specific details]

## Common Operations

[How to perform common tasks]

## Example Scripts

Use the provided scripts at scripts/[script-name] for [purpose].
```

### Pattern: Workflow Automation Skill

**Use Case:** Standardizing repeatable workflows (e.g., meeting notes, reports)

**Structure:**
```yaml
---
name: Workflow Name Automation
description: Automate [workflow name] by following company standards for [specific outputs]
---

# Workflow Name Automation Skill

This skill automates [workflow].

## When to Use This Skill

Invoke this skill when:
- Starting a new [workflow instance]
- Following the standard [process name]

## Workflow Steps

1. **Step Name:** Description and guidelines
2. **Step Name:** Description and guidelines

## Templates

[Provide templates or examples]
```

## Best Practices for Skill Development

### 1. Keep Skills Focused
- One skill = one domain or task area
- Create multiple small skills rather than one mega-skill
- If a skill covers multiple unrelated topics, split it

### 2. Write Discoverable Descriptions
- Include keywords Claude can match against user requests
- Specify both WHAT it does and WHEN to use it
- Use third-person form ("This skill should be used when...")
- Avoid vague or overly technical descriptions

### 3. Use Imperative/Infinitive Form
- Write using verb-first instructions: "To accomplish X, do Y"
- Avoid second person: Don't use "you should" or "you can"
- Maintain objective, instructional language throughout

### 4. Structure for Progressive Disclosure
- Put common cases and essential procedures in SKILL.md (<5k words)
- Move detailed specs and documentation to `references/`
- Extract deterministic operations to `scripts/`
- Place output templates and assets in `assets/`
- Avoid duplication between SKILL.md and bundled resources

### 5. Provide Clear Examples
- Show both correct and incorrect approaches
- Use realistic, complete examples
- Explain why one approach is better than another

### 6. Reference Bundled Resources Explicitly
- Tell Claude when and how to use scripts
- Direct Claude to read references for detailed information
- Show how to use assets in output

### 7. Test with Realistic Scenarios
- Verify Claude loads the skill at the right time
- Ensure the guidance is sufficient for common tasks
- Test that bundled resources are accessible and functional
- Iterate based on actual usage patterns

### 8. Include Context for Decision-Making
- Explain WHY rules exist, not just WHAT they are
- Help Claude make contextual decisions
- Provide rationale for best practices

## Skill Validation Checklist

Before finalizing a skill, verify:

**Frontmatter:**
- [ ] Name is clear and under 64 characters
- [ ] Description includes what it does AND when to use it
- [ ] Description is under 200 characters
- [ ] Version number included (if tracking versions)
- [ ] Dependencies listed (if any required)

**Content Structure:**
- [ ] Introduction section explains the skill's purpose
- [ ] "When to Use This Skill" section lists specific use cases
- [ ] Core guidelines are organized by topic with clear headers
- [ ] Examples show both good and bad patterns
- [ ] Best practices section provides actionable guidance

**Quality:**
- [ ] Skill is focused on one domain/task area
- [ ] Common cases are handled in core content
- [ ] Supporting files used for detailed specs (if needed)
- [ ] Examples are realistic and complete
- [ ] Language is clear, imperative, and actionable

**Integration:**
- [ ] Skill placed in appropriate directory
- [ ] Supporting files referenced correctly
- [ ] Plugin documentation updated (if applicable)
- [ ] Tested with realistic user requests

## Example: Complete Skill Template

```markdown
---
name: My Skill Name
description: Brief description of what this skill does and when to use it (max 200 chars)
dependencies: python>=3.8
---

# My Skill Name Skill

This skill provides [brief description of capability and purpose].

## When to Use This Skill

Invoke this skill when:
- [Specific use case 1]
- [Specific use case 2]
- [Specific use case 3]

## Core Guidelines

### Primary Topic

[Main instructions and guidelines]

**Key Rules:**
- [Rule 1 with rationale]
- [Rule 2 with rationale]
- [Rule 3 with rationale]

### Secondary Topic

[Additional instructions]

## Examples

**Example 1: Common Use Case**

Good approach:
```
[Code or text showing correct approach]
```

Bad approach:
```
[Code or text showing incorrect approach]
```

**Example 2: Edge Case**

[Another example with explanation]

## Best Practices

1. **Practice Name:** Why this matters and how to apply it
2. **Practice Name:** Why this matters and how to apply it
3. **Practice Name:** Why this matters and how to apply it

## Supporting Resources

For detailed specifications, read references/specifications.md.
For data processing utilities, execute scripts/process_data.py.

## Troubleshooting

**Problem:** Common issue description

**Solution:** How to resolve it
```

## Creating a Skill: Step-by-Step

When asked to create a new skill, follow these steps:

### Step 1: Gather Requirements

Ask the user:
- What is the skill's purpose?
- When should this skill be invoked?
- What domain knowledge does it provide?
- Are there examples of correct/incorrect approaches?
- Are supporting files needed (scripts, references)?

### Step 2: Create Directory Structure

```bash
mkdir -p skills/{skill-name}
```

### Step 3: Write SKILL.md with Frontmatter

Create the YAML frontmatter:
- Name: Clear, descriptive, max 64 chars
- Description: What it does + when to use it, max 200 chars
- Dependencies: List if any required

### Step 4: Write Core Content

Structure the markdown:
1. Title and introduction
2. "When to Use This Skill" section
3. Core guidelines organized by topic
4. Examples (good and bad)
5. Best practices
6. Troubleshooting (optional)

### Step 5: Create Supporting Files (If Needed)

Add:
- references/ directory for detailed specifications and documentation
- scripts/ directory for executable utilities
- assets/ directory for output templates and files

### Step 6: Validate and Test

Review:
- Is the description discoverable?
- Are the guidelines clear and actionable?
- Are examples realistic and helpful?
- Is the skill focused and well-structured?

### Step 7: Document the Skill

If part of a plugin:
- Update plugin README to mention the skill
- Update plugin.json version if significant addition
- Update CHANGELOG.md with the new skill

## Advanced Topics

### Skill Composition

Skills can reference each other:
```markdown
For authentication guidelines, the authentication-standards skill provides detailed guidance.
```

### Conditional Logic

Skills can provide context-dependent guidance:
```markdown
If working in a production environment:
- [Production-specific rules]

If working in a development environment:
- [Development-specific rules]
```

### Integration with MCP Tools

Skills can provide guidance for MCP tool usage:
```markdown
When using the Azure DevOps MCP server tools (mcp__ado__*):
- [Tool-specific guidelines]
```

## Troubleshooting Skill Creation

### Problem: Skill Not Loading When Expected

**Possible Causes:**
- Description doesn't include relevant keywords
- Description is too vague or generic
- Skill is too broad and Claude can't determine relevance

**Solution:**
- Revise description to include specific keywords and use cases
- Make description more specific about when to invoke
- Consider splitting into multiple focused skills

### Problem: Skill Loading Too Often

**Possible Causes:**
- Description is too broad
- Keywords match too many contexts

**Solution:**
- Narrow the description
- Add specific qualifiers (e.g., "when using Azure DevOps MCP tools")
- Make the "When to Use This Skill" section more restrictive

### Problem: Skill Content Too Long

**Possible Causes:**
- Trying to cover too many topics
- Including detailed specs that belong in references/ files
- Not using bundled resources effectively

**Solution:**
- Split into multiple focused skills
- Move detailed specs to references/ directory
- Extract code into scripts/ directory
- Focus core content on common cases only

## Security Considerations

When creating skills:

- Only bundle code from trusted sources
- Audit all scripts and dependencies
- Avoid hardcoding credentials or secrets
- Document any external network connections
- Don't bundle unvetted third-party code
- Don't include sensitive organizational data

Skills execute in Claude Code's environment with file and code execution access - treat them with appropriate security caution.

## Summary

Skills are modular packages that extend Claude's capabilities with specialized knowledge, workflows, and tools. To create effective skills:

1. **Understand concrete examples** - Gather specific use cases before building
2. **Plan reusable contents** - Identify what scripts, references, and assets are needed
3. **Create proper structure** - Use SKILL.md + bundled resources (scripts/, references/, assets/)
4. **Write in imperative form** - Use objective, instructional language
5. **Leverage progressive disclosure** - Keep SKILL.md lean (<5k words), move details to bundled resources
6. **Write discoverable descriptions** - Use keywords and third-person form
7. **Test and iterate** - Validate with realistic scenarios and improve based on feedback

By following the Skill Creation Process and these best practices, skills will enhance Claude Code's capabilities for specialized tasks while maintaining token efficiency and usability.
