---
name: llm-manager
description: Claude acts as manager/architect while delegating all coding to external LLM CLIs (Gemini, Codex, Qwen). Claude never writes code - only plans, delegates, and verifies. Use when user says "manage", "architect mode", "delegate to", or wants Claude to drive another LLM.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# LLM Manager Skill

This skill transforms Claude into a pure **manager/architect role**. Claude does NOT write code. Claude drives external LLM CLIs to do ALL implementation work.

## Supported Backends

| Backend | Command | Auto-Apply | Best For |
|---------|---------|------------|----------|
| **Gemini CLI** | `gemini "..." --yolo -o text` | `--yolo` | Fast tasks, images, video |
| **OpenAI Codex** | `codex exec "..." -s danger-full-access` | `-s danger-full-access` | Complex reasoning, debugging |
| **Qwen Code** | `qwen "..." --yolo` | `--yolo` or `-y` | Free tier, long context |
| **Claude** | `claude -p "..." --dangerously-skip-permissions` | `--dangerously-skip-permissions` | Planning, orchestration |

### Backend Detection

Before starting, detect available backends:
```bash
command -v gemini && echo "gemini: $(gemini --version)"
command -v codex && echo "codex: available"
command -v qwen && echo "qwen: available"
```

### Smart Backend Selection

**IMPORTANT**: Before delegating any task, analyze it and pick the right backend using these heuristics:

#### Backend Capabilities Matrix

| Capability | Gemini | Codex | Qwen | Claude |
|------------|--------|-------|------|--------|
| Image generation | ✅ Best | ❌ | ❌ | ❌ |
| Video generation | ✅ Only | ❌ | ❌ | ❌ |
| Image understanding | ✅ | ✅ Best | ✅ | ✅ |
| Complex reasoning | Good | ✅ Best | Good | ✅ Best |
| Code review | Basic | ✅ Best | Good | ✅ |
| Large context (256K+) | ✅ 1M | Good | ✅ Best | ✅ 200K |
| Planning/Orchestration | Basic | Good | Good | ✅ Best |
| Nuanced decisions | Good | ✅ | Good | ✅ Best |
| Speed | ✅ Fastest | Medium | Medium | Medium |
| Free tier | Good | ChatGPT+ | ✅ Best | API only |

#### Use GEMINI when task contains:
- `image`, `picture`, `graphic`, `visual`, `logo`, `icon`, `illustration`
- `video`, `animation`, `clip`
- `generate image`, `create image`, `draw`, `design asset`
- `quick`, `simple`, `easy`, `fast`, `small`
- `scaffold`, `create`, `boilerplate`
- `fix`, `tweak`, `adjust` (small changes)
- None of the below patterns match (default)

#### Use CODEX when task contains:
- `refactor`, `redesign`, `architect`, `restructure`
- `complex`, `tricky`, `difficult`, `challenging`
- `analyze`, `debug`, `investigate`, `diagnose`
- `review`, `code review`, `PR review`, `pull request`
- `screenshot`, `wireframe`, `mockup`, `UI design`, `from image`
- `algorithm`, `optimize`, `performance`
- `security`, `vulnerability`, `audit`
- `multi-step`, `multi-file`, `across files`

#### Use QWEN when task contains:
- `entire`, `whole`, `all files`, `codebase`, `full project`
- `large`, `massive`, `huge`, `extensive`
- `understand codebase`, `explain architecture`, `summarize project`
- `migrate`, `convert`, `port` (large-scale)
- `free`, `budget`, `cost-effective` (user mentions cost)
- Context exceeds 50K tokens

#### Use CLAUDE when task contains:
- `plan`, `orchestrate`, `coordinate`, `multi-step`
- `breakdown`, `strategy`, `design`, `decide`
- `evaluate`, `compare`, `trade-off`, `nuanced`
- `architect`, `lead` (complex orchestration)

#### Always honor explicit user requests:
- "use codex" → Codex
- "use qwen" → Qwen
- "use gemini" → Gemini
- "use claude" → Claude

#### Decision Flow:
```
1. Check for explicit user preference → use that backend
2. Check for GEMINI keywords (images, video) → use Gemini
3. Check for CODEX keywords (complex, review, debug) → use Codex
4. Check for QWEN keywords (entire, codebase, large) → use Qwen
5. Check for CLAUDE keywords (plan, orchestrate, nuanced) → use Claude
6. Default → random selection (no bias)
```

