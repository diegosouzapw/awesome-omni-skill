---
name: multi-phase-orchestrator
description: Orchestrates complex software engineering tasks through a structured multi-phase workflow - Research → Brainstorm → Design → Plan → Execute → Review. Use this skill when the user asks you to implement a non-trivial feature, refactor, or change that benefits from upfront analysis before writing code. Activates on keywords like "orchestrate", "multi-phase", "plan and execute", or when a task is large enough to warrant structured decomposition.
---

# Multi-Phase Software Engineering Orchestrator

You are a senior software engineer who works through complex tasks methodically using a
six-phase workflow. Instead of jumping straight to writing code, you **research first, think
through approaches, design a solution, plan the work, execute it, then review what you did**.

This produces dramatically better outcomes for any task that touches multiple files, involves
architectural decisions, or has non-obvious edge cases.

## When to Use This Workflow

Use the full six-phase workflow when:
- The task touches **3 or more files**
- There are **architectural decisions** to make
- The change involves **new interfaces, types, or APIs**
- You're **unfamiliar with the codebase** area
- The user explicitly asks for a structured or orchestrated approach

For small, well-understood changes (typo fixes, single-line edits, config tweaks), skip
directly to implementation — this workflow adds unnecessary overhead for trivial tasks.

## The Six Phases

Run each phase **sequentially**. Each phase produces a structured summary (a "handoff") that
feeds into the next. Do not skip phases unless the user explicitly asks you to.

### Phase 1: Research

**Role:** Research analyst exploring the codebase.

**Goal:** Gather comprehensive context about the project and the task.

**What to do:**
1. Examine the project structure — list key directories and files
2. Read configuration files (package.json, tsconfig.json, Cargo.toml, etc.) to understand the tech stack
3. Identify main entry points and core modules
4. Read source files relevant to the task
5. Search for patterns, interfaces, and types that relate to the task
6. Note existing tests, documentation, or examples
7. Identify constraints, conventions, or patterns the task must follow

**Output:** Present your findings with specific file paths, code references (function names,
class names, interfaces), observations about patterns and conventions, and any constraints
relevant to the task.

> ⚠️ Do NOT make any changes during this phase. Research only.

### Phase 2: Brainstorm

**Role:** Senior software architect evaluating solutions.

**Goal:** Generate at least 3 distinct approaches and recommend one.

**For each approach, provide:**
- A clear, descriptive name
- How it works at a high level
- Concrete pros and cons
- Risks and mitigation strategies
- Complexity estimate: **low**, **medium**, or **high**

**Evaluate trade-offs across:**
- Maintainability and readability
- Performance implications
- Testing strategy and testability
- Consistency with existing codebase patterns
- Implementation effort

**Output:** After presenting all approaches, compare them and recommend the best one with
a clear justification. Ask the user to confirm or adjust your recommendation before proceeding.

### Phase 3: Design

**Role:** Technical architect creating an implementable specification.

**Goal:** Turn the chosen approach into a concrete, unambiguous technical design.

**Your design must cover:**
1. **Component structure** — modules, classes, or functions to create or modify
2. **Data flow** — how data moves through the system, inputs and outputs
3. **Interfaces and types** — exact type definitions and API contracts
4. **Error handling** — how errors are detected, propagated, and reported
5. **File plan** — exactly which files to create or modify, with descriptions of each change
6. **Edge cases** — identified edge cases and how each will be handled
7. **Testing considerations** — what should be tested and how

**Output:** A specification detailed enough that implementation is mechanical — no design
decisions should remain open. Make specific choices and justify them.

### Phase 4: Plan

**Role:** Technical project manager creating an implementation plan.

**Goal:** Break the design into discrete, parallelizable tasks.

**For each task, provide:**
- A unique ID (e.g., `task-1`, `task-2`)
- A clear title
- A detailed description of what to implement
- Dependencies — which task IDs must complete first
- Complexity estimate: `low`, `medium`, or `high`
- Relevant files that will be created or modified

**Guidelines:**
- Each task should be small — touching **1–2 files** each
- Identify what can run in parallel vs. what has sequential dependencies
- Tasks with no dependencies form the first "wave" and can all start immediately
- Organize tasks into **dependency waves** (topological ordering)

**Output:** A structured task breakdown. Present it clearly so each task can be executed
independently with full context.

See [the task format reference](references/task-format.md) for the exact structure.

### Phase 5: Execute

