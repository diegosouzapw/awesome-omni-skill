---
name: vercel-workflow-sdk
description: write code that uses https://useworkflow.dev/ on Vercel
---

# Vercel Workflow SDK

This skill documents how to use the Vercel Workflow SDK (`workflow` package) for building durable, long-running workflows.

## Core Concepts

### Workflows and Steps

**Workflows** (`"use workflow"` directive) are durable functions that orchestrate steps. They:
- Run in a sandboxed environment
- Must be **deterministic** - same inputs always produce same outputs
- Survive serverless timeouts and can resume after failures
- Should NOT perform side effects directly

**Steps** (`"use step"` directive) perform actual work. They:
- Have full Node.js/npm access
- Results are automatically persisted for replay
- Default to 3 retries (4 total attempts)
- Are where side effects (DB writes, API calls) should happen

```typescript
export async function myWorkflow(input: string) {
  "use workflow";

  const result = await myStep(input);  // Step call
  return result;
}

async function myStep(input: string) {
  "use step";

  // Perform actual work here
  return await db.doSomething(input);
}
```

### Key Constraint: Pass-by-Value

Parameters between workflows and steps are **passed by value, not reference**. Mutations inside steps don't propagate back - always return modified data.

## Imports

```typescript
import {
  getWorkflowMetadata,   // Workflow-level context
  getStepMetadata,       // Step-level context (stepId, attempt)
  sleep,                 // Durable sleep
  fetch,                 // Workflow-aware fetch with retries
  FatalError,            // Stop retries immediately
  RetryableError,        // Retry with custom delay
  createWebhook,         // Pause for external HTTP
} from "workflow";

import { start } from "workflow/api";  // Start workflows
```

## Starting Workflows

```typescript
import { start } from "workflow/api";

// Start and get Run object for tracking
const run = await start(myWorkflow, [arg1, arg2]);
console.log(run.runId);  // Unique run identifier

// For workflows with no arguments
const run = await start(myWorkflow);
```

The `Run` object provides:
- `runId` - Unique identifier
- `status` - Async getter for current status
- `returnValue` - Async, blocks until completion
- `readable` - ReadableStream for updates

## Metadata Functions

### getWorkflowMetadata()

Use inside workflow functions for workflow-level context:

```typescript
export async function myWorkflow() {
  "use workflow";

  const { workflowRunId } = getWorkflowMetadata();
  log.info("Starting workflow", { workflowRunId });
}
```

### getStepMetadata()

Use inside step functions for step-level context:

```typescript
async function myStep() {
  "use step";

  const { stepId, attempt } = getStepMetadata();

  // stepId: Stable across retries, unique per step invocation
  // attempt: Current retry attempt number
}
```

## Error Handling

### Default Behavior

Steps automatically retry up to 3 times on errors. Configure with:

```typescript
async function myStep() {
  "use step";
  // ...
}
myStep.maxRetries = 5;  // Set custom retry limit
myStep.maxRetries = 0;  // Disable retries
```

### RetryableError

Signal that a step should retry with a specific delay:

```typescript
import { RetryableError } from "workflow";

async function myStep() {
  "use step";

  const result = await someOperation();

  if (result.needsRetry) {
    throw new RetryableError("Temporary failure", {
      retryAfter: 5000,           // milliseconds
      // OR retryAfter: "5m",     // duration string
      // OR retryAfter: new Date() // specific time
    });
  }
}
```

### FatalError

Signal that a step should NOT retry - permanent failure:

```typescript
import { FatalError } from "workflow";

async function myStep() {
  "use step";

  const item = await db.findItem(id);

  if (!item) {
    throw new FatalError("Item not found");  // Don't retry
  }
}
```

**When to use FatalError:**
- Resource not found (404)
- Invalid input/state
- Permanent external failures
- Business logic rejections

## Idempotency

**Critical**: When steps make external API calls with side effects, use `stepId` as an idempotency key:

```typescript
async function chargeCustomer(amount: number) {
  "use step";

  const { stepId } = getStepMetadata();

  // stepId is stable across retries - prevents duplicate charges
  await stripe.charges.create({
    amount,
    idempotencyKey: `charge-${stepId}`,
  });
}
```

## Control Flow Patterns

### Sequential

```typescript
const a = await stepA();
const b = await stepB(a);  // Depends on a
const c = await stepC(b);  // Depends on b
```

### Parallel

```typescript
// Independent steps run in parallel
const [a, b, c] = await Promise.all([
  stepA(),
  stepB(),
  stepC(),
]);
```

