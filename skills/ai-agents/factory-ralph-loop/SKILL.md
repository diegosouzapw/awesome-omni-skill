---
name: factory-ralph-loop
description: Iterative task execution using the Ralph Loop pattern (named after Ralph Wiggum). Use when you need to repeatedly run an agent until a condition is met—fixing all lint errors, passing all tests, or exhausting PRD tasks. The filesystem serves as memory between iterations.
---

# Ralph Loop Pattern

The Ralph Loop (named after Ralph Wiggum) is an agentic pattern where you run an AI agent in a continuous loop until a task is complete. Each iteration starts relatively fresh, with the filesystem serving as persistent memory.

## Core Characteristics

1. **Same systemPrompt repeated** — The agent receives consistent instructions each iteration
2. **Filesystem as memory** — Code changes persist on disk between iterations
3. **Fresh context** — Each iteration reduces context pollution vs. single long conversation
4. **Exit condition** — Loop ends when tests pass, lint is clean, or work is exhausted
5. **Simple orchestrator** — Just `while (!done) { run agent }`

## Basic Structure

```typescript
const maxIterations = 10;
let iteration = 0;
let done = false;

while (!done && iteration < maxIterations) {
  iteration++;
  factory.observe.log("info", `Iteration ${iteration}`, { maxIterations });

  const result = await factory.spawn({
    agent: "worker",
    systemPrompt: "You are fixing issues iteratively",
    prompt: "Fix the next issue",
    model: "anthropic/claude-sonnet-4-6",
    step: iteration,
  });

  // Check exit condition
  done = result.exitCode === 0 && result.text.includes("all clean");

  if (result.exitCode !== 0) {
    factory.observe.log("error", "Agent failed", { iteration, error: result.errorMessage });
    break;
  }
}
```

## Pattern 1: Fix All Lint Errors

Repeatedly run an agent until lint is clean:

```typescript
import { spawnSync } from "node:child_process";

const maxIterations = 20;
let iteration = 0;

while (iteration < maxIterations) {
  iteration++;

  const lintResult = spawnSync("npm", ["run", "lint"], {
    cwd: process.cwd(),
    encoding: "utf-8",
  });

  if (lintResult.status === 0) {
    factory.observe.log("info", "Lint clean!", { iterations: iteration });
    break;
  }

  factory.observe.log("info", `Iteration ${iteration}`, {
    exitCode: lintResult.status,
    errorCount: (lintResult.stdout.match(/error/gi) || []).length,
  });

  const result = await factory.spawn({
    agent: "linter",
    systemPrompt: `You fix lint errors iteratively.
Run 'npm run lint' to see current errors.
Fix one or more errors, focusing on the most common patterns.
Make minimal, focused changes.`,
    prompt: `Fix lint errors. Current output:\n\n${lintResult.stdout}\n${lintResult.stderr}`,
    model: "mistral/devstral-2512",
    step: iteration,
  });

  if (result.exitCode !== 0) {
    factory.observe.log("error", "Agent failed", { iteration });
    break;
  }
}
```

## Pattern 2: With Progress Tracking

Accumulate state across iterations to show progress:

```typescript
import { spawnSync } from "node:child_process";

interface ProgressState {
  fixedIssues: string[];
  lastErrorCount: number;
  stagnantIterations: number;
}

const maxIterations = 20;
let iteration = 0;

const progress: ProgressState = {
  fixedIssues: [],
  lastErrorCount: Infinity,
  stagnantIterations: 0,
};

while (iteration < maxIterations) {
  iteration++;

  const lintResult = spawnSync("npm", ["run", "lint"], {
    cwd: process.cwd(),
    encoding: "utf-8",
  });

  const errorCount = (lintResult.stdout.match(/error/gi) || []).length;

  if (lintResult.status === 0) {
    factory.observe.log("info", "All issues fixed!", {
      iterations: iteration,
      fixedIssues: progress.fixedIssues,
    });
    break;
  }

  // Track progress
  if (errorCount >= progress.lastErrorCount) {
    progress.stagnantIterations++;
  } else {
    progress.stagnantIterations = 0;
  }

  // Exit if stagnant
  if (progress.stagnantIterations >= 3) {
    factory.observe.log("warning", "No progress for 3 iterations", { errorCount });
    break;
  }

  factory.observe.log("info", `Iteration ${iteration}`, {
    errorCount,
    lastErrorCount: progress.lastErrorCount,
    fixed: progress.fixedIssues.length,
  });

  progress.lastErrorCount = errorCount;

  const result = await factory.spawn({
    agent: "fixer",
    systemPrompt: `You fix lint errors iteratively.