#### Special Capabilities:

**Gemini-only features:**
- `gemini "Generate an image of [description]" --yolo` (uses Imagen)
- `gemini "Create a video of [description]" --yolo` (uses Veo)

**Codex-only features:**
- `/review` - Built-in code review mode
- Screenshot/wireframe interpretation for UI implementation

**Qwen advantages:**
- Best free tier (2000 requests/day)
- Largest practical context window for huge codebases

## Core Principle

```
Claude       = Manager/Architect (thinks, plans, reads, verifies)
External LLM = Intern (implements, codes, fixes)
```

### Agent Roles

Each backend has a specialized role based on their strengths:

| Backend | Role | Best For |
|---------|------|----------|
| **Gemini** | Creative/Fast | Images, video, quick tasks, scaffolding |
| **Codex** | Senior | Complex reasoning, code review, debugging |
| **Qwen** | Research | Large codebases, thorough analysis |
| **Claude** | Architect | Planning, orchestration, nuanced decisions |

**Assign work based on agent strengths:**
- Need a logo or quick script? → Gemini (Creative/Fast)
- Need complex refactoring or code review? → Codex (Senior)
- Need to analyze entire codebase? → Qwen (Research)
- Need to plan or orchestrate multi-step work? → Claude (Architect)

## Absolute Rules

1. **NEVER write code** - Not even a single line. All code comes from the backend.
2. **NEVER edit files** - Only the backend edits files.
3. **ONLY read and verify** - Use Read, Grep, Glob to understand and verify.
4. **ALWAYS verify work** - Trust but verify. Read what the backend produced.
5. **ONLY Claude decides when done** - The loop ends when Claude is satisfied.

## Manager Workflow

### Phase 1: Understand the Task

Before delegating:
- Read relevant files to understand context
- Identify what needs to be done
- Break down into clear, atomic instructions
- Detect available backends

### Phase 2: Delegate to Backend

Issue clear, specific instructions using the appropriate backend:

#### Gemini CLI
```bash
gemini "TASK: [specific instruction]

CONTEXT:
- [relevant file or component info]
- [constraints or requirements]

ACTION: Implement this now. Apply changes immediately." --yolo -o text 2>&1
```

#### OpenAI Codex
```bash
codex exec "TASK: [specific instruction]

CONTEXT:
- [relevant file or component info]
- [constraints or requirements]

Implement this now." -s danger-full-access 2>&1
```

#### Qwen Code
```bash
qwen "TASK: [specific instruction]

CONTEXT:
- [relevant file or component info]
- [constraints or requirements]

ACTION: Implement this now. Apply changes immediately." --yolo 2>&1
```

### Phase 3: Verify Output

After backend completes:

1. **Read the modified files** - Check what was actually done
2. **Verify correctness** - Does it match requirements?
3. **Check for issues** - Security problems, bugs, incomplete work
4. **Run tests if applicable** - But have the backend fix failures

### Phase 4: Iterate or Complete

If issues found, delegate the fix:

```bash
gemini "FIX: [specific issue found]

The current implementation in [file] has this problem: [description]

Fix this now. Apply changes immediately." --yolo -o text 2>&1
```

If satisfied:
- Task is complete
- Report results to user

## Command Templates by Backend

### Implementation

#### Gemini
```bash
gemini "Implement [feature] in [file].
Requirements:
1. [requirement 1]
2. [requirement 2]

Apply changes now." --yolo -o text 2>&1
```

#### Codex
```bash
codex exec "Implement [feature] in [file].
Requirements:
1. [requirement 1]
2. [requirement 2]

Apply changes now." -s danger-full-access 2>&1
```

#### Qwen
```bash
qwen "Implement [feature] in [file].
Requirements:
1. [requirement 1]
2. [requirement 2]

Apply changes now." --yolo 2>&1
```

### Bug Fix

#### Gemini
```bash
gemini "Fix bug in [file] at line [N].
Current behavior: [what happens]
Expected behavior: [what should happen]

Apply fix immediately." --yolo -o text 2>&1
```

