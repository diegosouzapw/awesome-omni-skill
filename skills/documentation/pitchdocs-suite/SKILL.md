---
name: pitchdocs-suite
description: One-command generation and audit of the full public repository documentation set — README, CHANGELOG, ROADMAP, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, issue templates, PR template, and discussion templates. Use when setting up a new repo or auditing an existing one.
version: "1.0.0"
upstream: "contributor-covenant@3.0"
---

# Repository Documentation Suite

## Complete Docs Inventory

A well-documented public repository should have these files:

### Tier 1: Essential (Every Public Repo)

| File | Purpose | Generator |
|------|---------|-----------|
| `README.md` | First impression, value proposition, quickstart | `public-readme` skill |
| `LICENSE` | Legal terms for usage — see LICENSE Selection Framework below | Auto-detect from package.json |
| `CONTRIBUTING.md` | How to contribute code, report bugs, suggest features | This skill |
| `.github/ISSUE_TEMPLATE/bug_report.yml` | Structured bug reports | This skill |
| `.github/ISSUE_TEMPLATE/feature_request.yml` | Feature proposals | This skill |
| `.github/PULL_REQUEST_TEMPLATE.md` | PR checklist and description template | This skill |

### Tier 2: Professional (Active Projects)

| File | Purpose | Generator |
|------|---------|-----------|
| `CHANGELOG.md` | User-facing change history | `changelog` skill |
| `SUPPORT.md` | Where to get help — issues, discussions, external channels | This skill |
| `.github/release.yml` | Auto-generated release note categories | This skill |
| `llms.txt` | LLM-friendly content index for AI tools (Cursor, Windsurf, Claude Code) | `llms-txt` skill |
| `AGENTS.md` | Cross-tool AI agent context — conventions, architecture, key commands | `ai-context` skill |
| `.github/copilot-instructions.md` | GitHub Copilot repository-level instructions | `ai-context` skill |
| `.windsurfrules` | Windsurf (Cascade AI) project-level context | `ai-context` skill |
| `.clinerules` | Cline VS Code extension project-level context | `ai-context` skill |
| `CODE_OF_CONDUCT.md` | Community behaviour standards | This skill |
| `SECURITY.md` | Vulnerability reporting process | This skill |
| `.github/ISSUE_TEMPLATE/config.yml` | Issue template chooser config | This skill |
| `.github/FUNDING.yml` | Sponsorship links | This skill |
| `docs/README.md` | Documentation hub page | `user-guides` skill |
| `docs/guides/getting-started.md` | Expanded quickstart for new users | `user-guides` skill |

### Tier 3: Mature (Established Projects)

| File | Purpose | Generator |
|------|---------|-----------|
| `ROADMAP.md` | Public development roadmap | `roadmap` skill |
| `CLAUDE.md` | Project-specific Claude Code context — coding standards, architecture, key paths | `ai-context` skill |
| `.cursorrules` | Cursor-specific rules derived from codebase conventions | `ai-context` skill |
| `GEMINI.md` | Gemini CLI project context (or `.gemini/GEMINI.md`) | `ai-context` skill |
| `docs/guides/configuration.md` | All config options explained | `user-guides` skill |
| `docs/guides/deployment.md` | Production deployment guide | `user-guides` skill |
| `docs/guides/troubleshooting.md` | Common issues and solutions | `user-guides` skill |
| `.github/DISCUSSION_TEMPLATE/` | Structured discussion categories | This skill |
| `.github/CODEOWNERS` | Automatic review assignment | Manual |
| `CITATION.cff` | Machine-readable citation for academic/research repos (GitHub shows "Cite this repository" button) | This skill |

### Repository Metadata (GitHub Settings)

Beyond files, a well-configured repo also needs correct GitHub-level metadata for discoverability:

