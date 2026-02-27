---
name: debug
version: 0.1.0
description: Troubleshoots Outfitter Stack issues including Result handling, MCP problems, CLI output, exit codes, and logging. Use when debugging stack-specific issues, unexpected errors, wrong output modes, or when "debug Result", "MCP not working", "wrong exit code", or "logging issue" are mentioned.
allowed-tools: Read Grep Glob Bash(rg *) Bash(bun *)
---

# Stack Debugging

Troubleshoot @outfitter/\* package issues.

## Result Issues

### Always Getting Error

**Symptom:** Result is `err` when it should be `ok`.

**Check validation:**

```typescript
const inputResult = validateInput(rawInput);
if (inputResult.isErr()) {
  console.log("Validation failed:", inputResult.error.context);
  return inputResult;
}
```

**Check async:**

```typescript
// BAD: Missing await
const result = getUser(id); // Promise, not Result!

// GOOD
const result = await getUser(id);
```

### Type Narrowing Broken

**Symptom:** TypeScript doesn't know type after `isOk()`.

```typescript
// BAD: Reassigning breaks narrowing
let result = await getUser(id);
if (result.isOk()) {
  result = await updateUser(result.value); // Breaks!
}

// GOOD: Separate variables
const getResult = await getUser(id);
if (getResult.isErr()) return getResult;
const updateResult = await updateUser(getResult.value);
```

### Error Type Lost

**Use `_tag` for narrowing:**

```typescript
if (result.isErr()) {
  switch (result.error._tag) {
    case "ValidationError":
      console.log(result.error.context);
      break;
    case "NotFoundError":
      console.log(result.error.resourceId);
      break;
  }
}
```

## MCP Issues

### Tool Not Appearing

1. Register before `start()`:

   ```typescript
   server.registerTool(myTool);
   server.start(); // After registration!
   ```

2. Check schema is valid Zod with `.describe()`:
   ```typescript
   const schema = z.object({
     query: z.string().describe("Required for AI"),
   });
   ```

### Tool Invocation Failing

1. Verify handler is async:

   ```typescript
   handler: async (input) => {
     // Not sync!
     return Result.ok(data);
   };
   ```

2. Return Result, not raw value:

   ```typescript
   // BAD
   return { data: "value" };

   // GOOD
   return Result.ok({ data: "value" });
   ```

## CLI Output Issues

### JSON Not Printing

1. Force mode:

   ```typescript
   await output(data, { mode: "json" });
   ```

2. Check environment:

   ```bash
   OUTFITTER_JSON=1 myapp list
   OUTFITTER_JSON=0 myapp list --json  # Forces human!
   ```

3. Await output:

   ```typescript
   // BAD
   output(data);
   process.exit(0); // May exit before output!

   // GOOD
   await output(data);
   ```

### Wrong Exit Code

1. Use `exitWithError`:

   ```typescript
   // BAD
   process.exit(1);

   // GOOD
   exitWithError(result.error);
   ```

2. Exit code table:

   | Category   | Exit |
   | ---------- | ---- |
   | validation | 1    |
   | not_found  | 2    |
   | conflict   | 3    |
   | permission | 4    |
   | timeout    | 5    |
   | rate_limit | 6    |
   | network    | 7    |
   | internal   | 8    |
   | auth       | 9    |
   | cancelled  | 130  |

## Logging Issues

### Redaction Not Working

```typescript
const logger = createLogger({
  redaction: { enabled: true }, // Must be true!
});

// Custom patterns
const logger = createLogger({
  redaction: {
    enabled: true,
    patterns: ["password", "apiKey", "myCustomSecret"],
  },
});
```

### Missing Context

```typescript
import { createChildLogger } from "@outfitter/logging";

const requestLogger = createChildLogger(ctx.logger, {
  requestId: ctx.requestId,
  handler: "myHandler",
});

requestLogger.info("Processing", { data }); // Includes requestId
```

### Wrong Level

```typescript
const logger = createLogger({
  level: process.env.LOG_LEVEL || "info",
});

// Hierarchy: trace < debug < info < warn < error < fatal
// "info" hides trace and debug
```

## Debugging Tools

### Trace Result Chain

```typescript
function traceResult<T, E>(name: string, result: Result<T, E>): Result<T, E> {
  console.log(
    `[${name}]`,
    result.isOk() ? "OK:" : "ERR:",
    result.isOk() ? result.value : result.error
  );
  return result;
}

const result = traceResult("getUser", await getUser(id));
```

### Inspect Context

```typescript
console.log("Context:", {
  requestId: ctx.requestId,
  hasLogger: !!ctx.logger,
  hasConfig: !!ctx.config,
  hasSignal: !!ctx.signal,
  cwd: ctx.cwd,
});
```

### Validate Zod Schema

```typescript
const parseResult = schema.safeParse(rawInput);
if (!parseResult.success) {
  console.log("Zod errors:", parseResult.error.issues);
}
```

## Related Skills

- `stack:patterns` — Correct patterns
- `stack:review` — Systematic audit
- `stack:outfitter-testing` — Prevent regressions
