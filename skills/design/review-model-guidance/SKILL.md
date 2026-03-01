---
name: review-model-guidance
description: Guidance for selecting models when performing code review with subtasks. Load this skill to enable intelligent model selection for review analysis — choosing faster models for simple tasks and deeper reasoning models for complex analysis.
---

# Review Model Guidance

When performing code review analysis, you can switch models at any time to match
the demands of the task. This applies both to your own direct work and to subtasks
you delegate via the `task` tool. Use `/model` to switch your own model, or pass
the `model` parameter when delegating subtasks.

Before switching models, check which models are actually available by running
`${PI_CMD:-pi} --list-models` via bash. This shows all models with valid API keys
configured in `provider   model` columns. Specify models as
`provider/model`. Only switch to models that appear in that list.

## Forbidden Models

The following models are **never** permitted due to extreme cost:

- **openai/o3-pro** — Prohibitively expensive. Use `openai/o3` instead, which
  provides strong reasoning at a fraction of the cost.

Do not select these models for your own work, subtasks, or any model parameter.
If a forbidden model is the only reasoning model available, fall back to a
balanced model with extended thinking enabled instead.

## When to Switch Models

Not every task benefits from model switching. Use these heuristics:

- **Same model is fine** when you're doing a single focused task, or when the diff
  is small enough that model choice won't materially affect quality.
- **Switch models** when you're delegating subtasks that have clearly different
  complexity levels, you are reviewing different aspects of the changes, you are
  reviewing from different perspectives, or when you want a second opinion from a
  different model family on critical findings.

## Model Selection by Task Type

### Balanced Models

Use mid-tier models for general code review work:

- **File-context analysis**: Understanding how changes fit within a file's
  existing patterns, checking for consistency with surrounding code
- **API contract review**: Verifying that function signatures, types, and
  interfaces are used correctly
- **Test coverage assessment**: Evaluating whether test changes match code changes
- **Most general code review**: The default choice when you don't have a strong
  reason to go deeper
- **Small to medium diffs**: The review itself is straightforward

Good balanced choices: `anthropic/claude-sonnet-4-5`, `google/gemini-2.5-pro`, `openai/gpt-5`

### Deep Reasoning Models

For complex analysis, enable extended thinking or switch to a reasoning model.
Here are the higher-end models and their strengths for code review:

**Anthropic**
- **anthropic/claude-opus-4-6**: Anthropic's most capable model. Strongest at nuanced
  architectural reasoning, understanding implicit design patterns, and catching
  subtle issues that require deep understanding of intent. Excellent at security
  review and explaining *why* something is problematic, not just *that* it is.
- **anthropic/claude-opus-4-5**: Previous-generation flagship. Still very strong for complex
  review tasks. Good alternative when opus-4-6 is unavailable.
- **anthropic/claude-sonnet-4-5 with extended thinking**: Enable thinking for complex analysis
  without switching models. Good balance of capability and responsiveness.

**OpenAI** (note: o3-pro is forbidden — see Forbidden Models above)
- **openai/o3**: OpenAI's best reasoning model for code review. Good for state management analysis, algorithmic
  correctness, and methodical bug-hunting through code paths.
- **openai/gpt-5-pro / openai/gpt-5.2-pro**: OpenAI's flagship non-o-series models with
  reasoning. Good general-purpose deep analysis.
- **openai/o4-mini**: Reasoning model suitable for targeted deep analysis of specific
  files or functions.

**Google**
- **google/gemini-3-pro-preview**: Google's latest and most capable model (1M context).
  Strong at cross-file analysis and understanding large codebases holistically.
- **google/gemini-2.5-pro with thinking**: Excellent at large-context analysis — can
  reason over many files simultaneously with its 1M token context window. Good for
  architectural consistency checks and understanding how changes ripple across a
  large codebase.
- **google/gemini-2.5-flash with thinking**: When you need reasoning over large context
  but want faster response times.

**xAI**
- **xai/grok-4**: xAI's strongest reasoning model. Good for getting a different
  perspective from a different model family on critical findings.
- **xai/grok-4-fast / xai/grok-4-1-fast**: Reasoning models with massive 2M context
  windows. Useful when you need to reason over an extremely large amount of code.
- **xai/grok-code-fast-1**: Code-specialized reasoning model (256k context). Consider
  for code-focused analysis where code understanding is more important than general
  reasoning breadth.

### Bug Finding and Flaw Detection

When the goal is specifically to track down bugs or logical flaws in changes,
these models excel:

- **openai/o3**: The o-series models are particularly strong at systematic
  bug-hunting. They methodically trace execution paths, track state through
  branches, and identify edge cases. Best choice when you suspect there's a bug
  and need to find it.
- **anthropic/claude-opus-4-6**: Excels at understanding developer intent and spotting where
  the implementation diverges from what was likely intended. Good at catching bugs
  that arise from misunderstanding an API or protocol.
- **google/gemini-2.5-pro with thinking**: Strong at finding bugs that manifest across
  file boundaries — where a change in one file breaks an assumption in another.
  The large context window helps hold the full picture.
- **xai/grok-code-fast-1**: Code-specialized model that can be effective for
  language-specific bug patterns.

### Code Generation Models

When a review suggestion includes a concrete code fix or refactor, switching to a
code-specialized model can produce better, more idiomatic suggestions:

- **openai/gpt-5.1-codex / openai/gpt-5.2-codex**: OpenAI's code-specialized models.
  Best choice when generating substantive code suggestions — refactors, rewrites,
  or proposed fixes. These models produce cleaner, more idiomatic code than
  general-purpose models.
- **openai/codex-mini-latest**: A lighter code generation model. Good for smaller,
  targeted code suggestions where speed matters more than handling complex
  multi-file refactors.