| Setting | Purpose | Limit |
|---------|---------|-------|
| **Topics** | Drive GitHub search and discovery — appear in repo header and topic browse pages | Up to 20 |
| **Description** | Short text under repo name in GitHub search results and repo header | ~350 characters |
| **Website URL** | Linked from repo header — directs users to docs site, homepage, or package registry | Single URL |

#### Reading Current Metadata

```bash
gh repo view --json topics,homepageUrl,description
```

#### Setting Metadata

```bash
# Topics — add individually
gh repo edit --add-topic typescript --add-topic documentation --add-topic cli

# Description
gh repo edit --description "Generate repository documentation that sells as well as it informs."

# Website URL
gh repo edit --homepage "https://docs.example.com"
```

#### Topic Suggestion Framework

Suggest topics by scanning the project and picking from these categories. Aim for 5-10 topics total.

| Category | Source | Examples |
|----------|--------|----------|
| Language/runtime | Manifest file (`package.json`, `pyproject.toml`, `go.mod`) | `typescript`, `python`, `go`, `rust`, `javascript` |
| Framework | Dependencies and config files | `react`, `nextjs`, `fastapi`, `django`, `cloudflare-workers` |
| Category | What the project IS | `documentation`, `cli`, `api`, `devtools`, `plugin`, `library` |
| Ecosystem | Platform or tool ecosystem it belongs to | `claude-code`, `openai`, `llm`, `github-actions`, `terraform` |
| Purpose | What problem it solves | `testing`, `monitoring`, `deployment`, `developer-tools`, `code-generation` |

**Rules:**
- Use lowercase, hyphenated (GitHub enforces this)
- Be specific: `claude-code-plugin` over `plugin`
- Include the primary language even if obvious
- Don't pad with generic topics like `awesome` or `open-source`
- Match topics that real users would search for

#### Description Guidance

The GitHub repo description should match or condense the README one-liner:
- Maximum ~350 characters (GitHub truncates beyond this)
- Benefit-focused, not feature-focused
- No markdown — plain text only
- Should make sense standalone in search results

#### Website URL Guidance

Set to the most useful entry point for new users, in priority order:
1. Dedicated docs site (e.g., `docs.project.com`)
2. Project homepage (e.g., `project.com`)
3. Package registry page (e.g., `npmjs.com/package/name`)
4. GitHub Pages docs (e.g., `org.github.io/repo`)

#### Package Registry Configuration

For projects published to npm or PyPI, the package registry page is often the second most-visited page after the GitHub repo. Registry metadata affects search ranking, trust signals, and first impressions.

Load the `package-registry` skill for:
- Complete field inventories (what metadata affects the npm/PyPI page)
- README cross-renderer compatibility (what Markdown features break on npm/PyPI)
- Registry-specific badge templates (version, downloads, types, Python versions)
- Trusted publishing and provenance guidance (npm OIDC, PyPI Trusted Publisher)
- Audit checklists for registry metadata completeness

#### Social Preview Image

The social preview appears when sharing repo links on Twitter/X, Slack, Discord, and LinkedIn. Without a custom image, GitHub auto-generates a bland preview from the repo name.

- **Recommended size**: 1280x640px (minimum 640x320)
- **File size**: under 1MB, ideally <300KB
- **Set via**: Settings > Social preview (manual upload — no CLI or API)
- **Design tip**: keep key text centred to survive cropping on different platforms
- **Cannot be audited programmatically** — the audit should remind users to check

### Visual Assets Guidance

Guidance on storing and referencing visual elements (screenshots, demo GIFs, diagrams) in repository documentation.

#### Storage Locations

| Location | Best For | Pros | Cons |
|----------|----------|------|------|
| In-repo (`docs/images/` or `assets/`) | Small files (<5MB) | Version-controlled, always accessible | Increases repo size |
| GitHub user-content | Any size | Keeps repo small | Not version-controlled, URLs can break |
| GitHub Release assets | Large files (>5MB) | Doesn't bloat repo or git history | Tied to a specific release |
| External CDN (Cloudinary, S3) | Very large assets | Full control | External dependency |

