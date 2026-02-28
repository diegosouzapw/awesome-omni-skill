---
name: conda-env-setup
description: This skill should be used when the user asks to "setup conda environment", "configure Python environment", "activate conda automatically", "set conda environment for workspace", or mentions conda environment activation for Claude Code. Provides automatic conda environment configuration for workspaces.
version: 1.0.0
---

# Conda Environment Setup Skill

Automatically configure conda environments for Claude Code workspaces using environment variables instead of shell activation.

## Purpose

Conda environments cannot be activated through standard `conda activate` in Claude Code's bash sessions because each command runs in a new, non-interactive shell. This skill provides an alternative approach using workspace-level environment variables to make Python interpreters and packages available without shell activation.

## When to Use

Invoke this skill when:
- User requests conda environment activation for a workspace
- Python commands fail due to missing packages or wrong interpreter
- User mentions "activate conda", "setup Python environment", or similar
- Workspace needs specific conda environment for development

## Core Concept

Instead of activating conda environments with `conda activate` (which requires interactive shells), configure the workspace's `.claude/settings.json` to prioritize the conda environment's paths in the `PATH` environment variable.

This works because:
1. Shell commands look up executables using PATH
2. First match in PATH gets used
3. No shell initialization required

## Setup Process

### Step 1: Gather Environment Information

Identify the conda installation and environment details:

```bash
# Locate conda installation
which conda

# List all environments
conda env list

# Find environment path
conda env list | grep "env_name"
```

Typical conda installation locations:
- Windows: `C:\ProgramData\anaconda3` or `C:\Users\<username>\anaconda3`
- Linux/macOS: `/home/<username>/anaconda3` or `/opt/anaconda3`

Environment paths typically follow:
- `<conda_base>/envs/<env_name>`
- Example: `C:\ProgramData\anaconda3\envs\hrms-algo`

### Step 2: Create or Update Workspace Configuration

Check if workspace already has configuration:

```bash
# Check for existing config
cat .claude/settings.json
```

If `.claude/settings.json` exists, update it. If not, create it with the following template:

```json
{
  "env": {
    "PATH": "<env_path>:<env_path>/Scripts:<conda_base>/condabin:${PATH}",
    "CONDA_DEFAULT_ENV": "<env_name>",
    "CONDA_PREFIX": "<env_path>",
    "PYTHONPATH": "<env_path>/Lib/site-packages:${PYTHONPATH}"
  }
}
```

**Windows path format for Git Bash:**
- Replace backslashes with forward slashes
- Use drive letter format: `/c/Program Files/anaconda3`
- Example: `/c/ProgramData/anaconda3/envs/hrms-algo`

**Linux/macOS path format:**
- Use standard Unix paths
- Example: `/home/user/anaconda3/envs/hrms-algo`

### Step 3: Verify Configuration

Test the configuration:

```bash
# Check environment variables
echo $CONDA_DEFAULT_ENV
echo $CONDA_PREFIX

# Verify Python interpreter
which python
python -c "import sys; print(sys.executable)"

# Verify package availability
python -c "import <package_name>; print(<package_name>.__version__)"
```

Expected results:
- `CONDA_DEFAULT_ENV` should show the environment name
- `which python` should point to the conda environment
- Package imports should work without errors

## Troubleshooting

### Python Not Found in Environment

If `which python` doesn't show the conda environment:

1. Verify PATH entries are correct
2. Check for path syntax errors (Windows: use forward slashes)
3. Ensure `.claude/settings.json` is valid JSON
4. Restart Claude Code to reload configuration

### Packages Not Found

If packages are missing:

1. Verify PYTHONPATH includes the environment's site-packages
2. Check that packages are actually installed in the conda environment:
   ```bash
   /path/to/conda/envs/env_name/python -m pip list
   ```
3. Install missing packages if needed

### Configuration Not Applied

If environment variables don't appear to be set:

1. Confirm `.claude/settings.json` is in the workspace root
2. Validate JSON syntax (no trailing commas, proper quotes)
3. Check for conflicting global settings in `~/.claude/settings.json`
4. Restart Claude Code session

## Additional Resources

### Validation Script

Use the bundled validation script to verify setup:

```bash
# Run validation
python .claude/skills/conda-env-setup/scripts/validate_env.py
```

### Reference Documentation

- **`references/windows-paths.md`** - Windows path conversion guide for Git Bash
- **`references/environment-variables.md`** - Detailed environment variable reference
- **`examples/example-config.json`** - Working configuration examples

## Quick Reference

### Environment Variable Template

```json
{
  "env": {
    "PATH": "<env_path>:<env_path>/Scripts:<conda_base>/condabin:${PATH}",
    "CONDA_DEFAULT_ENV": "<env_name>",
    "CONDA_PREFIX": "<env_path>"
  }
}
```

### Common Path Formats

**Windows (Git Bash):**
```
/c/ProgramData/anaconda3/envs/myenv
/d/Program Files/anaconda3/envs/myenv
```

**Linux/macOS:**
```
/home/user/anaconda3/envs/myenv
/opt/anaconda3/envs/myenv
```

### Validation Commands

```bash
echo $CONDA_DEFAULT_ENV    # Should show env name
which python               # Should show env python
python -c "import sys; print(sys.executable)"  # Full path
```

## Why This Works

**Problem:** `conda activate` requires interactive shell with initialization

**Solution:** Set PATH to prioritize conda environment executables

**Benefits:**
- No shell initialization needed
- Works across platforms
- Persistent across commands
- No subprocess overhead
