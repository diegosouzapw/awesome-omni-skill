---
name: a2a-executor-patterns
description: Agent-to-Agent (A2A) executor implementation patterns for task handling, execution management, and agent coordination. Use when building A2A executors, implementing task handlers, creating agent execution flows, or when user mentions A2A protocol, task execution, agent executors, task handlers, or agent coordination.
allowed-tools: Read, Write, Bash, Grep, Glob
---

# A2A Executor Patterns

**Purpose:** Provide production-ready executor patterns for implementing Agent-to-Agent (A2A) protocol task handlers with proper error handling, retry logic, and execution flows.

**Activation Triggers:**
- Implementing A2A protocol executors
- Building task handler functions
- Creating agent execution flows
- Managing task lifecycle and state
- Implementing retry and error recovery
- Building executor middleware
- Task validation and sanitization

**Key Resources:**
- `templates/basic-executor.ts` - Simple synchronous executor
- `templates/basic-executor.py` - Python synchronous executor
- `templates/async-executor.ts` - Asynchronous task executor
- `templates/async-executor.py` - Python async executor
- `templates/streaming-executor.ts` - Streaming result executor
- `templates/streaming-executor.py` - Python streaming executor
- `templates/batch-executor.ts` - Batch task processing
- `scripts/validate-executor.sh` - Validate executor implementation
- `scripts/test-executor.sh` - Test executor against A2A spec
- `examples/` - Production executor implementations

## Core Executor Patterns

### 1. Basic Executor (Synchronous)

**When to use:** Simple, fast tasks with immediate results

**Template:** `templates/basic-executor.ts` or `templates/basic-executor.py`

**Pattern:**
```typescript
async function executeTask(task: A2ATask): Promise<A2AResult> {
  // 1. Validate input
  validateTask(task)

  // 2. Execute task
  const result = await processTask(task)

  // 3. Return result
  return {
    status: 'completed',
    result,
    taskId: task.id
  }
}
```

**Best for:** Quick operations, validation tasks, simple transformations

### 2. Async Executor (Long-Running)

**When to use:** Tasks that take time and need status updates

**Template:** `templates/async-executor.ts` or `templates/async-executor.py`

**Pattern:**
- Accept task and return task ID immediately
- Process task asynchronously
- Provide status endpoint
- Send completion callback

**Best for:** LLM inference, file processing, data analysis

### 3. Streaming Executor

**When to use:** Results should be delivered incrementally

**Template:** `templates/streaming-executor.ts` or `templates/streaming-executor.py`

**Pattern:**
- Open stream connection
- Send partial results as available
- Close stream on completion
- Handle backpressure

**Best for:** Text generation, real-time data, progressive results

### 4. Batch Executor

**When to use:** Processing multiple related tasks efficiently

**Template:** `templates/batch-executor.ts`

**Pattern:**
- Accept multiple tasks
- Group by similarity
- Process in parallel batches
- Return aggregated results

**Best for:** Bulk operations, parallel processing, resource optimization

## Execution Flow Components

### 1. Task Validation

```typescript
function validateTask(task: A2ATask): void {
  // Validate required fields
  if (!task.id) throw new ValidationError('Task ID required')
  if (!task.type) throw new ValidationError('Task type required')

  // Validate task parameters
  validateParameters(task.parameters)

  // Check executor capabilities
  if (!supportsTaskType(task.type)) {
    throw new UnsupportedTaskError(task.type)
  }
}
```

**Purpose:** Catch errors early, provide clear feedback

### 2. Error Handling

```typescript
async function executeWithErrorHandling(task: A2ATask) {
  try {
    return await executeTask(task)
  } catch (error) {
    if (error instanceof ValidationError) {
      return { status: 'failed', error: error.message }
    }
    if (error instanceof RetryableError) {
      return scheduleRetry(task)
    }
    // Log and return generic error
    logger.error('Task execution failed', { taskId: task.id, error })
    return { status: 'failed', error: 'Internal error' }
  }
}
```

**Error Types:**
- `ValidationError` - Invalid input, don't retry
- `RetryableError` - Temporary failure, safe to retry
- `FatalError` - Permanent failure, abort

### 3. Retry Logic

```typescript
const retryConfig = {
  maxAttempts: 3,
  backoff: 'exponential', // or 'linear', 'fixed'
  initialDelay: 1000, // ms
  maxDelay: 30000
}

async function executeWithRetry(
  task: A2ATask,
  attempt: number = 1
): Promise<A2AResult> {
  try {
    return await executeTask(task)
  } catch (error) {
    if (attempt >= retryConfig.maxAttempts) {
      throw new MaxRetriesExceededError(task.id)
    }

    if (error instanceof RetryableError) {
      const delay = calculateBackoff(attempt)
      await sleep(delay)
      return executeWithRetry(task, attempt + 1)
    }

    throw error
  }
}
```

**Retry Strategies:**
- Exponential backoff: `delay = initialDelay * (2 ^ attempt)`
- Linear backoff: `delay = initialDelay * attempt`
- Fixed delay: `delay = initialDelay`

### 4. Task State Management