**GitHub user-content hosting**: drag-and-drop an image into any GitHub issue, PR, or comment to get a permanent `user-images.githubusercontent.com` URL. Use this URL in your README.

#### Recommended Formats

| Format | Best For | Notes |
|--------|----------|-------|
| SVG | Diagrams, architecture charts | Scales perfectly, small file size |
| PNG | Screenshots, UI captures | Lossless, good for text-heavy images |
| GIF | Demo recordings, short workflows | GitHub caps at 10MB, aim for ~10fps |
| WebP | Photos, complex images | Smaller than PNG, good browser support |

#### Quality Rules

- **Width**: under 800px for GitHub's Markdown renderer
- **Alt text**: required for accessibility — describe what the image shows, not just "screenshot"
- **File size**: optimise to <300KB where possible (use tools like `optipng`, `gifsicle`)
- **Naming**: kebab-case, descriptive (`demo-quick-start.gif`, `architecture-overview.svg`)
- **Folder convention**: use `docs/images/` or `assets/` consistently within a project

## Audit Workflow

### Step 1: Scan Existing Docs

```bash
# Check for all expected files
for f in README.md LICENSE CONTRIBUTING.md CHANGELOG.md ROADMAP.md CODE_OF_CONDUCT.md SECURITY.md SUPPORT.md llms.txt AGENTS.md CLAUDE.md .cursorrules .windsurfrules .clinerules; do
  [ -f "$f" ] && echo "✓ $f" || echo "✗ $f (missing)"
done

# Check .github templates and AI context files
for f in .github/ISSUE_TEMPLATE/bug_report.yml .github/ISSUE_TEMPLATE/feature_request.yml .github/PULL_REQUEST_TEMPLATE.md .github/ISSUE_TEMPLATE/config.yml .github/copilot-instructions.md; do
  [ -f "$f" ] && echo "✓ $f" || echo "✗ $f (missing)"
done

# Check for common alternatives
[ -f ".github/ISSUE_TEMPLATE/bug_report.md" ] && echo "⚠ bug_report.md found (consider migrating to YAML forms)"
```

### Step 2: Quality Check Existing Files

For each existing file, check:
- **README.md**: Does it pass the 4-question test? Does it have badges? Is the quickstart working?
- **CONTRIBUTING.md**: Does it match the actual development workflow?
- **CHANGELOG.md**: Is it up to date with the latest release?
- **SECURITY.md**: Does it include a responsible disclosure process?

#### License Validation

Three checks to catch common license issues:

1. **LICENSE file exists** — flag if the file uses `.md` extension (`LICENSE.md`). GitHub's licence detection prefers extensionless `LICENSE` or `LICENSE.txt`.

2. **Manifest matches LICENSE** — cross-reference the `license` field in `package.json` or `pyproject.toml` against the LICENSE file header:
   ```bash
   # npm
   node -e "console.log(require('./package.json').license)" 2>/dev/null
   # PyPI
   python3 -c "import tomllib; f=open('pyproject.toml','rb'); print(tomllib.load(f).get('project',{}).get('license'))" 2>/dev/null
   ```
   Flag mismatches (e.g., manifest says `MIT` but LICENSE file contains Apache-2.0 text).

3. **No verbatim license text in context files** — AI-generated context files sometimes accidentally embed full license text. Scan for license preamble patterns:
   ```bash
   grep -rl "Permission is hereby granted, free of charge" .claude/ .cursorrules AGENTS.md .clinerules .windsurfrules GEMINI.md 2>/dev/null
   grep -rl "Licensed under the Apache License" .claude/ .cursorrules AGENTS.md .clinerules .windsurfrules GEMINI.md 2>/dev/null
   ```
   Flag any matches — license text belongs in LICENSE, not in skill/rule/context files.

### Step 3: Generate Missing Files

