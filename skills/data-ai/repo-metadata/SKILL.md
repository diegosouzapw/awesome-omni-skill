---
name: repo-metadata
description: This skill should be used when the user asks to "update repo description", "improve repository description", "generate topics", "add labels to repo", "optimize github metadata", "make repo more discoverable", "improve repo SEO", "update project description", or needs to create engaging repository descriptions and topics that improve discoverability. Analyzes project files to generate optimized GitHub metadata.
---

# Repository Metadata Generator

Generate engaging GitHub repository descriptions and topics by analyzing project intent, purpose, and features. Create metadata that improves discoverability and accurately represents what a project does.

## Overview

This skill analyzes a project's codebase, documentation, and configuration to understand its purpose, then generates:
1. An optimized repository description (default, no configuration needed)
2. Relevant topics/labels for discoverability
3. Optionally applies changes via `gh` CLI (when user confirms)

## GitHub Metadata Constraints

All generated metadata must meet GitHub's requirements:

**Description:**
- Maximum: 350 characters
- Should be concise, engaging, and searchable
- No markdown or special formatting
- Include key functionality and value proposition

**Topics:**
- Maximum: 20 topics per repository
- Each topic: lowercase, hyphenated, no spaces
- Maximum 50 characters per topic
- Should include: language, framework, domain, features

## Workflow

### Step 1: Check for Configuration

Look for optional settings file at `.claude/github-social.local.md`.

If file exists, check for metadata preferences:
- `description_style`: concise | detailed | technical
- `topic_count`: Number of topics to suggest (default: 10)
- `include_language`: Whether to include primary language as topic
- `include_framework`: Whether to include frameworks as topics

If no config exists, proceed with defaults.

### Step 2: Analyze Project

Gather project context by reading available files:

**Primary sources** (check in order):
1. `README.md` - Project description, features, purpose
2. `package.json` / `Cargo.toml` / `pyproject.toml` / `go.mod` - Name, description, keywords
3. `CLAUDE.md` - Project context and guidelines
4. Source code structure - Primary language, frameworks

**Extract these elements**:
- **Project name**: Official name
- **Purpose**: What problem does it solve? (1-2 sentences)
- **Key features**: Top 3-5 capabilities
- **Technology**: Primary language and frameworks
- **Domain**: What field/industry? (DevTools, AI, Web, etc.)
- **Target audience**: Who uses this?
- **Unique value**: What makes it different?

### Step 3: Generate Description

Craft an engaging description that:

1. **Leads with value** - What does it do for the user?
2. **Includes key features** - Top 2-3 capabilities
3. **Uses active voice** - Direct, engaging language
4. **Stays within 350 chars** - Concise but complete
5. **Avoids jargon** - Accessible to broader audience

**Description patterns by project type:**

**Library/Package:**
```
[Action verb] [what it does] for [language/platform]. Features [key capability 1], [key capability 2], and [key capability 3].
```

**CLI Tool:**
```
[Action verb] [what problem it solves] from the command line. [Key benefit or feature].
```

**Framework:**
```
[Adjective] [type] framework for building [what]. [Key differentiator].
```

**Application:**
```
[What it is] that [what it does]. [Key benefit for users].
```

### Step 4: Generate Topics

Select relevant topics from these categories:

**Language/Runtime** (1-2 topics):
- Primary language: `python`, `typescript`, `rust`, `go`
- Runtime: `nodejs`, `deno`, `bun`

**Framework/Library** (1-3 topics):
- Web: `react`, `vue`, `nextjs`, `fastapi`
- Data: `pandas`, `numpy`, `pytorch`
- CLI: `cli`, `command-line`, `terminal`

**Domain** (2-4 topics):
- `devtools`, `developer-tools`, `automation`
- `machine-learning`, `ai`, `data-science`
- `web-development`, `frontend`, `backend`
- `database`, `api`, `infrastructure`

**Features** (2-4 topics):
- `open-source`, `cross-platform`
- `testing`, `documentation`, `monitoring`
- Feature-specific: `image-generation`, `code-review`