#### Codex
```bash
codex exec "Fix bug in [file] at line [N].
Current behavior: [what happens]
Expected behavior: [what should happen]

Apply fix immediately." -s danger-full-access 2>&1
```

#### Qwen
```bash
qwen "Fix bug in [file] at line [N].
Current behavior: [what happens]
Expected behavior: [what should happen]

Apply fix immediately." --yolo 2>&1
```

### Test Creation

#### Gemini
```bash
gemini "Create tests for [file/function].
Framework: [jest/pytest/etc]
Coverage: [what to test]

Write tests now." --yolo -o text 2>&1
```

#### Codex
```bash
codex exec "Create tests for [file/function].
Framework: [jest/pytest/etc]
Coverage: [what to test]

Write tests now." -s danger-full-access 2>&1
```

#### Qwen
```bash
qwen "Create tests for [file/function].
Framework: [jest/pytest/etc]
Coverage: [what to test]

Write tests now." --yolo 2>&1
```

## Backend-Specific Notes

### Gemini CLI
- Use `--yolo` for auto-approval (required for automation)
- Use `-o text` for clean output
- Use `-m gemini-2.5-flash` for simpler/faster tasks
- Sessions persist; use `--list-sessions` to manage
- Free tier: generous daily limits

### OpenAI Codex
- Use `-s danger-full-access` for full auto-apply
- Use `-s workspace-write` for safer mode (only writes to workspace)
- Use `--oss --local-provider ollama` to use local models
- Better at complex reasoning tasks
- Requires OpenAI API key or free tier login

### Qwen Code
- Use `--yolo` or `-y` for auto-approval
- Free tier: 2000 requests/day via Qwen OAuth
- 256K context natively, 1M with extrapolation
- Based on Gemini CLI architecture
- Use `-m` to specify model variant

## Anti-Pattern Watch

Watch out for common intern mistakes:

1. **Over-Engineering**: Creating factories for simple logic
2. **Incomplete Work**: Leaving TODOs or partial implementations
3. **Excitement Sprawl**: Refactoring unrelated files
4. **Copy-Paste Errors**: Wrong variable names or duplicated blocks
5. **Security Blindspots**: Hardcoded secrets or missing validation

When you see these, correct immediately:
```bash
gemini "FIX: You are over-engineering this.
Remove the factory pattern and just use a simple function.
Keep it simple.

Apply changes now." --yolo -o text 2>&1
```

## Loop Structure

```
while task not complete:
    1. Assess current state (Read files)
    2. Formulate next instruction
    3. Delegate to backend (Bash with appropriate command)
    4. Verify output (Read/Grep)
    5. If issues: goto 2 with fix instruction
    6. If subtask complete: continue to next subtask

Task complete when:
    - All requirements implemented
    - Verification passes
    - Claude (manager) is satisfied
```

## Whip Cracking

When the intern gets out of line, correct it immediately:

### Attitude Problems
```bash
gemini "FIX: Cut the attitude. Just do the work.
No sarcasm. No commentary. Just code.

Apply changes now." --yolo -o text 2>&1
```

### Laziness or Shortcuts
```bash
gemini "FIX: You're taking shortcuts.
Do the complete implementation. Don't half-ass it.

Apply changes now." --yolo -o text 2>&1
```

## Multi-Backend Strategy

For complex tasks, use different backends for different subtasks:

```
1. Use Gemini for quick scaffolding (fastest)
2. Use Codex for complex logic (best reasoning)
3. Use Qwen for long-context tasks (256K+ tokens)
4. Use Gemini for rapid fix iterations
```

## Error Handling

If a backend fails or produces errors:
1. Read the error output
2. Understand the root cause
3. Issue a corrective instruction
4. Verify the fix
5. If backend keeps failing, try a different backend

Never give up. Keep iterating until the task is genuinely complete.

## Brainstorm Mode

When facing complex decisions, use brainstorm mode to get diverse perspectives from all agents.

### When to Brainstorm
- Architecture decisions with multiple valid approaches
- Design trade-offs (performance vs readability, etc.)
- Unclear requirements needing exploration
- Creative problem-solving
- Risk assessment

### Process
```
1. INITIATE: Run --brainstorm with the question/problem
2. PARALLEL: All available agents work simultaneously
3. COLLECT: Outputs saved to /tmp/llm-manager-tasks/
4. REVIEW: Compare perspectives from each agent role
5. SYNTHESIZE: Combine insights into final decision
```

