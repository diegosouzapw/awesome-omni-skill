---
name: ansible-fix
version: 1.0.0
description: ALWAYS use this skill immediately after running 'ansible-galaxy init' to create a new Ansible role. Also use when the user asks to fix ansible-lint errors or when ansible-lint output shows fixable issues like yaml[comments], schema[meta], meta-incorrect, name[play], or role-name[path] violations.
allowed-tools: Bash, Read, Write, Edit
changelog: CHANGELOG.md
---

# Ansible Lint Auto-Fixer

This skill automatically fixes common ansible-lint errors that can be easily automated, saving time when creating new Ansible roles.

## When to Use This Skill

**ALWAYS invoke this skill proactively:**
- Immediately after running `ansible-galaxy init roles/<name>`
- Do NOT wait for the user to ask - fix lint issues automatically
- This provides a better user experience by delivering clean, lint-free roles

**Also use when:**
- User explicitly asks to fix ansible-lint errors
- ansible-lint output shows any of these fixable errors:
  - `yaml[comments]` - Missing space after # in comments
  - `schema[meta]` - Unquoted version strings
  - `meta-incorrect` - Placeholder metadata values
  - `name[play]` - Unnamed plays
  - `role-name[path]` - Incorrect role import paths

## What It Fixes

The skill runs a Python script that automatically corrects:

1. **yaml[comments]**: Adds space after `#` in comments
   - `#comment` → `# comment`

2. **schema[meta]**: Quotes numeric version strings
   - `min_ansible_version: 2.1` → `min_ansible_version: "2.1"`

3. **meta-incorrect**: Replaces placeholder values with sensible defaults
   - `author: your name` → `author: Ansible User`
   - `company: your company (optional)` → `company: Community`
   - `license: license (GPL-2.0-or-later, MIT, etc)` → `license: MIT`

4. **name[play]**: Adds descriptive names to unnamed plays
   - Adds `name: Test playbook for <hosts>` to plays

5. **role-name[path]**: Fixes role import paths
   - `roles/nginx` → `nginx`

## Instructions

**IMPORTANT**: This skill should be invoked automatically whenever you create a new role with `ansible-galaxy init`. Do not wait for the user to ask - proactively fix lint issues to provide a clean role.

When invoked, follow these steps:

1. **Identify the role path** from context (e.g., if you just ran `ansible-galaxy init roles/nginx`, the path is `roles/nginx`). Do not ask the user - proceed automatically.

2. **Run the fixer script**:
   ```bash
   uv run .claude/skills/ansible-fix/ansible-lint-fix.py <role_path>
   ```

   Example:
   ```bash
   uv run .claude/skills/ansible-fix/ansible-lint-fix.py roles/nginx
   ```

3. **Review the output** to see what was fixed:
   - Number of files modified
   - Types of fixes applied
   - Count of each fix type

4. **Verify the fixes** by running ansible-lint:
   ```bash
   ansible-lint <role_path> 2>&1
   ```

5. **Report results** to the user:
   - Show summary of fixes applied
   - Confirm if all issues are resolved (0 failures, 0 warnings)
   - If issues remain, explain what still needs manual fixing

## Supporting Files

The skill uses the [ansible-lint-fix.py](ansible-lint-fix.py) script. This script:
- Scans all YAML files in the role directory
- Applies regex-based fixes for common patterns
- Reports detailed statistics on what was changed

## Examples

### Example 1: After creating a new role

```
User: I just created a new nginx role with ansible-galaxy init roles/nginx
Assistant: [Invokes ansible-fix skill]
- Runs: uv run .claude/skills/ansible-fix/ansible-lint-fix.py roles/nginx
- Reports: Fixed 12 issues across 6 files
- Verifies: ansible-lint shows 0 failures, 0 warnings
```

### Example 2: Fixing existing linting issues

```
User: Can you fix the ansible-lint errors in my web-server role?
Assistant: [Invokes ansible-fix skill]
- Runs: uv run .claude/skills/ansible-fix/ansible-lint-fix.py roles/web-server
- Reports: Fixed 8 issues across 4 files
- Verifies: ansible-lint results
```

## Best Practices

- Always run ansible-lint after using this skill to verify fixes
- The script is idempotent - safe to run multiple times
- Some issues may require manual intervention (explained in output)
- Review changes before committing to ensure they match project standards

## Limitations

This skill only fixes **syntactic** and **formatting** issues. It does not:
- Write role tasks or handlers
- Configure role variables
- Implement role logic
- Fix complex YAML structural problems beyond the patterns it recognizes
- Fix role naming issues (role names must match `^[a-z][a-z0-9_]*$` - lowercase, numbers, underscores only, no hyphens)

For complex linting issues or role development, manual intervention is needed.

**Note**: If ansible-lint reports `role-name` errors about the role name not matching the required pattern, the role directory itself needs to be renamed (e.g., `test-role` → `test_role`).