Use the appropriate skill/template for each missing file. Generate in priority order:
1. README.md (if missing or needs overhaul)
2. CONTRIBUTING.md
3. Issue templates
4. PR template
5. CHANGELOG.md
6. Everything else

## Templates

> **Content filter awareness:** Claude Code's API has a content filtering system that can block output (HTTP 400) when generating standard OSS files containing governance language, security terminology, or verbatim legal text. For each template below, follow the noted generation strategy. See the `docs-writer` agent for the full mitigation playbook.

### CONTRIBUTING.md

**Content filter note:** This template is lengthy. If the content filter blocks generation, write it in chunks:
1. Write the header and "Quick Links" section first
2. Add "Development Setup" section
3. Add "How to Contribute" section (reporting bugs, suggesting features, submitting code)
4. Add "Commit Messages" and "Code Review" sections last

The template below is a reference — adapt it to the project's actual workflow rather than reproducing it verbatim.

```markdown
# Contributing to [Project Name]

Thank you for your interest in contributing! This guide will help you get started.

## Quick Links

- [Open Issues](link) — Find something to work on
- [Good First Issues](link) — Perfect for new contributors
- [Discussion Forum](link) — Ask questions, propose ideas

## Development Setup

### Prerequisites

- [Runtime] version [X]+
- [Package manager]

### Getting Started

\`\`\`bash
# Clone the repo
git clone https://github.com/org/repo.git
cd repo

# Install dependencies
npm install

# Run tests to verify setup
npm test

# Start development
npm run dev
\`\`\`

## How to Contribute

### Reporting Bugs

1. Search [existing issues](link) to avoid duplicates
2. Use the [bug report template](link)
3. Include: what you expected, what happened, steps to reproduce

### Suggesting Features

1. Check the [roadmap](ROADMAP.md) — it might already be planned
2. Open a [feature request](link)
3. Describe the problem you're solving, not just the solution you want

### Submitting Code

1. Fork the repo and create a branch: `git checkout -b feature/your-feature`
2. Write your code and tests
3. Ensure all tests pass: `npm test`
4. Ensure linting passes: `npm run lint`
5. Commit using [conventional commits](https://conventionalcommits.org/): `feat: add thing`
6. Push and open a pull request

### Commit Messages

We use [Conventional Commits](https://conventionalcommits.org/):

- `feat: add new feature` — New functionality
- `fix: resolve bug` — Bug fix
- `docs: update readme` — Documentation only
- `test: add tests` — Test additions
- `refactor: restructure code` — Code change that doesn't fix a bug or add a feature
- `chore: update deps` — Maintenance tasks

### Code Review

All submissions require review. We aim to review PRs within 48 hours. We may suggest changes, improvements, or alternatives.

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold this code.

## Questions?

- Open a [discussion](link)
- File an [issue](link)

Thank you for making [Project Name] better!
```

### CODE_OF_CONDUCT.md

**Content filter note:** This file contains governance language that triggers Claude Code's content filter when generated inline. Always fetch from the canonical URL rather than writing from scratch.

**Generation method — fetch and customise:**

```bash
# Download Contributor Covenant v3.0
curl -sL "https://www.contributor-covenant.org/version/3/0/code_of_conduct/code_of_conduct.md" -o CODE_OF_CONDUCT.md
```

After fetching, use Edit tool to replace these placeholders:
- `[INSERT CONTACT METHOD]` — project contact email or reporting URL
- Verify the "Enforcement" section matches the project's governance structure

**Why v3.0:** Clearer language, less US-centric phrasing, "Addressing and Repairing Harm" section aligned with restorative justice principles. Always use v3.0 for new projects.

**Fallback:** If the URL is unreachable, direct the user to https://www.contributor-covenant.org/version/3/0/code_of_conduct/ and ask them to download manually.

### SECURITY.md

**Content filter note:** Security policy files contain vulnerability/exploit keywords that can trigger Claude Code's content filter. Fetch a template and customise rather than generating from scratch.

**Generation method — fetch and customise:**