```typescript
interface TaskState {
  id: string
  status: 'pending' | 'running' | 'completed' | 'failed'
  result?: any
  error?: string
  startTime: Date
  endTime?: Date
  attempts: number
}

class TaskStore {
  private tasks = new Map<string, TaskState>()

  createTask(id: string): TaskState {
    const state: TaskState = {
      id,
      status: 'pending',
      startTime: new Date(),
      attempts: 0
    }
    this.tasks.set(id, state)
    return state
  }

  updateTask(id: string, update: Partial<TaskState>): void {
    const state = this.tasks.get(id)
    if (state) {
      Object.assign(state, update)
    }
  }

  getTask(id: string): TaskState | undefined {
    return this.tasks.get(id)
  }
}
```

## Executor Middleware

### 1. Logging Middleware

```typescript
function loggingMiddleware(
  executor: Executor
): Executor {
  return async (task) => {
    logger.info('Task started', { taskId: task.id })
    const start = Date.now()

    try {
      const result = await executor(task)
      const duration = Date.now() - start
      logger.info('Task completed', { taskId: task.id, duration })
      return result
    } catch (error) {
      const duration = Date.now() - start
      logger.error('Task failed', { taskId: task.id, duration, error })
      throw error
    }
  }
}
```

### 2. Metrics Middleware

```typescript
function metricsMiddleware(
  executor: Executor
): Executor {
  return async (task) => {
    metrics.increment('tasks.started', { type: task.type })
    const start = Date.now()

    try {
      const result = await executor(task)
      const duration = Date.now() - start
      metrics.timing('tasks.duration', duration, { type: task.type })
      metrics.increment('tasks.completed', { type: task.type })
      return result
    } catch (error) {
      metrics.increment('tasks.failed', { type: task.type })
      throw error
    }
  }
}
```

### 3. Rate Limiting Middleware

```typescript
function rateLimitMiddleware(
  executor: Executor,
  limit: { requests: number, window: number }
): Executor {
  const limiter = new RateLimiter(limit.requests, limit.window)

  return async (task) => {
    await limiter.acquire()
    try {
      return await executor(task)
    } finally {
      limiter.release()
    }
  }
}
```

## Production Best Practices

### 1. Timeouts

```typescript
async function executeWithTimeout(
  task: A2ATask,
  timeoutMs: number
): Promise<A2AResult> {
  return Promise.race([
    executeTask(task),
    new Promise((_, reject) =>
      setTimeout(() => reject(new TimeoutError()), timeoutMs)
    )
  ])
}
```

### 2. Resource Cleanup

```typescript
async function executeWithCleanup(task: A2ATask) {
  const resources = []

  try {
    const resource = await allocateResource()
    resources.push(resource)

    return await executeTask(task, resource)
  } finally {
    // Always cleanup, even on error
    await Promise.all(
      resources.map(r => r.cleanup())
    )
  }
}
```

### 3. Graceful Shutdown

```typescript
class GracefulExecutor {
  private activeTasks = new Set<string>()
  private shuttingDown = false

  async execute(task: A2ATask): Promise<A2AResult> {
    if (this.shuttingDown) {
      throw new Error('Executor is shutting down')
    }

    this.activeTasks.add(task.id)

    try {
      return await executeTask(task)
    } finally {
      this.activeTasks.delete(task.id)
    }
  }

  async shutdown(): Promise<void> {
    this.shuttingDown = true

    // Wait for active tasks to complete
    while (this.activeTasks.size > 0) {
      await sleep(100)
    }
  }
}
```

## Common Executor Types

### 1. LLM Executor

**Example:** `examples/llm-executor.ts`

Executes LLM inference tasks with streaming

### 2. Function Executor

**Example:** `examples/function-executor.ts`

Calls functions/tools and returns results

### 3. Workflow Executor

**Example:** `examples/workflow-executor.ts`

Orchestrates multi-step workflows

### 4. Validation Executor

**Example:** `examples/validation-executor.ts`

Validates data and returns compliance results

## Validation and Testing

**Scripts:**
- `scripts/validate-executor.sh` - Validate executor structure
- `scripts/test-executor.sh` - Test against A2A spec

**Run validation:**
```bash
bash scripts/validate-executor.sh your-executor.ts
```

**Run tests:**
```bash
bash scripts/test-executor.sh your-executor.ts
```

## Resources

**TypeScript Templates:**
- `basic-executor.ts` - Simple sync executor
- `async-executor.ts` - Async with status tracking
- `streaming-executor.ts` - Streaming results
- `batch-executor.ts` - Batch processing

**Python Templates:**
- `basic-executor.py` - Simple sync executor
- `async-executor.py` - Async with status tracking
- `streaming-executor.py` - Streaming results

**Scripts:**
- `validate-executor.sh` - Structure validation
- `test-executor.sh` - A2A spec compliance

**Examples:**
- `llm-executor.ts` - LLM inference executor
- `function-executor.ts` - Function calling executor
- `workflow-executor.ts` - Multi-step workflows
- `validation-executor.ts` - Data validation

---

**Protocol Version:** A2A Protocol v1.0
**Runtime:** Node.js 18+, Python 3.9+

**Best Practice:** Start with basic executor, add complexity (async, streaming, batching) only as needed