Track your progress and avoid repeating unsuccessful approaches.
Previous fixes: ${progress.fixedIssues.join(", ") || "none yet"}
Error count: ${errorCount} (was ${progress.lastErrorCount === Infinity ? "unknown" : progress.lastErrorCount})`,
    prompt: `Fix lint errors:\n\n${lintResult.stdout}\n${lintResult.stderr}`,
    model: "anthropic/claude-sonnet-4-6",
    step: iteration,
  });

  if (result.exitCode === 0) {
    const fixMatch = result.text.match(/fixed?:?\s*(.+)/i);
    if (fixMatch) {
      progress.fixedIssues.push(fixMatch[1]);
    }
  }
}
```

## Pattern 3: Loop Until Tests Pass

Run agent repeatedly until test suite passes:

```typescript
import { spawnSync } from "node:child_process";

const testCommand = "npm test";
const [cmd, ...args] = testCommand.split(" ");
const maxIterations = 10;
let iteration = 0;

while (iteration < maxIterations) {
  iteration++;

  const testResult = spawnSync(cmd, args, {
    cwd: process.cwd(),
    encoding: "utf-8",
    timeout: 60000,
  });

  if (testResult.status === 0) {
    factory.observe.log("info", "Tests passing!", { iterations: iteration });
    break;
  }

  factory.observe.log("info", `Iteration ${iteration}`, {
    exitCode: testResult.status,
    timeout: testResult.signal === "SIGTERM",
  });

  const failureOutput = [testResult.stdout, testResult.stderr]
    .filter(Boolean)
    .join("\n")
    .slice(-5000); // Last 5KB to avoid huge prompt payloads

  const result = await factory.spawn({
    agent: "test-fixer",
    systemPrompt: `You fix failing tests iteratively.
Analyze test output, identify the root cause, and make minimal fixes.
Run the tests again to verify your changes.
Focus on one failure at a time if there are multiple.`,
    prompt: `Fix failing tests. Output from '${testCommand}':\n\n${failureOutput}`,
    model: "anthropic/claude-opus-4-6",
    step: iteration,
  });

  if (result.exitCode !== 0) {
    factory.observe.log("error", "Agent failed", { iteration });
    break;
  }
}
```

## Pattern 4: Exhaustive PRD Implementation

Work through Product Requirements Document tasks until all are complete:

```typescript
import fs from "node:fs";

interface PRDTask {
  id: string;
  description: string;
  completed: boolean;
}

const prdPath = "./PRD.md";
const tasksPath = "./tasks.json";
const maxIterations = 50;

// Load or initialize tasks
let tasks: PRDTask[];
if (fs.existsSync(tasksPath)) {
  tasks = JSON.parse(fs.readFileSync(tasksPath, "utf-8"));
} else {
  const prdContent = fs.readFileSync(prdPath, "utf-8");
  tasks = parsePRD(prdContent);
  fs.writeFileSync(tasksPath, JSON.stringify(tasks, null, 2));
}

let iteration = 0;

while (iteration < maxIterations) {
  const nextTask = tasks.find(t => !t.completed);
  if (!nextTask) {
    factory.observe.log("info", "All tasks completed!", { iterations: iteration });
    break;
  }

  iteration++;
  factory.observe.log("info", `Iteration ${iteration}: ${nextTask.id}`, {
    remaining: tasks.filter(t => !t.completed).length,
  });

  const result = await factory.spawn({
    agent: "implementer",
    systemPrompt: `You implement PRD tasks iteratively.
Read the PRD at ${prdPath}.
Complete tasks one at a time.
Mark tasks complete by updating ${tasksPath}.`,
    prompt: `Implement: ${nextTask.id} - ${nextTask.description}\n\nCompleted so far:\n${
      tasks.filter(t => t.completed).map(t => `+ ${t.id}`).join("\n")
    }`,
    model: "openai-codex/gpt-5.3-codex",
    step: iteration,
  });

  if (result.exitCode !== 0) {
    factory.observe.log("error", "Agent failed", { iteration, task: nextTask.id });
    break;
  }

  // Reload tasks (agent may have updated them)
  if (fs.existsSync(tasksPath)) {
    tasks = JSON.parse(fs.readFileSync(tasksPath, "utf-8"));
  }
}

function parsePRD(content: string): PRDTask[] {
  const matches = content.matchAll(/^[-*]\s*\[\s*\]\s*(.+)$/gm);
  const tasks: PRDTask[] = [];
  let id = 1;

  for (const match of matches) {
    tasks.push({
      id: `TASK-${id++}`,
      description: match[1].trim(),
      completed: false,
    });
  }

  return tasks;
}
```

## Pattern 5: Combined Safety Checks

Comprehensive safety and exit logic:

