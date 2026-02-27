---
name: find-skills
description: Find and install agent skills with `npx playbooks find skill` and `npx playbooks add skill`. Use whenever a skill needs to be discovered or installed.
---

# find-skills

- Search: `npx playbooks find skill "<query>"`
- Search is lexical by default; use `--semantic` for natural language search
- Output is ordered by best match and prints ready-to-run install commands
- If a repo/source is known, you can list skills: `npx playbooks add skill <source> --list`
- List agent IDs: `npx playbooks list agents`
- Ask which agent(s) to install to and whether global (`-g`) or project
- Install a specific skill: `npx playbooks add skill <source> --skill <skill-name> -a <agent> [-g] [-y]`
- Install all skills from a source only if user requests: `npx playbooks add skill <source> --all`
