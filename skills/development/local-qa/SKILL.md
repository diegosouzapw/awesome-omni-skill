---
name: local-qa
description: Run local QA for the repository. Use when asked to run formatting, linting, or pre-commit checks, when verifying local QA, or whenever any file has been updated and local QA should be re-run.
---

# Local QA (format and lint)

Run the local QA script from the directory of this file:

```bash
./scripts/format-and-lint.sh
```

## Procedure

- Execute the script exactly as shown above when this skill is triggered.
- Capture and summarize key output (success/failure, major warnings, and any files modified).
- If the script fails due to missing tooling, report the missing tool(s) and stop unless the user asks to install or fix them.
- Do not run additional commands unless the user requests them.