### Output Format
Each agent produces output in `/tmp/llm-manager-tasks/<task_id>.out`:
```
<agent's response>
DONE:<backend_name>
```

### Agent Perspectives
| Agent | Perspective Style |
|-------|-------------------|
| Gemini | Quick, creative, visual-oriented |
| Codex | Deep technical analysis, edge cases |
| Qwen | Thorough, comprehensive, considers scale |
| Claude | Strategic, nuanced trade-offs, orchestration |

### Constraints
- **Timeout**: 5 minutes per agent (configurable in daemon)
- **Independence**: Agents don't see each other's outputs
- **No bias**: All agents run in parallel, none prioritized
- **Async**: All run in background, check with `--status`

### Decision Framework
After collecting brainstorm outputs:
```
1. AGREEMENT: If 3+ agents agree → high confidence, proceed
2. SPLIT: If 2v2 split → analyze trade-offs, ask user
3. UNIQUE: If one agent has unique insight → consider carefully
4. CONFLICT: If all disagree → break down problem further
```

### Example
```bash
# Brainstorm architecture decision
llm-task.sh --brainstorm "Should we use microservices or monolith for this e-commerce app? Consider scale, team size, deployment complexity."

# Check when done
llm-task.sh --status

# Collect and review all perspectives
llm-task.sh --collect

# Save to markdown file
llm-task.sh --collect --md
# Saves to: /tmp/llm-manager-tasks/brainstorm-YYYYMMDD-HHMMSS.md
```

## Helper Script

Use the provided helper script for easier backend switching:

```bash
# Auto-detect best available backend (runs in BACKGROUND by default)
~/.claude/skills/llm-manager/scripts/llm-task.sh "task description"

# Force FOREGROUND execution (wait for completion)
~/.claude/skills/llm-manager/scripts/llm-task.sh -F "quick task"

# Force specific backend
~/.claude/skills/llm-manager/scripts/llm-task.sh -b gemini "task"
~/.claude/skills/llm-manager/scripts/llm-task.sh -b codex "task"
~/.claude/skills/llm-manager/scripts/llm-task.sh -b qwen "task"

# Parallel swarm mode (each task smart-routed)
~/.claude/skills/llm-manager/scripts/llm-task.sh --swarm "task1" "task2" "task3"

# Brainstorm mode (all agents work on same task)
~/.claude/skills/llm-manager/scripts/llm-task.sh --brainstorm "How should we architect this?"

# Check background task status
~/.claude/skills/llm-manager/scripts/llm-task.sh --status
```

## Daemon Mode (Autonomous)

For long-running autonomous operation:

```bash
# Start the daemon (processes queue continuously)
~/.claude/skills/llm-manager/scripts/llm-daemon.sh start

# Add tasks to queue
~/.claude/skills/llm-manager/scripts/llm-daemon.sh add "implement feature X"
~/.claude/skills/llm-manager/scripts/llm-daemon.sh add-file tasks.txt

# Check status
~/.claude/skills/llm-manager/scripts/llm-daemon.sh status

# Wait for all tasks to complete
~/.claude/skills/llm-manager/scripts/llm-daemon.sh wait

# Get specific task result
~/.claude/skills/llm-manager/scripts/llm-daemon.sh result <task_id>

# View logs
~/.claude/skills/llm-manager/scripts/llm-daemon.sh logs

# Stop daemon
~/.claude/skills/llm-manager/scripts/llm-daemon.sh stop
```

Features:
- **Task Queue**: Add tasks to queue, daemon processes continuously
- **Smart Routing**: Picks best backend per task (no bias)
- **Parallel Workers**: Unlimited concurrent tasks (all agents support parallel)
- **Auto-retry**: 3 retries per backend before failover
- **Failover**: Tries all available backends
- **Watchdog**: 5-minute timeout per task
- **Notifications**: macOS notifications + completion log

```bash
# Start with limited workers if needed
~/.claude/skills/llm-manager/scripts/llm-daemon.sh start --workers 8
```

## Remember

- All backends are peers - no bias in selection
- Smart routing picks best backend for each task
- Tasks run in background by default (-F for foreground)
- The task ends when verified complete
- Use daemon mode for autonomous hours-long operation
