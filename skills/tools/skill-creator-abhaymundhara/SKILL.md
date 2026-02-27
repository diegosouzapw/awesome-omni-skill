---
name: skill-creator
description: Create new skills for ChatDock
emoji: 🛠️
---

# Skill Creator

This skill helps you create new skills for ChatDock.

## What are Skills?

Skills are markdown-based instructions that extend the agent's capabilities. Unlike tools (which are code), skills are natural language instructions that guide the agent on how to accomplish specific tasks.

## Skill Format

Each skill lives in its own directory under `~/.chatdock/skills/` (user skills) or `src/server/skills/` (built-in skills).

Required file: `SKILL.md`

```markdown
---
name: my-skill
description: Short description of what this skill does
emoji: 🎯
requires:
  bins: ["required-cli-tool"]    # Optional: CLI tools needed
os: ["darwin", "linux"]          # Optional: Supported OS
---

# Skill Name

Instructions for the agent on how to use this skill.

## When to use

Describe trigger phrases and scenarios.

## How to use

Provide step-by-step instructions, commands, and examples.
```

## Creating a New Skill

### Step 1: Choose a name

Skill names should be:
- Lowercase with hyphens
- Descriptive but concise
- Unique (not conflicting with existing skills)

### Step 2: Create the directory

```bash
# User skill (persists across updates)
mkdir -p ~/.chatdock/skills/my-skill

# Or built-in skill (requires code access)
mkdir -p src/server/skills/my-skill
```

### Step 3: Create SKILL.md

```bash
cat > ~/.chatdock/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: What this skill does
emoji: 🎯
---

# My Skill

Instructions here...
EOF
```

### Step 4: Test the skill

Restart ChatDock or ask the agent to reload skills. Then try using the skill with a trigger phrase.

## Core Principles

### 1. Be Concise

Skills should be minimal but complete. Avoid unnecessary verbosity.

### 2. Provide Examples

Always include concrete examples with actual commands.

### 3. Define Triggers

Clearly state when the agent should use this skill.

### 4. Handle Errors

Include guidance for common error scenarios.

## Example: Creating a Docker Skill

```bash
mkdir -p ~/.chatdock/skills/docker

cat > ~/.chatdock/skills/docker/SKILL.md << 'EOF'
---
name: docker
description: Manage Docker containers and images
emoji: 🐳
requires:
  bins: ["docker"]
---

# Docker Skill

Use this skill for Docker container and image management.

## When to use

- "list my containers"
- "start/stop container X"
- "build docker image"
- "show docker logs"

## Common Commands

```bash
# List running containers
docker ps

# List all containers
docker ps -a

# Start a container
docker start <container-id>

# Stop a container
docker stop <container-id>

# View logs
docker logs -f <container-id>

# Build an image
docker build -t <name>:<tag> .

# Run a container
docker run -d --name <name> -p 8080:80 <image>
```

## Cleanup

```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove everything unused
docker system prune -a
```
EOF
```

## Skill Discovery

ChatDock looks for skills in:
1. `~/.chatdock/skills/` (user skills, higher priority)
2. `src/server/skills/` (built-in skills)

Skills are loaded at startup. To reload skills, restart the server.

## Tips

- Put more specific skills before general ones
- Use YAML frontmatter for metadata
- Include both quick-start and detailed usage
- Reference other skills if they complement each other
