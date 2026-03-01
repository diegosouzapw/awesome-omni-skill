---
name: ai-tuning
description: Optimize AI assistant configurations (CLAUDE.md, copilot-instructions.md, AGENTS.md, MCP). Deduplicate memory files. Use when asked to "improve CLAUDE.md", "better copilot instructions", "tune AI", "condense CLAUDE.md", or "optimize prompts".
user-invocable: true
disable-model-invocation: false
allowed-tools: [Read, Write, Edit, Glob, Grep]
---

# AI Tuning

Optimize AI assistant configurations. Standards: `instructions/ai-tuning.instructions.md`

<instructions>

## Triggers

| User says | Target |
|-----------|--------|
| "improve CLAUDE.md" | CLAUDE.md |
| "better copilot instructions" | copilot-instructions.md |
| "condense/deduplicate" | CLAUDE.md hierarchy |
| "tune AI" | All configs |
| "optimize prompts" | Prompts, AGENTS.md |
| "add MCP servers" | .vscode/mcp.json |

## Workflow

1. **Analyze**: Read all existing AI config files in the project
   - CLAUDE.md, AGENTS.md, copilot-instructions.md
   - `.claude/rules/*.md`, `.github/copilot-instructions.md`
   - `.vscode/mcp.json`, `.cursor/rules/`

2. **Identify gaps**: Check for:
   - Missing context (tech stack, conventions, tool preferences)
   - Vague instructions ("write good code" vs "use ruff for linting")
   - Outdated commands or patterns
   - Cross-file duplication
   - Misplaced content (subdir file with project-wide rules)

3. **Optimize**: Apply these principles:

   <optimization_rules>
   - Dense over verbose: `Python 3.12 | ruff | pytest` not paragraphs
   - Examples over descriptions: code block with `...` over "use type annotations"
   - Tables over prose: `| Action | Command |` for build/test/lint
   - Specific over general: exact commands, exact tool names
   - Hierarchy over flat: root CLAUDE.md (project) > topic rules > subdir rules
   </optimization_rules>

4. **Validate**: Every command must be exact and executable in the project

## Condensation (CLAUDE.md Deduplication)

Think through deduplication systematically:

| Phase | Action |
|-------|--------|
| Discovery | Find all CLAUDE.md files; detect intra-file and cross-file duplication |
| Analysis | Identify misplaced content (subdir file with project-wide rules) |
| Present | Show duplicates, affected files, proposed consolidation; wait for approval |
| Implement | Remove duplicates, move misplaced content, merge similar rules |

**Hierarchy**: `./CLAUDE.md` (project) > `./.claude/rules/*.md` (topic) > `./subdir/CLAUDE.md` (dir-only)

</instructions>

<examples>

### Before optimization (verbose, vague)
```markdown
# Project Guidelines
We use Python for our backend. Please make sure to use type hints
when writing Python code. For testing, we prefer pytest. Please run
the linter before committing any code.
```

### After optimization (dense, actionable)
```markdown
# Stack
Python 3.12 | FastAPI | PostgreSQL | Redis

# Dev Commands
| Task | Command |
|------|---------|
| Lint | `ruff check --fix .` |
| Format | `ruff format .` |
| Test | `pytest -x --tb=short` |
| Types | `mypy src/` |

# Conventions
- Type hints on all public functions
- `result: T | None` not `Optional[T]`
- Pydantic models for API schemas
```

### Deduplication example
```
Found duplication:
  ./CLAUDE.md line 15: "Use ruff for linting"
  ./backend/CLAUDE.md line 3: "Always run ruff before committing"
  ./.claude/rules/python.md line 8: "Lint with ruff check"

Action: Keep in ./CLAUDE.md only (project-wide rule).
Remove from backend/CLAUDE.md and .claude/rules/python.md.
```

</examples>
