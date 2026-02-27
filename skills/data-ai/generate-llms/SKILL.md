---
name: generate-llms
description: Generate llms.txt and llms-full.txt files for AI agent consumption following the llmstxt.org standard. Use when updating site content that should be reflected in the llms files, or when building/deploying the site.
---

# Generate LLMs Files

Generate `/llms.txt` and `/llms-full.txt` files that help AI agents understand and use the agentconfig.org site content.

## Overview

This skill generates machine-readable documentation files following the [llmstxt.org](https://llmstxt.org) standard:

- **`/llms.txt`** — A curated table of contents with links to detailed pages
- **`/llms-full.txt`** — Comprehensive markdown containing all site content
- **`/*.md`** — Page-specific markdown files (skills.md, agents.md, mcp.md, etc.)

## Page Registry

Pages are automatically discovered from the **page registry** at `site/src/data/pages.ts`:

```typescript
export const pages: readonly PageMeta[] = [
  {
    slug: 'skills',
    title: 'Skills Tutorial',
    description: 'How to create agent skills...',
    mdFile: 'skills.md',
    partNumber: 3,
  },
  // ... more pages
]
```

When creating a new page, add an entry to the registry to include it in the llms files.

## When to Use

Use this skill when:
- You've updated content in the data files (`site/src/data/*.ts`)
- You've added a new page (remember to add it to the page registry!)
- Before deploying or releasing a new version of the site
- When an agent needs to understand what content should be in llms files

## Data Sources

The llms files are generated from these TypeScript data files:

| File | Content |
|------|---------|
| `site/src/data/pages.ts` | **Page registry** - lists all pages for llms generation |
| `site/src/data/primitives.ts` | 10 AI primitives with descriptions, use cases, and provider implementations |
| `site/src/data/comparison.ts` | Provider comparison matrix (Copilot vs Claude) |
| `site/src/data/skillsTutorial.ts` | Skills tutorial sections and concepts |
| `site/src/data/skillExamples.ts` | 5 example skills with full code |
| `site/src/data/agentsTutorial.ts` | Agent definitions tutorial with code samples |
| `site/src/data/mcpTutorial.ts` | MCP tutorial sections and code samples |

## File Structure

```
site/public/
├── llms.txt           # Table of contents (llmstxt.org format)
├── llms-full.txt      # Complete site content in one file
├── skills.md          # Skills page content (rich markdown)
├── agents.md          # Agents page content (rich markdown)
└── mcp.md             # MCP page content (rich markdown)
```

## Generation Process

### Step 1: Generate `/llms.txt` (Table of Contents)

The homepage llms.txt follows the strict llmstxt.org format:
- H1 with project name
- Blockquote with summary
- Optional paragraph (no headings)
- H2 sections with link lists only

```markdown
# agentconfig.org

> A reference site for configuring AI coding assistants. Covers 10 AI primitives,
> provider comparison (GitHub Copilot vs Claude Code), and tutorials for skills
> and agent definitions.

## Pages

- [Homepage](https://agentconfig.org/): AI primitives reference, file tree, provider comparison
- [Skills Tutorial](https://agentconfig.org/skills): How to create agent skills following agentskills.io
- [Agents Tutorial](https://agentconfig.org/agents): Agent definition files for Copilot and Claude

## Docs

- [Full site content](/llms-full.txt): Complete content for deep context
- [Skills page content](/skills.md): Skills tutorial with 5 example skills
- [Agents page content](/agents.md): Agents tutorial with code samples

## Optional

- [agentskills.io specification](https://agentskills.io/specification): The skills format
- [AGENTS.md specification](https://agents.md): Open format for coding agents
```

### Step 2: Generate `/llms-full.txt` (Complete Content)

Run the generation script:

```bash
bun .github/skills/generate-llms/scripts/generate-llms-full.ts
```

This reads all data files and generates comprehensive markdown including:

1. **Site overview** with purpose and structure
2. **All 10 AI primitives** with full details:
   - Description
   - What it is
   - When to use it
   - What it prevents
   - What to combine it with
   - Provider implementations (Copilot and Claude)
3. **Provider comparison table** showing support levels
4. **Config file locations** for both providers
5. **Skills tutorial** with all concepts and 5 example skills
6. **Agents tutorial** with all sections and code samples

### Step 3: Generate Page-Specific `.md` Files

**`/skills.md`:**
- Tutorial sections (Understanding the Spec, Progressive Disclosure, etc.)
- 5 example skills with full SKILL.md content
- Key takeaways for each example

**`/agents.md`:**
- 9 tutorial sections from beginner to advanced
- Code samples for AGENTS.md, CLAUDE.md, copilot-instructions.md
- Path-scoped rules, agent personas, monorepo strategies

## Manual Generation

If the script isn't available, generate manually by:

1. Read each data file in `site/src/data/`
2. Extract the exported arrays/objects
3. Format as markdown following the templates below

### Template: Primitive Entry

```markdown
### {name}

{description}

**What it is:** {whatItIs}

**Use when:**
{useWhen as bullet list}

**Prevents:** {prevents}

**Combine with:** {combineWith as comma-separated list}

**Provider Implementations:**

| Provider | Implementation | Location | Support |
|----------|---------------|----------|---------|
| Copilot | {implementation} | {location} | {support} |
| Claude | {implementation} | {location} | {support} |
```

### Template: Skill Example

```markdown
### {displayName}

**Complexity:** {complexity}
**Demonstrates:** {demonstrates}

{description}

**Files:**

\`\`\`{language}
// {path}
{content}
\`\`\`

**Key Takeaways:**
{keyTakeaways as bullet list}
```

## Validation

After generating, verify:

1. **Links work** — All internal links resolve
2. **Content is current** — Matches what's shown on the site
3. **Format is correct** — Valid markdown, proper headings
4. **Size is reasonable** — llms-full.txt should be comprehensive but not bloated

## Example Prompts

### Generate all llms files
```
Generate all the llms.txt files for agentconfig.org
```

### Update after content change
```
I updated the primitives data, regenerate the llms files
```

### Check what should be in llms-full.txt
```
What content should be included in llms-full.txt?
```