**Quality/Style** (1-2 topics):
- `lightweight`, `fast`, `minimal`
- `production-ready`, `enterprise`

**Topic formatting rules:**
- All lowercase
- Hyphens for multi-word: `machine-learning` not `machine_learning`
- No special characters
- Max 50 chars per topic

### Step 5: Output or Apply

**Default (output only):**

Display the generated metadata:

```markdown
## Generated Repository Metadata

### Description (X characters)
[Generated description]

### Topics (X topics)
[topic-1] [topic-2] [topic-3] ...

### To Apply Manually
1. Go to repository Settings → General
2. Update "Description" field
3. Add topics under "Topics"

### To Apply via CLI
gh repo edit --description "[description]"
gh repo edit --add-topic topic-1 --add-topic topic-2 ...
```

**Apply mode (when user confirms):**

If user says "apply", "update", or "yes, update the repo":

```bash
# Update description
gh repo edit --description "[generated description]"

# Add topics (one at a time)
gh repo edit --add-topic topic-1
gh repo edit --add-topic topic-2
# ... for each topic
```

Verify changes:
```bash
gh repo view --json description,repositoryTopics
```

Report success or errors.

## Description Writing Guidelines

### Do:
- Start with a strong action verb
- Focus on user benefit
- Include searchable keywords naturally
- Be specific about what it does
- Keep it scannable

### Don't:
- Start with "A" or "This is"
- Use passive voice
- Include version numbers
- Use excessive punctuation
- Make unsubstantiated claims ("best", "fastest")

### Examples by quality:

**Poor:**
```
This is a tool for working with images. It can do many things.
```

**Good:**
```
Generate social preview images for GitHub repos by analyzing project intent. Supports DALL-E, Stable Diffusion, and manual prompts.
```

**Poor:**
```
A Python library.
```

**Good:**
```
Fast, type-safe HTTP client for Python with automatic retries, connection pooling, and OpenAPI integration.
```

## Topic Selection Strategy

### For maximum discoverability:

1. **Include the obvious** - Language, primary framework
2. **Add the domain** - What category of tool is this?
3. **Feature keywords** - What can users search for?
4. **Ecosystem tags** - Related tools/standards
5. **Audience tags** - Who is this for?

### Example topic sets:

**Python CLI tool:**
```
python cli command-line terminal devtools automation pip hacktoberfest
```

**React component library:**
```
react typescript components ui-library frontend web npm design-system
```

**Machine learning project:**
```
python machine-learning deep-learning pytorch neural-network ai data-science
```

**DevOps tool:**
```
devops kubernetes docker infrastructure automation ci-cd cloud-native golang
```

## Error Handling

**No README or project files found:**
Ask user to describe the project, then proceed with that information.

**gh CLI not available:**
Output metadata only with manual instructions.

**gh repo edit fails:**
1. Report the error
2. Check if user has repo admin access
3. Suggest manual update via GitHub UI

**Repository not found:**
Verify user is in a git repo with GitHub remote.

## Examples

### Example 1: CLI Tool

**Project analysis:**
- Name: github-social
- Purpose: Generate social preview images
- Tech: Claude Code plugin
- Features: DALL-E support, multiple styles, zero-config

**Generated description:**
```
Generate social preview images for GitHub repositories by analyzing project intent. Supports DALL-E, Stable Diffusion, and manual prompt generation.
```
(148 characters)

**Generated topics:**
```
github claude-code plugin social-preview image-generation dalle openai devtools automation open-source
```

### Example 2: Python Library

**Project analysis:**
- Name: fasthttp
- Purpose: HTTP client library
- Tech: Python, async
- Features: Fast, typed, retries

**Generated description:**
```
Async HTTP client for Python with automatic retries, connection pooling, and full type hints. Built for performance and developer experience.
```
(142 characters)

**Generated topics:**
```
python http-client async asyncio requests api web typed networking pip
```

## Additional Resources

### Reference Files
- **`references/description-patterns.md`** - Detailed description templates by project type
- **`references/topic-taxonomy.md`** - Comprehensive topic categories and examples
