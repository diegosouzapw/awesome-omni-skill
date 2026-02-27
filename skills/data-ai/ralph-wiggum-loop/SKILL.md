---
name: ralph-wiggum-loop
description: The Ralph Wiggum technique for autonomous AI development loops. Use when the user wants to run an agent in a loop for greenfield projects, reverse engineering, code migrations, large refactors, overnight batch work, or any task requiring persistent iteration until completion. Triggers include "ralph loop", "gumloop", "run ralph", "autonomous loop", "overnight coding", "while loop agent", "iterative development", "let it ralph", "run until done", "agentic loop", "context rotation", or when users want to set up autonomous coding that runs unattended. This skill implements Geoffrey Huntley's original philosophy: context rotation with git persistence, not complex multi-agent orchestration.
---

# The Ralph Wiggum Loop

> "Ralph is a loop." — Geoffrey Huntley

## The Core Insight: Context Rotation

LLM context windows are like memory with `malloc()` but no `free()`.

As an agent works, context fills:
```
Start:        [PROMPT]
Read files:   [PROMPT][file1][file2][file3]
Tool calls:   [PROMPT][file1][file2][file3][results][more_results]
Errors:       [PROMPT][file1][file2][file3][results][more_results][stack_trace]
Eventually:   [PROMPT][file1][file2][...everything...][gutter]
```

When context fills with stale information, the agent enters the **"gutter"** — poor decisions because critical information is buried. There's no way to selectively free context.

**The solution: loop termination IS the `free()`.**

```
Iteration 1: Fresh context → work → commit → EXIT (freed)
Iteration 2: Fresh context → work → commit → EXIT (freed)
Iteration 3: Fresh context → work → commit → EXIT (freed)
```

Each iteration:
1. Starts with **zero memory** from previous iteration
2. Loads only the prompt (small, deterministic)
3. Reads current state from **disk** (git, files)
4. Does **ONE task**
5. Commits progress
6. Exits → context garbage collected

**Progress persists in files and git, not in context.**

## The CLI: gumloop

```bash
# Install
curl -fsSL https://raw.githubusercontent.com/adriancodes/gumloop/main/install.sh | bash

# Quick start
cd my-project
gumloop init
gumloop run -p "Fix all failing tests" --choo-choo
```

### Run Commands

```bash
gumloop run -p "Fix the tests"                # Run once with inline prompt
gumloop run --prompt-file PROMPT.md            # Run once with prompt file
gumloop run --choo-choo -p "Fix bugs"          # Loop until no changes
gumloop run --choo-choo 20 -p "Migrate to TS"  # Loop, max 20 iterations
gumloop run --model sonnet -p "..."            # Use a specific model
gumloop run --verify "npm test" -p "..."       # Run tests after each iteration
gumloop run --memory --choo-choo -p "..."      # Session memory between runs
```

### Other Commands

```bash
gumloop init                    # Interactive setup wizard
gumloop config show             # Show effective merged config
gumloop config set cli codex    # Set project config
gumloop memory show             # Display session memory
gumloop memory clear            # Delete session memory file
gumloop recover                 # Discard uncommitted changes
gumloop recover 3               # Reset last 3 commits
gumloop update                  # Update to latest version
gumloop uninstall               # Remove gumloop
```

### Supported Agents

```bash
gumloop run --cli claude -p "..."       # Claude Code (default)
gumloop run --cli codex -p "..."        # OpenAI Codex
gumloop run --cli gemini -p "..."       # Google Gemini
gumloop run --cli cursor -p "..."       # Cursor Agent CLI
gumloop run --cli opencode -p "..."     # OpenCode
gumloop run --cli ollama -p "..."       # Ollama (local)
```

### Loop Behavior

- Without `--choo-choo`: runs once and exits
- With `--choo-choo`: loops until no git changes detected
- Stuck detection: exits after N iterations with changes but no commits (default: 3)
- `--memory`: saves session state so the next run picks up where it left off
- Ctrl+C to exit cleanly at any time

## Why One Task Per Loop

Multiple tasks = context accumulation = gutter state.

One task = fresh context = sharp reasoning = atomic commits.

Your prompt should say "pick the most important task" not "do all tasks."

## The Philosophy

### Deterministically Bad

> "That's the beauty of Ralph - the technique is deterministically bad in an undeterministic world."

Ralph will fail. Failures are predictable and fixable. Better than succeeding unpredictably.

### Eventual Consistency

If the agent keeps correcting itself, it will eventually converge. Each iteration sees results of previous iterations through git and files.

### LLMs Are Mirrors

Success depends on writing good prompts. Quality of output reflects quality of input.

### Monolithic

Single process, single repository, one task per loop. NOT multi-agent orchestration or loops within loops.

## Tuning: The Playground Metaphor

> "Ralph is given instructions to construct a playground. Ralph comes home bruised because he fell off the slide, so you add a sign: 'SLIDE DOWN, DON'T JUMP.' Eventually all Ralph thinks about is the signs — that's a tuned prompt."

Start minimal. Observe failures. Add specific guardrails.

## Minimal Setup

```bash
mkdir project && cd project
git init

# Run once
gumloop run -p "What does this codebase do?"

# Loop until done
gumloop run -p "Fix all failing tests" --choo-choo
```

## Spec-Driven Workflow

For large projects, use three phases: **Spec → Plan → Execute**.

**1. Create Spec** — Write `SPEC.md` with architecture, components, task backlog.

**2. Generate Plan** — Run once to create lightweight task list:
```bash
gumloop run -p "Read SPEC.md. Create IMPLEMENTATION_PLAN.md with checkboxes for each task. Planning only."
```

**3. Execute from Plan** — Loop with a prompt that references both:
```bash
gumloop run --choo-choo -p "Read IMPLEMENTATION_PLAN.md. Find first unchecked task. Read SPEC.md for details. Implement it. Mark done. Commit with task ID."
```

The plan is a lightweight index into the heavy spec. Agent reads plan to know *what*, spec to know *how*.

## Safety

**Built-in checks:**
- Refuses dangerous directories (~, /, /etc, /usr, /var, /tmp)
- Requires git repository
- Warns before choo-choo mode in home subdirectories

**Git is your safety net:**
```bash
gumloop recover       # Discard uncommitted changes
gumloop recover 3     # Reset last 3 commits
git reset --hard      # Instant recovery
```

**For true isolation:** E2B, Fly Sprites, Modal, dedicated VM.

## When to Use

**Good for:** Greenfield, reverse engineering, refactors, migrations, test coverage, documentation, overnight work.

**Not ideal for:** Judgment-heavy work, unclear completion criteria, highly exploratory research.

## References

- `references/context-engineering.md` — Why the loop works
- `references/workflow-patterns.md` — Two-mode, specs directories
- `references/prompt-templates.md` — Starter prompts
- `references/tuning-guide.md` — Guardrail patterns
- `references/safety.md` — Safety considerations
