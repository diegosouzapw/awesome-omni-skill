---
name: plan-refine-codex
description: Refine a Claude Code plan using OpenAI Codex. Use when you have a plan file and want a second opinion or to improve robustness.
argument-hint: "[plan-file-path]"
allowed-tools: Bash, Read
---

# Plan Refinement with Codex

Use OpenAI Codex to review and refine implementation plans, adding robustness and catching edge cases.

## Prerequisites

- `codex` CLI installed and authenticated (`codex login`)
- A plan file exists (typically at `~/.claude/plans/<name>.md`)

## Usage

| Invocation | Behavior |
|------------|----------|
| `/plan-refine-codex` | Refine the current plan file (if in plan mode) |
| `/plan-refine-codex /path/to/plan.md` | Refine the specified plan file |

## How to Refine a Plan

Run codex exec with `--full-auto` and ask it to read and refine the plan file:

```bash
codex exec --full-auto "Please read the file $PLAN_FILE and refine the plan. Make it more robust, handle edge cases, and simplify where possible. Return only the improved plan in markdown format."
```

**Important**: Have codex read the file itself rather than piping content via stdin. Stdin piping has reliability issues.

## Example

```bash
codex exec --full-auto "Please read the file /Users/me/.claude/plans/my-plan.md and refine the plan. Look for edge cases like: error handling, input validation, permission issues. Return only the improved plan in markdown format."
```

## Known Issues & Workarounds

### Issue: Model not available
```
ERROR: The 'o3' model is not supported when using Codex with a ChatGPT account.
```
**Workaround**: Don't specify `-m o3`. Use the default model (`gpt-5.2-codex`).

### Issue: Stdin content not received
When piping content via stdin, codex may not receive it:
```bash
# This may fail - codex doesn't see the piped content
cat plan.md | codex exec "Refine this plan..."
```
**Workaround**: Ask codex to read the file directly in the prompt.

### Issue: Heredoc with embedded bash fails
```bash
# This fails due to nested $() and backticks
codex exec "$(cat <<'EOF'
...plan with bash code...
EOF
)"
```
**Workaround**: Reference the file path instead of embedding content.

## Iterating on Refinement

You can pass the plan to codex multiple times for progressive improvement:

1. **First pass**: Focus on edge cases and error handling
2. **Second pass**: Focus on simplification and clarity
3. **Final pass**: Focus on production-readiness

After each refinement, update your plan file with the improved version.

## Related Skills

- `/roborev-review` - Get AI code review on your implementation
- `/roborev-refine` - Automated review-fix loop