```typescript
import { spawnSync } from "node:child_process";

const maxIterations = 20;
const maxStagnantIterations = 3;
const maxFailedIterations = 2;
const checkCommand = "npm run lint";

let iteration = 0;
let stagnantCount = 0;
let failedCount = 0;
let lastCheckOutput = "";

while (iteration < maxIterations) {
  iteration++;

  // Periodic check
  const [cmd, ...args] = checkCommand.split(" ");
  const checkResult = spawnSync(cmd, args, {
    cwd: process.cwd(),
    encoding: "utf-8",
  });

  if (checkResult.status === 0) {
    factory.observe.log("info", "Check passed!", { iterations: iteration });
    break;
  }

  // Track stagnation
  const currentOutput = checkResult.stdout + checkResult.stderr;
  if (currentOutput === lastCheckOutput) {
    stagnantCount++;
    factory.observe.log("warning", "No change detected", { stagnantCount });
  } else {
    stagnantCount = 0;
  }
  lastCheckOutput = currentOutput;

  if (stagnantCount >= maxStagnantIterations) {
    factory.observe.log("error", "Stagnant iterations exceeded", { stagnantCount });
    break;
  }

  factory.observe.log("info", `Iteration ${iteration}`, {
    stagnantCount,
    failedCount,
    max: maxIterations,
  });

  const result = await factory.spawn({
    agent: "worker",
    systemPrompt: "You are fixing issues iteratively",
    prompt: "Continue fixing issues",
    model: "anthropic/claude-sonnet-4-6",
    step: iteration,
  });

  if (result.exitCode !== 0) {
    failedCount++;
    factory.observe.log("error", "Agent failed", { iteration, failedCount });

    if (failedCount >= maxFailedIterations) {
      factory.observe.log("error", "Failed iterations exceeded", { failedCount });
      break;
    }
  } else {
    failedCount = 0;
  }
}
```

## Best Practices

### 1. **Set max iterations**

Always have an upper bound to prevent infinite loops:

```typescript
const maxIterations = 20; // Sensible default
```

### 2. **Detect stagnation**

Track if the agent is making progress:

```typescript
if (currentState === lastState) {
  stagnantCount++;
  if (stagnantCount >= 3) break;
}
```

### 3. **Use bash exit conditions**

Shell out to authoritative checks (tests, lint, build):

```typescript
const result = spawnSync("npm", ["test"], { encoding: "utf-8" });
if (result.status === 0) break;
```

### 4. **Provide context to agent**

Include iteration number, progress, previous attempts:

```typescript
prompt: `Iteration ${iteration}/${maxIterations}
Fixed so far: ${fixed.join(", ")}
Current errors: ${errorCount}
...`
```

### 5. **Log everything**

Observability is critical for debugging loops:

```typescript
factory.observe.log("info", "Loop state", {
  iteration,
  errorCount,
  stagnantCount,
  lastChange
});
```

### 6. **Limit context size**

Truncate large outputs to avoid prompt bloat:

```typescript
const recentOutput = fullOutput.slice(-5000); // Last 5KB
```

### 7. **Allow early exit**

If the goal is achieved, return immediately:

```typescript
if (testsPassing) break;
```

## When to Use Ralph Loop

Good for:
- Fixing lint/type errors iteratively
- Making tests pass one by one
- Implementing PRD tasks sequentially
- Refactoring with incremental validation
- Code generation with iterative refinement

Not ideal for:
- Tasks requiring deep context across iterations
- Complex multi-step reasoning within a single problem
- When the agent needs to remember detailed discussions
- Parallel work (use `Promise.all` with `factory.spawn` instead)

## Advanced: Nested Loops

You can nest Ralph Loops for hierarchical work:

```typescript
const modules = ["src/auth", "src/api", "src/db"];

for (const module of modules) {
  factory.observe.log("info", `Processing module: ${module}`);

  let iteration = 0;
  while (iteration < 10) {
    iteration++;

    const result = await factory.spawn({
      agent: "module-fixer",
      systemPrompt: `Fix issues in ${module}`,
      prompt: "Run checks and fix issues",
      model: "mistral/devstral-2512",
      step: iteration,
    });

    const check = spawnSync("npm", ["run", "lint", module], {
      cwd: process.cwd(),
      encoding: "utf-8",
    });

    if (check.status === 0) break;
  }
}
```

## Summary

The Ralph Loop is a simple but powerful pattern:
- **While loop** around `await factory.spawn()`
- **Filesystem persistence** between iterations
- **Bash exit conditions** for authoritative checks
- **Progress tracking** to detect stagnation
- **Max iterations** for safety

It works because the agent sees fresh context each iteration, making progress incrementally while the filesystem accumulates changes. Perfect for iterative tasks where "run it again" is a valid strategy.