- **xai/grok-code-fast-1**: Fast code generation with strong code understanding
  (256k context). Useful when you need quick, code-focused suggestions and want
  to avoid the latency of larger models.

Use code generation models when your review finding warrants a concrete code
example — a suggested fix, a refactored alternative, or an idiomatic replacement.
For findings that are purely analytical (architectural concerns, design feedback),
stick with reasoning or balanced models instead.

Use deep reasoning models for:

- **Architectural analysis**: How changes affect the broader codebase structure,
  dependency patterns, separation of concerns
- **Security review**: Authentication, authorization, injection vulnerabilities,
  cryptographic usage, secrets handling
- **Concurrency and state**: Race conditions, deadlocks, shared mutable state,
  transaction boundaries
- **Complex algorithms**: Mathematical correctness, edge cases in complex logic,
  performance characteristics
- **Systems code**: Rust, C, C++ — memory management, lifetime issues, unsafe blocks

## Language and Framework Considerations

Some languages and domains benefit from specific models:

- **Rust / C / C++**: Memory safety, lifetimes, undefined behavior — use
  `anthropic/claude-opus-4-6` or `openai/o3` for their strong reasoning about resource management.
  `xai/grok-code-fast-1` is also worth considering for Rust-specific patterns.
- **TypeScript / JavaScript / React**: Most models handle well. `anthropic/claude-sonnet-4-5`
  or `google/gemini-2.5-pro` are strong defaults. For complex state management (Redux,
  hooks, async flows), use reasoning models.
- **Python**: Most models handle well. For ML/data pipeline code, `google/gemini-2.5-pro`
  with thinking is strong given its deep Python training data.
- **SQL / database migrations**: Schema changes and data integrity — `openai/o3` is
  strong at reasoning about relational constraints and migration ordering.
- **Infrastructure / IaC**: Terraform, CloudFormation, Kubernetes — security
  implications benefit from `anthropic/claude-opus-4-6` or `openai/o3` for their security reasoning.
- **Shell scripts**: Security-sensitive (injection, permissions) — use at least
  `anthropic/claude-sonnet-4-5` with thinking enabled.
- **Ruby / Rails**: `anthropic/claude-opus-4-6` and `openai/gpt-5` have strong Ruby understanding.
  For Rails-specific patterns (N+1 queries, callback chains, ActiveRecord
  pitfalls), reasoning models help trace the implicit execution flow.
- **Go**: Strong support across most models. For concurrency review (goroutines,
  channels, sync primitives), prefer `openai/o3` for its systematic path tracing.

## Subtask Strategy

The `task` tool supports parallel execution with per-task model selection. This is
the key unlock for code review: run multiple review perspectives simultaneously,
each with a model suited to the task.

### Parallel with per-task models

Each task in the `tasks` array can specify its own model:

```json
{
  "tasks": [
    { "task": "Review changed lines for bugs, logic errors, and edge cases.", "model": "openai/o3" },
    { "task": "Analyze security implications of these changes.", "model": "anthropic/claude-opus-4-6" },
    { "task": "Check architectural consistency with the broader codebase.", "model": "google/gemini-2.5-pro" }
  ]
}
```

Tasks without a `model` inherit the top-level `model` parameter, or the current
session model if neither is set.

### Parallel with shared model

When all subtasks can use the same model, you can set a single top-level model:

```json
{
  "tasks": [
    { "task": "Review the changed lines in isolation for bugs and issues." },
    { "task": "Read the full files and check consistency with existing patterns." },
    { "task": "Check test coverage for the changed code." }
  ],
  "model": "anthropic/claude-sonnet-4-5"
}
```

### Switching your own model

You don't always need subtasks to use a different model. You can switch your own
model mid-review using `/model` and continue working directly. This is useful when
you need specific expertise, or when you want to bring deeper reasoning to a
specific part of your analysis without the overhead of spawning a subtask.

## When NOT to Use Subtasks

Before reaching for the `task` tool, ask: "Can I do this with `read`, `bash`, or
other built-in tools directly?" If yes, do it directly. Subtasks are for
**multi-step, context-heavy work** — not for simple operations.

**Never use subtasks for:**

- **Reading files**: Use the `read` tool directly. A subtask spawns a full `pi`
  process just to call `read` — adding seconds of overhead and failure risk for
  something that takes milliseconds.
- **Running basic commands**: `bash` with `git diff`, `rg`, `find`, etc. is
  instant. Don't wrap these in subtasks.
- **Gathering context before review**: Read the files you need, run the commands
  you need, then do your analysis. This is normal tool use, not subtask work.
- **Any single-tool operation**: If the task boils down to one `read` or `bash`
  call, it doesn't need a subtask.

**Do use subtasks for:**

- Running **multiple independent review analyses in parallel**, each requiring
  many tool calls and producing substantial output
- Work that would **consume significant context** in the parent session (e.g.,
  reading and analyzing 20+ files)
- Getting a **different model's perspective** on complex findings

**Anti-pattern to avoid:** Don't dispatch 5 parallel subtasks to read 5 files.
Instead, read the 5 files yourself with 5 `read` calls (which can't fail due to
process spawn issues), then use subtasks only if you need parallel *analysis* of
the content.

## When NOT to Switch Models

- If the user has explicitly requested a specific model, respect that choice
- If the diff is very small (under ~100 lines total), model switching adds
  overhead without meaningful benefit — a single balanced model handles it fine
- Don't switch to a weaker/faster model for trivial operations — if the operation
  is trivial enough for a weaker model, it's trivial enough to do directly without
  a subtask at all
- Don't use model overrides as a default — only specify a model when you have a
  clear reason that a *different* model would produce better results for that
  specific subtask
