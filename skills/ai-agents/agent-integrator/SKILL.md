---
name: agent-integrator
description: Use this skill to create or update the root AGENTS.md file to register AgenticDev skills for AI agent discovery. Triggers include "register AgenticDev", "update AGENTS.md", "setup agent guide", or initializing a new project.
---

# Agent Integrator Skill

## Purpose

Idempotently create or update the AGENTS.md file in a project to register AgenticDev skills for discovery by AI agents. This skill ensures that any compatible AI agent working in the repository can discover and use the AgenticDev methodology and available skills.

## When to Use

Use this skill in the following situations:

- After running project-init to initialize AgenticDev in a new project
- When installing AgenticDev skills in an existing project
- After adding new skills to the `.claude/skills/` directory
- Updating the agent guide with new workflow information
- Ensuring AI agents can discover available AgenticDev capabilities

## Prerequisites

- Project has `.claude/skills/` directory with AgenticDev skills installed
- Write permissions to project root directory
- Optional: Existing AGENTS.md file (script creates if missing)

## AGENTS.md Purpose

The AGENTS.md file serves as a discovery mechanism for AI agents:

- **Agent Discovery**: AI agents read this file to learn about available workflows
- **Methodology Documentation**: Explains AgenticDev philosophy and core principles
- **Skill Catalog**: Lists all available skills and their purposes
- **Getting Started**: Provides entry point for new agents working in the project

## Workflow

### Step 1: Determine If Update Is Needed

Check if AGENTS.md needs to be created or updated:

```bash
# Check if file exists
ls -la AGENTS.md

# Check if AgenticDev section exists
grep "SYNTHESIS_FLOW" AGENTS.md
```

### Step 2: Run the Helper Script

Execute the script to update AGENTS.md:

```bash
# Use default location (AGENTS.md in project root)
bash scripts/update-agents-file.sh

# Or specify custom location
bash scripts/update-agents-file.sh -f path/to/custom-agents.md
```

### Step 3: Understand What the Script Does

The helper script uses an idempotent update strategy:

1. **Creates file if missing**:
   - Uses `touch` to ensure target file exists
   - Safe to run even if file doesn't exist yet

2. **Checks for existing content**:
   - Looks for `<!-- SYNTHESIS_FLOW_START -->` marker
   - Determines if this is an update or initial creation

3. **Updates existing content**:
   - If markers found, replaces content between markers
   - Preserves any other content in the file
   - Uses awk to safely replace marked section

4. **Adds new content**:
   - If markers not found, appends AgenticDev guide to end of file
   - Adds both start and end markers for future updates

5. **Preserves other content**:
   - Only modifies content between markers
   - Safe to run multiple times (idempotent)
   - Won't overwrite other project documentation

### Step 4: Verify the Update

Check that AGENTS.md was updated correctly:

```bash
# View the file
cat AGENTS.md

# Verify markers are present
grep -A 5 "SYNTHESIS_FLOW_START" AGENTS.md
```

### Step 5: Commit the Changes

If the update looks correct, commit to the repository:

```bash
git add AGENTS.md
git commit -m "docs: Update AGENTS.md with AgenticDev guide"
git push
```

## Error Handling

### Permission Denied

**Symptom**: Script cannot write to AGENTS.md

**Solution**:
- Check file permissions: `ls -la AGENTS.md`
- Ensure you have write access to project root
- Run with appropriate permissions

### Marker Corruption

**Symptom**: Content between markers is malformed

**Solution**:
- Manually edit AGENTS.md to fix markers
- Ensure both `<!-- SYNTHESIS_FLOW_START -->` and `<!-- SYNTHESIS_FLOW_END -->` are present
- Re-run script to regenerate content

### Custom File Path Issues

**Symptom**: Script creates file in wrong location

**Solution**:
- Use `-f` flag with full path: `bash scripts/update-agents-file.sh -f /full/path/to/file.md`
- Verify path exists: `mkdir -p /path/to/directory`
- Check current working directory

## Notes

- **Idempotent design**: Safe to run multiple times without side effects
- **Preserves other content**: Only updates content between markers
- **Marker-based**: Uses HTML comments as markers (invisible in rendered markdown)
- **Default location**: AGENTS.md in project root (standard convention)
- **Custom locations**: Use `-f` flag for alternative file paths
- **Run after setup**: Typically run once after project-init, then rarely
- **Update when skills change**: Re-run if new skills are added or removed
- **AI agent discovery**: Helps agents understand available AgenticDev capabilities
- **Version control**: Commit AGENTS.md so all contributors see the guide