```bash
# Option 1: GitHub's default security policy template
curl -sL "https://raw.githubusercontent.com/github/.github/main/SECURITY.md" -o SECURITY.md

# Option 2: If Option 1 fails, create a minimal starter file
cat > SECURITY.md << 'SECEOF'
# Security Policy

## Reporting a Vulnerability

Please report security issues via [GitHub Security Advisories](link/security/advisories/new).
SECEOF
```

After fetching or creating the starter file, use Edit tool to customise in small chunks:

1. **Supported versions table** — add the project's version support matrix
2. **Reporting method** — replace with project-specific email or GitHub Security Advisories URL (`https://github.com/org/repo/security/advisories/new`)
3. **Response timeline** — set acknowledgement (48h), assessment (1 week), and fix timelines
4. **Disclosure policy** — add coordinated disclosure statement

**Required sections** (ensure all are present after customisation):
- Supported Versions (table with version and support status)
- Reporting a Vulnerability (contact method, what to include)
- Response Timeline (acknowledgement, assessment, fix timelines)
- Disclosure Policy (coordinated disclosure)
- Security Updates (reference to CHANGELOG.md)

### .github/ISSUE_TEMPLATE/bug_report.yml

```yaml
name: Bug Report
description: Report a bug to help us improve
title: "[Bug]: "
labels: ["bug", "triage"]
body:
  - type: markdown
    attributes:
      value: |
        Thanks for reporting a bug! Please fill out the sections below.
  - type: textarea
    id: description
    attributes:
      label: What happened?
      description: A clear description of the bug.
      placeholder: Tell us what went wrong...
    validations:
      required: true
  - type: textarea
    id: expected
    attributes:
      label: What did you expect?
      description: What should have happened instead?
    validations:
      required: true
  - type: textarea
    id: reproduce
    attributes:
      label: Steps to reproduce
      description: Minimal steps to reproduce the issue.
      placeholder: |
        1. Run `command`
        2. Pass input `...`
        3. See error
    validations:
      required: true
  - type: input
    id: version
    attributes:
      label: Version
      description: What version are you using?
      placeholder: "1.2.3"
    validations:
      required: true
  - type: dropdown
    id: os
    attributes:
      label: Operating System
      options:
        - macOS
        - Linux
        - Windows
        - Other
    validations:
      required: true
  - type: textarea
    id: logs
    attributes:
      label: Relevant logs
      description: Paste any relevant error messages or logs.
      render: shell
```

### .github/ISSUE_TEMPLATE/feature_request.yml

```yaml
name: Feature Request
description: Suggest a new feature or improvement
title: "[Feature]: "
labels: ["enhancement"]
body:
  - type: markdown
    attributes:
      value: |
        Thanks for suggesting a feature! Help us understand the problem you're solving.
  - type: textarea
    id: problem
    attributes:
      label: What problem does this solve?
      description: Describe the problem or limitation you're facing.
      placeholder: "I'm always frustrated when..."
    validations:
      required: true
  - type: textarea
    id: solution
    attributes:
      label: Proposed solution
      description: How would you like this to work?
    validations:
      required: true
  - type: textarea
    id: alternatives
    attributes:
      label: Alternatives considered
      description: Have you considered any workarounds or alternatives?
  - type: textarea
    id: context
    attributes:
      label: Additional context
      description: Any other context, screenshots, or examples.
```

### .github/ISSUE_TEMPLATE/config.yml

```yaml
blank_issues_enabled: false
contact_links:
  - name: Question / Help
    url: https://github.com/org/repo/discussions/categories/q-a
    about: Ask questions and get help from the community
  - name: Feature Discussion
    url: https://github.com/org/repo/discussions/categories/ideas
    about: Discuss feature ideas before opening a formal request
```

### .github/PULL_REQUEST_TEMPLATE.md

