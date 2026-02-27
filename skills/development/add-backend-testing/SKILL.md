---
name: add-backend-testing
description: 'Add backend integration testing with Vitest to an existing app. Sets up isolated test database schema and writes tests for tRPC routers.'
---

# Add Backend Testing

**Goal:** Set up backend integration testing with Vitest using an isolated test database schema.

---

## Task 1: Gather Information

Before starting, you need:
- `service_id`: The Timescale Cloud service ID for the database

If not provided, check the `DATABASE_URL` in `.env` to find it, or use `service_list` MCP tool.

---

## Task 2: Set Up Test Infrastructure

1. Use the `setup_testing` MCP tool:
   ```
   setup_testing(application_directory: ".", service_id: "<service_id>")
   ```

2. Install Vitest:
   ```bash
   npm install -D vitest dotenv
   ```

3. Add test scripts to package.json:
   ```json
   {
     "scripts": {
       "test": "vitest run",
       "test:watch": "vitest"
     }
   }
   ```

4. Write integration tests for each tRPC router using this pattern:
   ```typescript
   import { describe, it, expect } from "vitest";
   import { appRouter } from "~/server/api/root";
   import { createCallerFactory } from "~/server/api/trpc";
   import { db } from "~/server/db";

   const createCaller = createCallerFactory(appRouter);
   const caller = createCaller({ session: null, db, headers: new Headers() });

   describe("exampleRouter", () => {
     it("returns data", async () => {
       const result = await caller.example.getAll();
       expect(result).toBeDefined();
     });
   });
   ```

5. Run `npm test` and ensure all tests pass before completing

---

## Task 3: Update CLAUDE.md

Add a Testing section to CLAUDE.md with the following content:

```markdown
## Testing

This app uses Vitest for backend integration testing with an isolated test database schema.

**Test infrastructure:**
- Tests run against a separate PostgreSQL schema (see `DATABASE_SCHEMA` in `.env.test.local`)
- A dedicated test user has permissions only on the test schema
- Schema is automatically pushed before tests via global setup
- Tests use `.env.test.local` for database configuration (gitignored)

**Writing tests:**
\`\`\`typescript
import { describe, it, expect } from "vitest";
import { appRouter } from "~/server/api/root";
import { createCallerFactory } from "~/server/api/trpc";
import { db } from "~/server/db";

const createCaller = createCallerFactory(appRouter);
const caller = createCaller({ session: null, db, headers: new Headers() });

describe("myRouter", () => {
  it("returns data", async () => {
    const result = await caller.my.getData();
    expect(result).toBeDefined();
  });
});
\`\`\`
```

Also update:
- Add `npm test` and `npm run test:watch` to the Commands section
- Add "3. Add tests in `src/test/routers`" to the "New tRPC Router" checklist
- Update "Before Committing" section to include `npm test`:
  ```bash
  npm test && npm run check
  ```

---

## Task 4: Commit

Ask the user if they want to commit the changes.

---

## Task 5: Offer Further Hardening

Ask the user: "Would you like to add stricter TypeScript checks as well? This catches additional bugs that standard TypeScript misses."

If yes, follow the `add-strict-checks` skill.
