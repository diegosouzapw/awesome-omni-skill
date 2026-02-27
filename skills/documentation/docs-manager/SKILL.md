---
name: docs-manager
description: Generate and organize repository documentation. Use when the user asks to "create documentation", "generate docs", "set up CLAUDE.md", "bootstrap project docs", "organize documentation", "improve my docs", "create AGENTS.md", "set up README", "add CONTRIBUTING.md", or wants help structuring repository documentation files.
version: 1.0.0
---

# Documentation Manager for Claude Code

## Overview

This skill generates well-structured repository documentation where each file has clear ownership of its content. Whether starting fresh or improving existing docs, the approach is the same: analyze the project and produce properly-structured documentation.

**Key principles:**
- **Single Source of Truth**: Each piece of information lives in exactly one place
- **Audience Separation**: Human-facing docs (README, CONTRIBUTING, STYLE_GUIDE) vs AI-facing docs (AGENTS.md)
- **Cross-Reference, Don't Duplicate**: Link between files rather than repeating content
- **Actionable Commands**: All commands should be copy-paste ready

## Canonical File Structure

| File | Audience | Canonical Content |
|------|----------|-------------------|
| README.md | Humans (new users) | Project overview, quick start, badges |
| CONTRIBUTING.md | Humans (contributors) | Setup, workflow, PR process |
| STYLE_GUIDE.md | Humans (authors) | Formatting rules, examples, linting config |
| LICENSE | Humans (legal) | Full license text |
| CODE_OF_CONDUCT.md | Humans (community) | Community guidelines, enforcement |
| SECURITY.md | Humans (security) | Vulnerability reporting, security policy |
| CHANGELOG.md | Humans (users) | Version history, release notes |
| AGENTS.md | AI (all agents) | Commands, architecture overview, patterns, gotchas |
| CLAUDE.md | AI (Claude Code) | Redirect to AGENTS.md |
| docs/ | Humans/AI | Full architecture, detailed documentation |
| walkthroughs/ | Humans | Interactive code walkthroughs (virgil format) |

**Note on CLAUDE.md vs AGENTS.md:** Most AI agents look for AGENTS.md, but Claude Code looks for CLAUDE.md. To maintain a single source of truth, AGENTS.md contains all agent guidance and CLAUDE.md simply redirects Claude to read AGENTS.md.

## Workflow

### Phase 1: Gather Context

Before generating any documentation, thoroughly analyze the project:

**1. Discover Project Configuration:**
- Read `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `Makefile`, etc.
- Identify available commands (build, test, lint, format)
- Note dependencies and their purposes

**2. Read Existing Documentation:**
- Check for existing README.md, CONTRIBUTING.md, STYLE_GUIDE.md
- Check for existing CLAUDE.md, AGENTS.md
- Extract valuable information that should be preserved
- Note any documented patterns, gotchas, or conventions

**3. Analyze Codebase Structure:**
- Identify entry points and key directories
- Note architectural patterns (monorepo, microservices, etc.)
- Find test directories and testing conventions
- Identify CI/CD configuration

**4. Detect Technology Stack:**
- Languages used (TypeScript, Python, Rust, Go, etc.)
- Frameworks (React, FastAPI, Actix, etc.)
- Build tools (npm, poetry, cargo, make)
- Linting/formatting tools (ESLint, Prettier, Ruff, rustfmt)

### Phase 2: Ask User

Use AskUserQuestion to confirm and gather additional information:

**Essential questions:**
1. "What is the one-line description of this project?" (for README)
2. "Are there any patterns or gotchas developers should know about?"
3. "Which documentation files should I generate?" (offer: all, or specific subset)

**Optional questions based on context:**
- "I found commands X, Y, Z. Are these the primary development commands?"
- "Is there a specific PR/contribution workflow to document?"
- "Are there any security considerations or authentication patterns?"

### Phase 3: Generate Files

Generate each requested file using the templates in `assets/templates/` as starting points. Fill in discovered information and preserve valuable content from existing docs.

**Generation order (if generating all):**
1. AGENTS.md - Commands and architecture overview (primary AI reference)
2. CLAUDE.md - Redirect to AGENTS.md
3. STYLE_GUIDE.md - Formatting rules
4. CONTRIBUTING.md - Links to STYLE_GUIDE
5. CODE_OF_CONDUCT.md - Community guidelines (if applicable)
6. SECURITY.md - Security policy (if applicable)
7. README.md - Links to other docs
8. LICENSE - Full license text
9. CHANGELOG.md - Version history (if applicable)

**For each file:**
1. Read the corresponding template from `assets/templates/`
2. Fill in placeholders with discovered information
3. Merge valuable content from existing docs
4. Ensure cross-references to other generated files

### Phase 4: Present and Refine

After generating files:
1. Summarize what was created/modified
2. Highlight any content that was migrated between files
3. Ask if adjustments are needed
4. Make refinements based on feedback

## Content Ownership Rules

### README.md Owns:
- Project name and badge line
- One-paragraph description
- Quick start (3-5 steps max)
- Links to other documentation
- Links to external resources

### README.md Does NOT Contain:
- Detailed setup instructions (→ CONTRIBUTING.md)
- Style rules (→ STYLE_GUIDE.md)
- Full architecture (→ docs/)
- Command reference tables (→ AGENTS.md)
- License text (→ LICENSE, just link to it)

### CONTRIBUTING.md Owns:
- Prerequisites and environment setup
- Detailed installation steps
- Development workflow
- How to run tests
- PR process and guidelines
- Code of conduct reference

### CONTRIBUTING.md Does NOT Contain:
- Style rules themselves (link to STYLE_GUIDE.md)
- Command reference (link to AGENTS.md)
- Project overview (link to README.md)

### STYLE_GUIDE.md Owns:
- Code formatting rules
- Naming conventions
- File organization patterns
- Linting configuration explanations
- Examples of correct/incorrect patterns
- Documentation style rules

### STYLE_GUIDE.md Does NOT Contain:
- Setup instructions
- Commands to run linters (→ AGENTS.md)
- Contribution workflow

### AGENTS.md Owns:
- Build/test/lint/format commands (canonical location)
- Project architecture overview (high-level)
- Key files and entry points
- Non-obvious patterns and gotchas
- Tool usage patterns specific to this codebase
- Multi-step workflow guidance
- Context that helps AI agents work effectively

### AGENTS.md Does NOT Contain:
- Detailed style rules (link to STYLE_GUIDE.md)
- Human-oriented prose
- Contribution workflow (link to CONTRIBUTING.md)
- Full architecture documentation (→ docs/)

### CLAUDE.md Owns:
- A redirect pointing Claude to AGENTS.md

### CLAUDE.md Does NOT Contain:
- Any actual documentation (all content lives in AGENTS.md)

### LICENSE Owns:
- Full license text
- Copyright holder and year

### CODE_OF_CONDUCT.md Owns:
- Expected behavior standards
- Unacceptable behavior examples
- Enforcement procedures
- Contact information for reporting

### SECURITY.md Owns:
- Supported versions
- How to report vulnerabilities
- Security policy
- Disclosure timeline

### CHANGELOG.md Owns:
- Version history
- Release dates
- Changes per version (Added, Changed, Deprecated, Removed, Fixed, Security)

### docs/ Directory Owns:
- Full architecture documentation
- Detailed technical documentation
- API documentation
- Design documents

### walkthroughs/ Directory Owns:
- Interactive code walkthroughs (virgil format)
- Use the **virgil** skill to create walkthroughs

## Cross-Reference Patterns

Use these patterns for linking between files:

**From README.md:**
```markdown
See [Contributing](CONTRIBUTING.md) for development setup.
See [LICENSE](LICENSE) for license information.
```

**From CONTRIBUTING.md:**
```markdown
Follow our [Style Guide](STYLE_GUIDE.md) for code formatting.
For available commands, see [AGENTS.md](AGENTS.md#commands).
```

**From AGENTS.md:**
```markdown
For detailed style rules, see [STYLE_GUIDE.md](STYLE_GUIDE.md).
For full architecture documentation, see [docs/](docs/).
```

**From CLAUDE.md (redirect only):**
```markdown
Read [AGENTS.md](AGENTS.md) for all agent guidance.
```

## File Templates

Templates are available in `assets/templates/`:
- `README.md` - Project overview template
- `AGENTS.md` - AI guidance template (primary, all agent content)
- `CLAUDE.md` - Redirect to AGENTS.md
- `CONTRIBUTING.md` - Contribution guide template
- `STYLE_GUIDE.md` - Style guide template
- `LICENSE` - License text template
- `CODE_OF_CONDUCT.md` - Community guidelines template
- `SECURITY.md` - Security policy template
- `CHANGELOG.md` - Version history template

Each template contains:
- Section structure for the file
- Placeholder markers (`{{PROJECT_NAME}}`, `{{DESCRIPTION}}`, etc.)
- Cross-reference links already in place
- Comments explaining what goes in each section

## Handling Existing Documentation

When existing docs contain valuable content:

1. **Identify content type**: Is it a command, a style rule, a gotcha, setup instructions?
2. **Route to canonical location**: Move content to the file that owns that type of information
3. **Add cross-reference**: Replace moved content with a link to its new location
4. **Preserve unique insights**: Don't lose valuable information discovered over time

**Example migration:**
- Commands table in README → move to AGENTS.md, add link in README
- Style rules in CONTRIBUTING → move to STYLE_GUIDE, add link in CONTRIBUTING
- Full architecture in any file → move to docs/, add link
- Content in CLAUDE.md → move to AGENTS.md, make CLAUDE.md a redirect

## Reference Files

For detailed guidance on each file's purpose and structure:
- See `references/file-purposes.md`

## Common Patterns

### Monorepo Projects
- Note workspace/package structure in AGENTS.md
- Document per-package commands if applicable
- Consider package-level AGENTS.md files

### Python Projects
- Document virtual environment setup
- Include pip/poetry/uv commands
- Note type checking configuration

### TypeScript/JavaScript Projects
- Document Node version requirements
- Include npm/yarn/pnpm commands
- Note bundler configuration

### Rust Projects
- Document cargo commands
- Note feature flags
- Include clippy/rustfmt configuration

## Validation Checklist

Before finalizing documentation:

- [ ] Each file stays within its content boundaries
- [ ] No duplicate information across files
- [ ] All cross-references are valid
- [ ] Commands are copy-paste ready
- [ ] Quick start in README is actually quick (3-5 steps)
- [ ] Style rules are in STYLE_GUIDE.md only
- [ ] All AI-specific content is in AGENTS.md (single source of truth)
- [ ] CLAUDE.md only contains a redirect to AGENTS.md
- [ ] License text is in LICENSE (no extension, README just links to it)
- [ ] Full architecture is in docs/ (AGENTS.md has overview only)
