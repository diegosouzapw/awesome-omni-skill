---
name: utils:find-claude-plugin-root
description: This skill should be used when the user needs to locate a plugin's installation path, when ${CLAUDE_PLUGIN_ROOT} doesn't expand in markdown files, or when invoked via /utils:find-claude-plugin-root. Generates a CPR resolver script at /tmp/cpr.py.
version: 0.1.0
---

# Find Claude Plugin Root

This skill generates a Python resolver script at `/tmp/cpr.py` that locates a plugin's installation directory by reading `~/.claude/plugins/installed_plugins.json`.

## Problem It Solves

`${CLAUDE_PLUGIN_ROOT}` environment variable doesn't expand in markdown command files, making it impossible to reference plugin scripts and resources. This is a known issue: https://github.com/anthropics/claude-code/issues/9354

## Solution

Generate a Python script that:
1. Accepts plugin name as argument
2. Tries `${CLAUDE_PLUGIN_ROOT}` first (backwards compatible)
3. Reads installed_plugins.json and searches for exact match
4. If no exact match, finds similar plugin names (fuzzy matching)
5. Outputs the plugin installation path to stdout
6. Saves to `/tmp/cpr.py` (ephemeral, no project pollution)

## Usage

Invoke this skill before executing plugin scripts:

```bash
# Generate the resolver
Skill(skill="utils:find-claude-plugin-root")

# Use it to find a plugin and execute its scripts
PLUGIN_ROOT=$(python3 /tmp/cpr.py readme-and-co)
python "$PLUGIN_ROOT/scripts/populate_license.py" --license MIT
```

## Implementation

### Step 1: Create the CPR resolver Python script

```bash
cat > /tmp/cpr.py << 'CPREOF'
#!/usr/bin/env python3
"""
Claude Plugin Root (CPR) Resolver

Usage: python3 /tmp/cpr.py <plugin-name>
Returns: absolute path to plugin installation directory

Searches for plugins in ~/.claude/plugins/installed_plugins.json with fuzzy matching.
"""

import json
import os
import sys
from pathlib import Path
from difflib import SequenceMatcher


def similarity(a, b):
    """Calculate similarity ratio between two strings."""
    return SequenceMatcher(None, a.lower(), b.lower()).ratio()


def find_plugin_root(plugin_name):
    """
    Find plugin installation directory.

    Returns: (plugin_root_path, match_type)
        match_type: 'env_var', 'exact', 'fuzzy', or None
    """
    # Try CLAUDE_PLUGIN_ROOT first (backwards compatible)
    env_root = os.environ.get('CLAUDE_PLUGIN_ROOT')
    if env_root and os.path.isdir(env_root):
        return env_root.rstrip('/'), 'env_var'

    # Read installed_plugins.json
    plugins_file = Path.home() / '.claude' / 'plugins' / 'installed_plugins.json'

    if not plugins_file.exists():
        return None, None

    try:
        with open(plugins_file, 'r') as f:
            data = json.load(f)
            plugins = data.get('plugins', {})
    except (OSError, json.JSONDecodeError):
        return None, None

    # Try exact match first (case-insensitive)
    for key, value in plugins.items():
        if plugin_name.lower() in key.lower():
            # Handle list or dict value
            if isinstance(value, list) and len(value) > 0:
                value = value[0]
            install_path = value.get('installPath', '').rstrip('/')
            if install_path and os.path.isdir(install_path):
                return install_path, 'exact'

    # Try fuzzy matching if no exact match
    matches = []
    for key, value in plugins.items():
        # Extract just the plugin name from key (e.g., "owner/plugin-name" -> "plugin-name")
        key_parts = key.split('/')
        plugin_part = key_parts[-1] if key_parts else key
        # Also handle @ separator (e.g., "plugin-name@plugin-name")
        plugin_part = plugin_part.split('@')[0]

        ratio = similarity(plugin_name, plugin_part)
        if ratio > 0.6:  # 60% similarity threshold
            # Handle list or dict value
            if isinstance(value, list) and len(value) > 0:
                value = value[0]
            install_path = value.get('installPath', '').rstrip('/')
            if install_path and os.path.isdir(install_path):
                matches.append((ratio, install_path, key))

    if matches:
        # Return best match
        matches.sort(reverse=True, key=lambda x: x[0])
        best_match = matches[0]
        return best_match[1], 'fuzzy'

    return None, None


def main():
    if len(sys.argv) < 2:
        print("Usage: python3 /tmp/cpr.py <plugin-name>", file=sys.stderr)
        print("Example: python3 /tmp/cpr.py readme-and-co", file=sys.stderr)
        sys.exit(1)

    plugin_name = sys.argv[1]
    plugin_root, match_type = find_plugin_root(plugin_name)

    if plugin_root:
        # Output just the path to stdout (for command substitution)
        print(plugin_root)
        sys.exit(0)
    else:
        print(f"Error: Could not locate plugin '{plugin_name}'", file=sys.stderr)
        print(f"Checked: $CLAUDE_PLUGIN_ROOT, ~/.claude/plugins/installed_plugins.json", file=sys.stderr)
        sys.exit(1)


if __name__ == '__main__':
    main()
CPREOF

chmod +x /tmp/cpr.py
```

### Step 2: Verify the script works

```bash
# Test finding the utils plugin itself
if PLUGIN_ROOT=$(python3 /tmp/cpr.py utils); then
  echo "✓ CPR resolver created at /tmp/cpr.py"
  echo "✓ Test lookup succeeded: $PLUGIN_ROOT"
else
  echo "❌ CPR resolver test failed" >&2
  exit 1
fi
```

## Examples

### Find and use readme-and-co plugin

```bash
# Invoke this skill first
Skill(skill="utils:find-claude-plugin-root")

# Then use the resolver - run script and capture output
PLUGIN_ROOT=$(python3 /tmp/cpr.py readme-and-co)
python "$PLUGIN_ROOT/scripts/detect_project_info.py"
```

### Find and use any plugin

```bash
# The resolver works for any plugin
PLUGIN_ROOT=$(python3 /tmp/cpr.py my-plugin)
node "$PLUGIN_ROOT/tools/analyzer.js"
```

## Benefits

- **No project pollution** - Script saved to /tmp, not in project
- **Backwards compatible** - Tries ${CLAUDE_PLUGIN_ROOT} first
- **Fuzzy matching** - Finds plugins even if name doesn't exactly match
- **Pure Python** - No external dependencies (jq not needed)
- **Reusable** - One skill for all plugins
- **Ephemeral** - /tmp/cpr.py cleaned up on reboot

## Limitations

- Recreated on each system reboot (since /tmp is ephemeral)
- Requires Python 3 (standard on all modern systems)