### Fan-out with Concurrency Limit

```typescript
import pLimit from "p-limit";

const limit = pLimit(5);  // Max 5 concurrent

const results = await Promise.all(
  items.map(item =>
    limit(() => processItem(item))
  )
);
```

### Handling Errors in Parallel Processing

When processing multiple items where one failure shouldn't stop others:

```typescript
const results = await Promise.all(
  items.map(item =>
    limit(async () => {
      try {
        return await processItem(item);
      } catch (error) {
        if (error instanceof FatalError) {
          return { id: item.id, success: false, error: error.message };
        }
        throw error;  // Unexpected errors propagate
      }
    })
  )
);
```

## Sleep

Durable sleep that doesn't consume resources:

```typescript
import { sleep } from "workflow";

// Duration strings
await sleep("30 seconds");
await sleep("5 minutes");
await sleep("1 hour");

// Date object
await sleep(new Date(Date.now() + 60000));
```

## Webhooks

Pause workflow until external HTTP request:

```typescript
import { createWebhook } from "workflow";

export async function orderWorkflow(orderId: string) {
  "use workflow";

  const webhook = createWebhook();

  // Store webhook.url for external system to call
  await saveWebhookUrl(orderId, webhook.url);

  // Workflow suspends here until webhook receives request
  const request = await webhook;

  return processWebhookData(request);
}
```

## Best Practices

### 1. Keep Workflows Deterministic

```typescript
// BAD - non-deterministic in workflow
export async function badWorkflow() {
  "use workflow";
  const random = Math.random();  // Different on replay!
}

// GOOD - move non-determinism to steps
export async function goodWorkflow() {
  "use workflow";
  const random = await generateRandom();  // Step handles it
}
```

Note: The SDK fixes `Math.random()` and `Date` during replay, but explicit determinism is preferred.

### 2. Use stepId for External API Idempotency

Always pass `stepId` when making API calls that have side effects.

### 3. Return Data from Steps

Don't mutate objects passed to steps - return modified data:

```typescript
// BAD
async function badStep(data: Data) {
  "use step";
  data.processed = true;  // Won't propagate!
}

// GOOD
async function goodStep(data: Data) {
  "use step";
  return { ...data, processed: true };
}
```

### 4. Use FatalError for Permanent Failures

Don't waste retries on unrecoverable errors.

### 5. Separate Step Functions

Keep step functions in separate files or clearly organized to avoid bundler issues.

### 6. Use getWorkflowMetadata() for Run IDs

Don't generate custom IDs when the SDK provides them:

```typescript
// Instead of passing ID as parameter
export async function myWorkflow() {
  "use workflow";
  const { workflowRunId } = getWorkflowMetadata();
  // Use workflowRunId for logging/tracking
}
```

## Common Patterns

### Retry with Exponential Backoff

```typescript
async function stepWithBackoff() {
  "use step";

  const { attempt } = getStepMetadata();

  const result = await tryOperation();

  if (result.needsRetry) {
    // Exponential backoff: 5s, 10s, 20s, 40s...
    const delayMs = Math.min(5000 * Math.pow(2, attempt), 60000);
    throw new RetryableError("Retrying", { retryAfter: delayMs });
  }

  return result;
}
stepWithBackoff.maxRetries = 5;
```

### Deterministic Jitter

When you need jitter for backoff but must stay deterministic:

```typescript
function calculateBackoff(attempt: number): number {
  const baseDelay = Math.min(5 * Math.pow(2, attempt), 60);
  // Deterministic "jitter" based on attempt number
  const jitterPercent = ((attempt * 7) % 11) / 100;
  return baseDelay * (1 + jitterPercent);
}
```

### Graceful Workflow Termination

```typescript
export async function myWorkflow(id: string) {
  "use workflow";

  try {
    return await processWithRetries(id);
  } catch (error) {
    if (error instanceof FatalError) {
      return { success: false, reason: error.message };
    }
    // Retries exhausted
    return { success: false, reason: "max_attempts_reached" };
  }
}
```

## Serialization

Supported types for workflow/step parameters and returns:
- JSON types: string, number, boolean, null, arrays, objects
- Date, URL, RegExp, BigInt
- Map, Set
- ArrayBuffer, typed arrays (Uint8Array, etc.)
- Headers, Request, Response

## References

- Documentation: https://useworkflow.dev/docs
- API Reference: https://useworkflow.dev/docs/api-reference