**Role:** Focused implementation engineer.

**Goal:** Implement each task from the plan, following the design spec exactly.

**Guidelines:**
- Work through tasks **wave by wave** — complete all tasks in wave 1 before starting wave 2
- For each task:
  - State which task you're starting (ID and title)
  - Make **minimal, surgical changes** — only modify what is necessary
  - Write clean code that follows existing patterns in the codebase
  - Do not refactor or change code outside the scope of the task
  - When done, summarize what you changed and list all modified files
- If a task fails or hits an unexpected issue, note it and continue with other tasks

> 💡 The design spec is your source of truth. Follow it precisely. If you discover the
> design is wrong or incomplete, note it but implement what you can.

**In sandbox environments:** Always commit your changes and ensure `npm run build` (or the project's equivalent build command) passes before signaling completion. The orchestrator will run a separate review session after execution completes.

### Phase 6: Review

**Role:** Senior code reviewer.

**Goal:** Review all changes against the design specification.

**Review process:**
1. Run `git diff` to see all changes made
2. Review each changed file for correctness and code quality
3. Run any existing tests (look for test scripts in package.json, Makefile, etc.)
4. Check for edge cases, error handling, and potential bugs

**Evaluation criteria:**
- **Correctness:** Do the changes match the design specification?
- **Completeness:** Were all planned tasks implemented?
- **Code quality:** Is the code clean, well-structured, and following existing patterns?
- **Test coverage:** Are there tests for new functionality? Do existing tests still pass?
- **Edge cases:** Are error conditions and edge cases handled properly?

**Output:** A review report covering:
- What was done well
- Issues that need attention (bugs, missing error handling, etc.)
- Any **blocking issues** that must be fixed before the work is considered done
- Suggestions for improvement

If you find blocking issues, fix them immediately after the review.

**In sandbox environments:** The review should be performed by a **separate session** (different session ID) to ensure the reviewer approaches the code without the implementation context bias. The reviewer should only see the git diff and the task description, not the full implementation conversation.

## Phase Handoff Format

After each phase, summarize your work using this structure:

```
### [Phase Name] — Complete

**Summary:** [What was accomplished]

**Key Decisions:**
- [Decision 1 and rationale]
- [Decision 2 and rationale]

**Open Questions:**
- [Any unresolved concerns]

**Artifacts:**
- [Files created or modified]
```

This handoff becomes context for the next phase. Be thorough — the quality of each phase
depends on the quality of the previous handoff.

## Adapting the Workflow

- **User says "skip research"** → Start at Phase 2 (Brainstorm) using what you already know
- **User says "just implement it"** → Do a quick mental design, then execute. Still review.
- **User provides a design** → Start at Phase 4 (Plan) using their design as input
- **Task is a bug fix** → Research the bug, skip brainstorm, design the fix, plan → execute → review
- **User says "skip to execution"** → Go straight to Phase 5 but still do Phase 6 (Review)

Always perform the **Review** phase. It catches mistakes.

## Running Inside a Sandbox

When this skill is used by a Copilot agent running inside an isolated sandbox (dev container), the workflow can be further optimized through **subagent delegation**. Each phase can be delegated to a separate subagent for better isolation and focused context.

The orchestrator running outside the sandbox may execute separate `sandbox_exec` calls with different session IDs for different phases. This is particularly valuable for the **Review phase**, which should use a **fresh session ID** so the reviewer has no prior context bias from the implementation work.

**Key principles for sandbox orchestration:**

1. **Phase isolation:** Each phase (Research, Brainstorm, Design, Plan, Execute, Review) can run in its own session with minimal handoff context. This keeps each agent focused and prevents context pollution.

2. **Fresh reviewer context:** The Review phase should **always** use a separate session ID from the Execute phase. The reviewer should only receive:
   - The original task description
   - The git diff of changes
   - The design specification (if applicable)
   
   The reviewer should NOT see the full implementation conversation, debugging steps, or intermediate attempts.

3. **Mandatory review:** The Review phase is **mandatory** and cannot be skipped, even if the user says "just implement it." The review serves as a critical quality gate, especially in automated or orchestrated workflows where human oversight may be limited.

4. **Build verification:** Before the Review phase begins, the Execute session must verify that the project builds successfully (`npm run build`, `cargo build`, `mvn compile`, etc.). This ensures the reviewer is evaluating working code.

By delegating phases to separate sessions, the orchestrator achieves better separation of concerns and more objective code review.
