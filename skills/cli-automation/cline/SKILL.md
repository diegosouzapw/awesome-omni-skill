---
name: "Cline"
description: "Tool: VS Code extension with autonomous mode"
version: "1.0.0"
category: "cli"
author: "Rulebook"
tags: ["cli", "cli-tool"]
dependencies: []
conflicts: []
---
<!-- CLINE:START -->
# Cline CLI Rules

**Tool**: VS Code extension with autonomous mode

## Quick Start

Install Cline extension in VS Code or use CLI mode.

## Usage

```bash
# In prompts, always reference standards:
"Follow @AGENTS.md. Implement [feature] with tests first (95%+ coverage)."

# Cline will:
- Read AGENTS.md
- Write tests
- Implement feature
- Run quality checks
```

## Workflow

1. Keep AGENTS.md open in workspace
2. Request features with "Follow @AGENTS.md" prefix
3. Review proposed changes before approval
4. Verify tests pass after implementation

**Critical**: Reference @AGENTS.md in every prompt for consistent output.

<!-- CLINE:END -->
