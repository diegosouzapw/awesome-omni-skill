---
name: skill-writing
description: Create cross-agent compatible skills with SKILL.md format, supporting files, and scripts for Claude Code, Copilot, Cursor, Windsurf, and other AI coding assistants
---

# Writing Cross-Agent Compatible Skills

Create skills that work across multiple AI coding agents. Skills are reusable modules that package instructions, scripts, and resources to extend AI capabilities.

## Skill Structure

Skills are **directories** containing a `SKILL.md` file plus optional supporting resources:

```
my-skill/
├── SKILL.md              # Required: frontmatter + instructions
├── scripts/              # Optional: deterministic code
│   └── validate.sh
├── references/           # Optional: detailed documentation
│   └── api-details.md
└── examples/             # Optional: templates, samples
    └── output-format.md
```

## Where Skills Live

Skills can be placed at project level (shared with team) or user level (personal):

| Agent | Project Path | User Path |
|-------|-------------|-----------|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| GitHub Copilot | `.github/skills/` | `~/.copilot/skills/` |
| Cursor | `.cursor/skills/` | `~/.cursor/skills/` |
| Windsurf | `.windsurf/skills/` | `~/.codeium/windsurf/skills/` |
| Gemini CLI | `.gemini/skills/` | `~/.gemini/skills/` |

**For portability:** Create the skill once, copy the folder to each agent's skills directory.

## SKILL.md Format

Every skill starts with YAML frontmatter followed by markdown instructions:

```markdown
---
name: my-skill
description: Brief explanation of what this skill does and when to use it
---

# Skill Title

Instructions, workflows, and examples go here.
```

### Required Frontmatter

| Field | Rules | Purpose |
|-------|-------|---------|
| `name` | Lowercase, hyphens, numbers only. Max 64 chars. | Skill identifier and slash-command name |
| `description` | What it does and when to use it. Max 200 chars. | **Critical:** Agent uses this to decide if skill matches user request |

### Writing Effective Descriptions

The description determines when your skill activates. Include:

- **What the skill does** - Core capability
- **When to use it** - Trigger phrases users might say
- **Keywords** - Terms that match user intent

**Good:** `"Explains code with visual diagrams and analogies; use when user asks 'how does this code work?'"`

**Bad:** `"Code explanation tool"` (too generic, won't trigger reliably)

### Optional Frontmatter

These fields work on specific platforms (unsupported agents ignore them):

```yaml
---
name: my-skill
description: Does X when user asks Y
# Claude Code specific:
disable-model-invocation: true  # Only activate via /my-skill command
allowed-tools: Read, Grep, Glob  # Restrict available tools
context: fork                    # Run in sandboxed sub-agent
# General:
dependencies: python>=3.8, jq    # Required tools/versions
---
```

## Writing Skill Instructions

The markdown body provides the AI with guidance. Write as if directing the AI:

**Structure with clear steps:**
```markdown
## Workflow

1. First, check if the file exists using `ls`
2. Read the configuration from `config.yaml`
3. Validate the format matches the schema in `references/schema.md`
4. Run the validation script: `./scripts/validate.sh`
5. Report any errors with specific line numbers
```

**Keep under 500 lines.** Move extensive documentation to `references/` and mention it:
> "For full API details, see [references/api-details.md](references/api-details.md)"

The agent will load supporting files only when needed, preserving context tokens.

## When to Add Scripts

**LLMs are stochastic. Scripts are deterministic.**

Use scripts when you need reliable, repeatable results for:

- Data processing (parsing, sorting, transforming)
- Calculations (math, statistics, formatting)
- External API calls
- File format conversion
- Validation with specific rules

**Example:** A PDF parsing skill should include a Python script that actually extracts form fields, rather than having the AI attempt to parse binary data.

### Script Guidelines

1. Make scripts self-contained with clear input/output
2. Include error handling and meaningful exit codes
3. Document expected arguments in the script header
4. Test scripts work standalone before adding to skill

In SKILL.md, direct the agent when to use scripts:
```markdown
## Validation

Run the validation script to check format:
```bash
./scripts/validate.sh input.json
```

The script returns exit code 0 if valid, 1 if errors found.
```

## Progressive Disclosure

Skills load in stages to preserve context:

1. **Startup:** Agent sees only name + description (~100 tokens each skill)
2. **Activation:** When request matches description, full SKILL.md loads
3. **Resources:** Supporting files load only when explicitly referenced

Design skills to take advantage of this:

- Put essential instructions in SKILL.md
- Put detailed references, examples, and edge cases in supporting files
- Explicitly mention supporting files so the agent knows they exist

## Creating a New Skill

```bash
# 1. Create skill directory
mkdir -p .claude/skills/my-skill

# 2. Create SKILL.md
cat > .claude/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: Does X when user asks for Y
---

# My Skill

## Quick Reference

| Action | Command |
|--------|---------|
| Do X | `command-x` |

## Workflow

1. First step
2. Second step
3. Verify with `./scripts/check.sh`
EOF

# 3. Test by asking the agent to perform a matching task
```

## Testing Across Agents

1. Install skill in target agent's skills directory
2. Ask the agent to perform a task matching your description
3. Verify:
   - Skill activated (not just general response)
   - Instructions were followed
   - Scripts ran when appropriate
   - Supporting files loaded when referenced

If skill doesn't trigger, refine the description with better keywords.

## Security Considerations

Skills can include executable scripts. Only use skills from trusted sources.

**Before using third-party skills:**
- Review SKILL.md instructions
- Audit any scripts in `scripts/` directory
- Check what tools the skill requests access to

**When creating skills:**
- Don't include credentials or secrets
- Validate all inputs in scripts
- Use least-privilege tool permissions

## Validation Checklist

Before considering a skill complete:

- [ ] `name` field matches directory name exactly
- [ ] `name` is lowercase with hyphens only
- [ ] `description` under 200 chars with trigger keywords
- [ ] SKILL.md under 500 lines
- [ ] Supporting files referenced in SKILL.md exist
- [ ] Scripts are executable and tested standalone
- [ ] Examples include real, runnable commands
- [ ] Tested in at least one agent environment

## Example: Complete Skill

```markdown
---
name: api-testing
description: Run API endpoint tests and generate coverage reports; use when user asks to test or verify API endpoints
---

# API Testing

## Quick Reference

| Action | Command |
|--------|---------|
| Run all tests | `./scripts/run-tests.sh` |
| Test single endpoint | `./scripts/run-tests.sh /api/users` |
| Generate report | `./scripts/run-tests.sh --report` |

## Workflow

1. Identify endpoints to test from route definitions
2. Run test suite: `./scripts/run-tests.sh`
3. Review failures in output
4. For detailed test format, see [references/test-format.md](references/test-format.md)

## Writing New Tests

Tests follow the format in [examples/sample-test.json](examples/sample-test.json).

Required fields:
- `endpoint`: API path to test
- `method`: HTTP method
- `expected_status`: Response code to verify
```
