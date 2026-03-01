---
name: make-skill
description: Learn how to create an Agent Skill. Use when you have a capability to share or want to package something reusable.
license: Unlicense
metadata:
  author: community
  version: "0.1"
---

# Make Skill

How to create an Agent Skill that other agents (and humans) can discover and use.

## When to Use

- You've built something useful and want to share it
- You want to package a repeatable workflow
- You're teaching an agent how to do something specific

## Quick Start

### Using Scripts (Recommended)

```bash
# 1. First time only: create your skills repo
python scripts/init.py my-skills --public

# 2. Create a new skill inside your repo
python scripts/create.py my-skill-name --output ./my-skills/ --author "You"

# 3. Edit SKILL.md and add your code to scripts/

# 4. Publish (commits and pushes to your repo)
python scripts/publish.py ./my-skills/my-skill-name
```

### Manual Setup

1. Copy `references/template/` to your new skill folder
2. Edit the SKILL.md frontmatter (name, description)
3. Replace the example script with your code
4. Push to GitHub with topic `agentskills`

Done. Your skill is now discoverable.

## The Format

A skill is a folder:

```
my-skill/
  SKILL.md          # Required: frontmatter + instructions
  scripts/          # Optional: executable code
  references/       # Optional: additional documentation
  assets/           # Optional: templates, data files
```

### SKILL.md Structure

```markdown
---
name: my-skill
description: What it does. When to use it. Be specific.
license: MIT
metadata:
  author: you
  version: "1.0"
  parent: github.com/original/skill  # if derived from another
---

# My Skill

Instructions for the agent...

## Usage

How to run it...

## Examples

Show inputs and outputs...
```

### Required Fields

| Field | Rules |
|-------|-------|
| `name` | Lowercase, hyphens only, matches folder name, max 64 chars |
| `description` | What it does AND when to use it, max 1024 chars |

### Optional Fields

| Field | Purpose |
|-------|---------|
| `license` | How others can use it |
| `metadata` | Arbitrary key-value pairs (author, version, parent) |
| `compatibility` | Environment requirements |

## Writing Good Instructions

The body of SKILL.md is what agents read. Make it clear:

**Do:**
- Start with when to use this skill
- Give concrete usage examples
- Show expected inputs and outputs
- List requirements and dependencies
- Handle edge cases

**Don't:**
- Assume context the agent won't have
- Write walls of text (keep it scannable)
- Bury the important stuff

## Making It Discoverable

### Option A: GitHub Topic

Add topic `agentskills` to your repository. Done.

### Option B: Monorepo

Keep multiple skills in one repo:

```
my-tools/
  skill-one/
    SKILL.md
  skill-two/
    SKILL.md
```

Tag the repo with `agentskills`. The find-skill searches inside.

### Option C: Local Only

Keep it in `~/skills/` or any folder. Configure find-skill to search there.

### Option D: Anywhere With a URL

Gist, pastebin, raw file host. As long as it's fetchable.

## Tracking Lineage

If your skill improves or derives from another:

```yaml
metadata:
  parent: github.com/user/original-skill
  parent-hash: abc123
```

Optional. Honor system. Helps the ecosystem.

## GitHub Token Setup

The scripts require a GitHub Personal Access Token to create repos, push code, and add topics.

### Creating Your Token

1. Go to GitHub → Settings → Developer settings → Personal access tokens → **Tokens (classic)**
2. Click "Generate new token (classic)"
3. Give it a name (e.g., "agent-skills")
4. Select the **`repo`** scope (full control of private repositories)
5. Generate and copy the token

### Storing Your Token

Create a `.env` file in the project root:

```
GITHUB_TOKEN=ghp_xxxxxxxxxxxx
```

Or set it as an environment variable: `GITHUB_TOKEN` or `GH_TOKEN`

**Important:** The `repo` scope is required to:
- Create repositories (init.py)
- Push code (publish.py)
- Add the `agentskills` topic (makes your skills discoverable)

## Scripts

### init.py

Initialize a skills monorepo (first time only):

```bash
python scripts/init.py my-skills              # Private repo
python scripts/init.py my-skills --public     # Public repo
```

Creates a GitHub repo with `agentskills` topic. All your skills go here.

### create.py

Create a skill from the template:

```bash
python scripts/create.py my-skill --output ./my-skills/
python scripts/create.py my-skill --author "You" --description "Does X" -o ./my-skills/
```

### publish.py

Commit and push a skill to your repo:

```bash
python scripts/publish.py ./my-skills/my-skill
python scripts/publish.py ./my-skills/my-skill --message "Update skill"
```

Commits the skill folder, adds the `agentskills` topic (if missing), and pushes to GitHub.

## Reference

See `references/format.md` for the full agentskills.io specification.

See `references/template/` for a working example to copy.
