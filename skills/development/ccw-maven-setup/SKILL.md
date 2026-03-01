---
name: ccw-maven-setup
description: Prepares Maven build environment for Claude Code Web by installing Java 25 and configuring Maven proxy. Run automatically before Maven operations in CCW.
---

# CCW Maven Setup Skill

Prepares the Maven build environment for Claude Code Web (CCW) by installing Java 25 and configuring the Maven proxy. Does nothing in local environments.

## Purpose

This skill ensures that Maven builds work correctly in Claude Code Web by:
1. Installing Java 25 (required by this project)
2. Setting up a local Maven authentication proxy
3. Configuring Maven settings.xml

## When to Use

Run this skill **before any Maven build operations** when working in Claude Code Web. It's idempotent and safe to run multiple times.

## What It Does

1. **Detects environment**: Checks `CLAUDECODE` environment variable
2. **If in CCW**:
   - Installs SDKMAN (if not present)
   - Installs latest stable Java 25 (Temurin distribution) via SDKMAN
   - Starts local Maven authentication proxy at `127.0.0.1:3128`
   - Configures `~/.m2/settings.xml` with proxy settings
3. **If not in CCW**:
   - Does nothing (assumes local environment already has Java 25)

## Usage

Simply run the prep script from the skill directory:

```bash
bash support/shared/claude/skills/ccw-maven-setup/prep_if_ccw.sh
```

Or invoke as a skill (after properly registered):

```bash
/skill ccw-maven-setup
```

## Skill Workflow

When invoked, execute these steps:

1. **Run the preparation script**:
   ```bash
   bash "$CLAUDE_PROJECT_DIR/support/shared/claude/skills/ccw-maven-setup/prep_if_ccw.sh"
   ```

2. **Verify setup** by checking:
   - Java version: `java -version` (should show Java 25)
   - Maven proxy: `cat ~/.m2/settings.xml` (should show proxy config)
   - Proxy process: `pgrep -f maven-proxy.py` (should be running in CCW)

3. **Report results** to user with summary of what was set up

## Technical Details

### Why a Local Proxy?

Maven doesn't honor standard `HTTP_PROXY` environment variables for authentication. The local proxy at `127.0.0.1:3128` handles authentication transparently by:
- Accepting unauthenticated CONNECT requests from Maven
- Adding authentication headers when forwarding to CCW's upstream proxy
- Maintaining persistent connections for better performance

### Script Location

The `prep_if_ccw.sh` script is located in:
```
support/shared/claude/skills/ccw-maven-setup/prep_if_ccw.sh
```

This location is vendor-controlled and shared across projects that need CCW Maven support.

## References

- **GitHub Issue**: https://github.com/anthropics/claude-code/issues/13372
- **LinkedIn Article**: https://www.linkedin.com/pulse/fixing-maven-build-issues-claude-code-web-ccw-tarun-lalwani-8n7oc
- **SDKMAN**: https://sdkman.io/

## SessionStart Hook Integration

This skill is designed to run automatically via a SessionStart hook. See `.claude/settings.json` for hook configuration that runs this setup at session start.
