---
name: plugin-marketplace
description: Plugin marketplace — add, remove, list, and search skills from GitHub repositories
---

# /plugin marketplace — Skill Marketplace

Manage GGV3 skills sourced from GitHub repositories and npm packages.

## Commands

```
/plugin marketplace add <owner>/<repo>           # Install a skill from GitHub
/plugin marketplace remove <skill-name>          # Remove an installed skill
/plugin marketplace list                         # List installed skills
/plugin marketplace search <query>               # Search GitHub for skills
```

---

## `/plugin marketplace add <owner>/<repo>`

Install a skill or tool from a GitHub repository into `.agent/skills/`.

### Step 1 — Resolve the source

Determine source type from the identifier:

| Format                  | Source                  | Example                         |
| ----------------------- | ----------------------- | ------------------------------- |
| `owner/repo`            | GitHub repository       | `anthropics/claude-code`        |
| `owner/repo#skill-name` | Subfolder of a monorepo | `google/skills#web-perf`        |
| `npm:package-name`      | npm global package      | `npm:@anthropic-ai/claude-code` |

### Step 2 — Fetch metadata from GitHub

// turbo

```bash
gh repo view <owner>/<repo> --json name,description,url,defaultBranchRef 2>/dev/null || echo "REPO_NOT_FOUND"
```

If `REPO_NOT_FOUND`, check if it's an npm package instead:

```bash
npm view <package-name> name description version 2>/dev/null || echo "PKG_NOT_FOUND"
```

### Step 3 — Determine install type

**Type A — CLI Tool (npm package):**
If the source is an npm package or the repo's `package.json` has a `bin` field:

```bash
npm install -g <package-name>
```

Then create a thin skill wrapper:

```
.agent/skills/<tool-name>/
├── SKILL.md          # Documents the CLI, trigger patterns, usage
```

**Type B — Skill Repository (has SKILL.md):**
If the repo contains a `SKILL.md` at root or in a skill subdirectory:

```bash
# Clone to temp, extract skill files
git clone --depth 1 https://github.com/<owner>/<repo>.git /tmp/plugin-install-<repo>
cp -r /tmp/plugin-install-<repo>/ .agent/skills/<skill-name>/
rm -rf /tmp/plugin-install-<repo>
```

**Type C — Reference Repository (no SKILL.md, no bin):**
Create a new skill based on the repo's README and source code:

1. Read the repo's README via GitHub API
2. Analyze what the tool/library does
3. Create a SKILL.md that documents:
   - When to use the tool
   - How it was installed (CLI, library, etc.)
   - Key commands and patterns
   - Trigger keywords for auto-invoke

### Step 4 — Create/validate SKILL.md

Every installed plugin MUST have a valid SKILL.md with:

```yaml
---
name: <skill-name>
version: <semver or repo SHA>
source: <owner>/<repo> OR npm:<package>
installed: <ISO-8601 timestamp>
description: <one-line description>
---
```

Followed by:

- **When to use** — Clear trigger conditions
- **Installation** — How it was installed, dependencies
- **Usage** — Key commands, APIs, patterns
- **Examples** — At least one usage example

### Step 5 — Register the skill

**A) Add to `.agent/rules/skill-auto-invoke.md`:**

```markdown
| <skill-name> | .agent/skills/<skill-name>/SKILL.md | <trigger keywords> |
```

**B) Add to `CLAUDE.md` `<skill_registry>` section:**

```markdown
| <skill-name> | .agent/skills/<skill-name>/SKILL.md | <trigger keywords> |
```

> Note: `AGENTS.md`, `GEMINI.md`, and `codex.md` are symlinks to `CLAUDE.md` — updating one updates all.

### Step 6 — Verify installation

```bash
# For CLI tools — verify the binary exists
which <tool-name> && <tool-name> --version

# For skills — verify SKILL.md exists and is valid
cat .agent/skills/<skill-name>/SKILL.md | head -20
```

### Step 7 — Report installation summary

Output a summary:

```
✅ Plugin installed: <skill-name>
   Source: <owner>/<repo> (or npm:<package>)
   Type: CLI Tool | Skill | Reference
   Location: .agent/skills/<skill-name>/
   Triggers: <comma-separated trigger keywords>
```

---

## `/plugin marketplace remove <skill-name>`

### Step 1 — Verify skill exists

```bash
ls .agent/skills/<skill-name>/SKILL.md 2>/dev/null || echo "SKILL_NOT_FOUND"
```

### Step 2 — Check for npm global package

Read the SKILL.md frontmatter for a `source: npm:*` entry. If found:

```bash
npm uninstall -g <package-name>
```

### Step 3 — Remove skill directory

```bash
rm -rf .agent/skills/<skill-name>
```

### Step 4 — Deregister from skill registry

Remove the skill's row from:

- `.agent/rules/skill-auto-invoke.md`
- `CLAUDE.md` `<skill_registry>` section

### Step 5 — Confirm removal

```
🗑️ Plugin removed: <skill-name>
```

---

## `/plugin marketplace list`

### Step 1 — Enumerate installed skills

// turbo

```bash
for skill in .agent/skills/*/SKILL.md; do
  name=$(basename $(dirname "$skill"))
  source=$(grep -m1 '^source:' "$skill" 2>/dev/null | sed 's/source: *//')
  version=$(grep -m1 '^version:' "$skill" 2>/dev/null | sed 's/version: *//')
  echo "$name | $source | $version"
done
```

### Step 2 — Format as table

| Skill | Source | Version |
| ----- | ------ | ------- |
| ...   | ...    | ...     |

---

## `/plugin marketplace search <query>`

### Step 1 — Search GitHub

Use the GitHub MCP `search_repositories` tool:

```
query: "<query> topic:agent-skill OR topic:claude-skill OR topic:mcp-tool"
```

Or search for code patterns:

```
query: "SKILL.md name: <query>"
```

### Step 2 — Display results

| Repository | Stars | Description |
| ---------- | ----- | ----------- |
| owner/repo | ⭐ N  | ...         |

### Step 3 — Prompt for install

Ask: "Would you like to install any of these? Use `/plugin marketplace add <owner>/<repo>`"

---

## Known Plugin Sources

| Plugin      | Source                        | Type | Description                    |
| ----------- | ----------------------------- | ---- | ------------------------------ |
| claude-code | npm:@anthropic-ai/claude-code | CLI  | Anthropic's agentic coding CLI |

---

## Notes

- All skills are installed to `.agent/skills/<name>/`
- CLI tools are installed globally via npm and get a thin SKILL.md wrapper
- The SKILL.md frontmatter `source:` field tracks provenance for updates
- Use `installed:` timestamp to track when a plugin was added