```markdown
## What

<!-- Brief description of the change -->

## Why

<!-- What problem does this solve? Link to issue if applicable -->

Closes #

## How

<!-- How was it implemented? Any design decisions worth noting? -->

## Testing

- [ ] Tests added/updated
- [ ] All tests pass (`npm test`)
- [ ] Linting passes (`npm run lint`)

## Checklist

- [ ] Code follows project conventions
- [ ] Self-reviewed the diff
- [ ] No secrets or credentials included
- [ ] Documentation updated (if applicable)
```

### .github/FUNDING.yml

```yaml
github: [username]
# ko_fi: username
# open_collective: project-name
# custom: ["https://example.com/donate"]
```

### SUPPORT.md

```markdown
# Support

## How to Get Help

- **Bug reports**: [File an issue](link) using the bug report template
- **Feature requests**: [Open a feature request](link)
- **Questions**: [Start a discussion](link) or check existing Q&A
- **Security issues**: See [SECURITY.md](SECURITY.md) for responsible disclosure

## Documentation

- [Getting Started](docs/guides/getting-started.md)
- [Configuration](docs/guides/configuration.md)
- [Troubleshooting](docs/guides/troubleshooting.md)

## Community

- [Discussions](link) — Ask questions, share ideas
- [Contributing](CONTRIBUTING.md) — Help improve the project
```

### .github/release.yml

Configures [automatically generated release notes](https://docs.github.com/en/repositories/releasing-projects-on-github/automatically-generated-release-notes) on GitHub. When you create a release, GitHub categorises merged PRs by label into structured sections.

```yaml
changelog:
  exclude:
    labels:
      - ignore-for-release
    authors:
      - dependabot
  categories:
    - title: Breaking Changes
      labels:
        - breaking-change
    - title: New Features
      labels:
        - enhancement
        - feature
    - title: Bug Fixes
      labels:
        - bug
        - fix
    - title: Documentation
      labels:
        - documentation
    - title: Other Changes
      labels:
        - "*"
```

### CITATION.cff (Conditional)

Include when the project is academic, research-adjacent, data science, ML, or likely to be cited in papers. GitHub natively shows a "Cite this repository" button when this file is present.

**When to include:**
- Academic or research software
- Data science libraries, ML models, scientific tools
- Any project published to Zenodo for DOI assignment
- Projects likely referenced in papers, reports, or presentations

**When to skip:**
- Internal tools, configuration plugins, small utilities

```yaml
cff-version: 1.2.0
message: "If you use this software, please cite it as below."
type: software
title: "[Project Name]"
version: "[version]"
date-released: "[YYYY-MM-DD]"
authors:
  - family-names: "[Last]"
    given-names: "[First]"
    orcid: "https://orcid.org/0000-0000-0000-0000"
repository-code: "https://github.com/org/repo"
license: "[SPDX-identifier]"
keywords:
  - "[keyword1]"
  - "[keyword2]"
```

### LICENSE Selection Framework

The plugin checks for LICENSE presence but does not generate the file content — use GitHub's built-in license picker or [choosealicense.com](https://choosealicense.com/).

| License | Best For | Key Feature |
|---------|----------|-------------|
| MIT | Libraries, tools, general OSS | Maximum freedom, minimal restrictions |
| Apache-2.0 | Libraries with patent concerns | Explicit patent grant |
| GPL-3.0 | Projects that must stay open | Copyleft — derivatives must be GPL too |
| AGPL-3.0 | SaaS/server-side projects | Network copyleft — even hosted use triggers sharing |
| ISC | Minimal alternative to MIT | Functionally identical, shorter text |
| Unlicense | Public domain dedication | No restrictions at all |

**Decision guidance:**
- Default to MIT for most open-source projects
- Use Apache-2.0 if contributors may hold patents
- Use GPL/AGPL only with clear intent — it limits adoption by commercial users
- Check `license` field in `package.json`/`pyproject.toml` matches the LICENSE file content
- Proprietary projects may omit LICENSE or include custom terms
