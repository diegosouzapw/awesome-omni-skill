---
name: preferences-cloudflare-wrangler-reference
description: Cloudflare wrangler comprehensive reference for Workers, D1, R2, and KV configuration. Load when working with Cloudflare deployment or wrangler.toml.
---

# Cloudflare wrangler comprehensive reference

Comprehensive configuration reference for Cloudflare Wrangler synthesized from production patterns.

See @~/.claude/skills/preferences-web-application-deployment/SKILL.md for deployment workflows and practical patterns.

## Configuration schema

```jsonc
{
  // JSON schema for IDE autocomplete and validation
  "$schema": "node_modules/wrangler/config-schema.json"
}
```

Enable autocomplete and validation in your IDE by including the schema reference at the top of wrangler.jsonc.

## Core configuration

### Worker identification

```jsonc
{
  // Worker name - used as deployment identifier
  "name": "my-cloudflare-worker",

  // Entry point for the worker
  // Can be .ts, .js, or .mjs - build tools handle transpilation
  "main": "src/index.ts"
}
```

### Runtime compatibility

```jsonc
{
  // Compatibility date - locks API behavior to specific date
  // Use recent date to get latest features
  "compatibility_date": "2025-04-10",

  // Compatibility flags - enable specific runtime features
  "compatibility_flags": [
    "nodejs_compat" // Enable Node.js compatibility layer
  ]
}
```

**Important**: Update compatibility_date regularly to access new platform features.

### Workers.dev subdomain

```jsonc
{
  // Enable workers.dev subdomain for testing
  // Set to false in production to avoid accidental exposure
  "workers_dev": true
}
```

## Static assets configuration

Serve static assets alongside worker logic (SPAs, SSR apps).

```jsonc
{
  "assets": {
    // Directory containing static assets to serve
    "directory": ".output/public",

    // Binding name to access assets in worker code
    "binding": "ASSETS",

    // Handle 404s as SPA - serve index.html for missing routes
    "not_found_handling": "single-page-application",

    // Patterns where worker runs before checking for static assets
    // Useful for API routes, auth endpoints, or dynamic paths
    "run_worker_first": ["/api/*", "/trpc/*", "/auth/*"]
  }
}
```

**Pattern**: TanStack Start or similar SSR frameworks serve frontend assets while worker handles API routes.

## Build configuration

```jsonc
{
  "build": {
    // Command to run before deployment
    "command": "bun run build",

    // Working directory for build command
    "cwd": ".",

    // Directories to watch during `wrangler dev`
    "watch_dir": "src"
  }
}
```

**Alternative**: Use framework-specific build tools (Vite, etc.) and skip wrangler build command.

## Routing configuration

### Custom domain routing

```jsonc
{
  "routes": [
    {
      // Custom domain routing (requires DNS setup in Cloudflare)
      "pattern": "example.com",
      "custom_domain": true
    },
    {
      // Zone-based routing with wildcard pattern
      "zone_name": "example.com",
      "pattern": "api.example.com/*"
    }
  ]
}
```

**Setup**:
1. Add domain to Cloudflare DNS
2. Configure route pattern in wrangler.jsonc
3. Deploy worker - route activates automatically

## Observability

```jsonc
{
  "observability": {
    // Enable metrics, logs, and traces in Cloudflare dashboard
    "enabled": true
  }
}
```

Access observability data: Cloudflare Dashboard > Workers & Pages > [Your Worker] > Metrics

## Smart Placement

Automatically places worker near users for optimal latency.

```jsonc
{
  "placement": {
    "mode": "smart"
  }
}
```

**When to use**: Geographic user distribution with latency-sensitive workloads.

See: https://developers.cloudflare.com/workers/configuration/smart-placement/

## Environment variables

### Non-sensitive configuration (vars)

```jsonc
{
  "vars": {
    "API_URL": "https://api.example.com",
    "FEATURE_FLAG_NEW_UI": "true",
    "LOG_LEVEL": "info"
  }
}
```

**Access in worker**:

```typescript
import { env } from "cloudflare:workers";

const apiUrl = env.API_URL;
const featureEnabled = env.FEATURE_FLAG_NEW_UI === "true";
```

### Sensitive data (secrets)

Secrets not stored in wrangler.jsonc - managed via CLI:

```bash
# Set secret
wrangler secret put DATABASE_PASSWORD

# List secrets
wrangler secret list

# Delete secret
wrangler secret delete OLD_SECRET

# Set secret for specific environment
wrangler secret put API_KEY --env production
```

**Access in worker** (same as vars):

```typescript
import { env } from "cloudflare:workers";

const dbPassword = env.DATABASE_PASSWORD;
```

## Cloudflare bindings

Connect to Cloudflare platform resources.

See: https://developers.cloudflare.com/workers/runtime-apis/bindings/

### D1 databases (serverless SQL)

```jsonc
{
  "d1_databases": [
    {
      "binding": "DB", // Access via env.DB in worker code
      "database_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "database_name": "my-app-db",
      "experimental_remote": true // Allow remote access during local dev
    }
  ]
}
```

**Usage**:

```typescript
import { drizzle } from "drizzle-orm/d1";

const db = drizzle(env.DB);
const users = await db.select().from(usersTable);
```

**Create database**:

```bash
wrangler d1 create my-app-db
```

### KV namespaces (key-value storage)

```jsonc
{
  "kv_namespaces": [
    {
      "binding": "CACHE",
      "id": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
      "experimental_remote": true
    }
  ]
}
```

**Usage**:

```typescript
// Write
await env.CACHE.put("key", "value", {
  expirationTtl: 3600 // 1 hour
});

// Read
const value = await env.CACHE.get("key");

// Read as JSON
const data = await env.CACHE.get("key", "json");

// Delete
await env.CACHE.delete("key");

// List keys
const list = await env.CACHE.list();
```

**Create namespace**:

```bash
wrangler kv namespace create CACHE
```

### R2 buckets (object storage)

```jsonc
{
  "r2_buckets": [
    {
      "binding": "BUCKET",
      "bucket_name": "my-storage-bucket"
    }
  ]
}
```

**Usage** (S3-compatible API):

```typescript
// Upload object
await env.BUCKET.put("uploads/file.pdf", fileData, {
  httpMetadata: {
    contentType: "application/pdf"
  }
});

// Download object
const file = await env.BUCKET.get("uploads/file.pdf");
const blob = await file.blob();

// Delete object
await env.BUCKET.delete("uploads/file.pdf");

// List objects
const list = await env.BUCKET.list({ prefix: "uploads/" });
```

**Create bucket**:

```bash
wrangler r2 bucket create my-storage-bucket
```

### Service bindings (worker-to-worker communication)

```jsonc
{
  "services": [
    {
      "binding": "BACKEND_SERVICE",
      "service": "data-service-production",
      "experimental_remote": true // Enable service-to-service calls in dev
    }
  ]
}
```

**Usage**:

```typescript
// Call another worker service
const response = await env.BACKEND_SERVICE.fetch(
  new Request("https://internal/api/data", {
    method: "POST",
    body: JSON.stringify({ query: "..." })
  })
);

const data = await response.json();
```

**Pattern**: Microservices architecture with multiple workers.

```
apps/
  user-application/     # Frontend worker
  data-service/         # Backend API worker
  auth-service/         # Authentication worker
```

### Browser rendering (headless browser)

```jsonc
{
  "browser": {
    "binding": "VIRTUAL_BROWSER"
  }
}
```

**Usage**:

```typescript
const browser = await env.VIRTUAL_BROWSER.launch();
const page = await browser.newPage();
await page.goto("https://example.com");

// Take screenshot
const screenshot = await page.screenshot();

// Get page content
const html = await page.content();

await browser.close();
```

**Use cases**: Web scraping, PDF generation, automated testing, SEO previews.

### Workers AI (AI model inference)

```jsonc
{
  "ai": {
    "binding": "AI"
  }
}
```

**Usage**:

```typescript
// Text generation
const response = await env.AI.run("@cf/meta/llama-2-7b-chat-int8", {
  prompt: "What is the capital of France?"
});

// Image classification
const result = await env.AI.run("@cf/microsoft/resnet-50", {
  image: imageData
});

// Text embeddings
const embeddings = await env.AI.run("@cf/baai/bge-base-en-v1.5", {
  text: "Hello world"
});
```

See: https://developers.cloudflare.com/workers-ai/

### Queues (async message processing)

```jsonc
{
  "queues": {
    "consumers": [
      {
        "queue": "data-processing-queue",
        "dead_letter_queue": "data-processing-dlq" // Failed messages route here
      },
      {
        "queue": "data-processing-dlq",
        "retry_delay": 0 // Manual retry for DLQ messages
      }
    ],
    "producers": [
      {
        "binding": "QUEUE",
        "queue": "data-processing-queue"
      }
    ]
  }
}
```

**Producer usage**:

```typescript
// Send single message
await env.QUEUE.send({
  userId: "123",
  action: "process_upload"
});

// Batch send
await env.QUEUE.sendBatch([
  { body: { userId: "123" } },
  { body: { userId: "456" } }
]);
```

**Consumer handler**:

```typescript
export default {
  async queue(batch: MessageBatch, env: Env): Promise<void> {
    for (const message of batch.messages) {
      try {
        await processMessage(message.body);
        message.ack();
      } catch (error) {
        message.retry(); // Retry or send to DLQ
      }
    }
  }
}
```

**Create queue**:

```bash
wrangler queues create data-processing-queue
wrangler queues create data-processing-dlq
```

### Workflows (durable execution)

Long-running, durable processes that survive worker restarts.

```jsonc
{
  "workflows": [
    {
      "binding": "MY_WORKFLOW",
      "name": "my-workflow-production",
      "class_name": "MyWorkflow" // Must match exported class in code
    }
  ]
}
```

**Workflow definition**:

```typescript
import { WorkflowEntrypoint, WorkflowStep } from "cloudflare:workers";

export class MyWorkflow extends WorkflowEntrypoint {
  async run(event: any, step: WorkflowStep) {
    // Steps are checkpointed - execution resumes on failure
    const result1 = await step.do("fetch-data", async () => {
      return await fetch("https://api.example.com/data");
    });

    // Sleep preserves state
    await step.sleep("wait-for-processing", "1 hour");

    const result2 = await step.do("process-data", async () => {
      return processData(result1);
    });

    return result2;
  }
}
```

**Trigger workflow**:

```typescript
const instance = await env.MY_WORKFLOW.create({
  params: { userId: "123" }
});

// Check status
const status = await instance.status();
```

### Durable Objects (stateful compute)

Strongly consistent, stateful workers with persistent storage.

```jsonc
{
  "durable_objects": {
    "bindings": [
      {
        "name": "TRACKER_OBJECT",
        "class_name": "EventTracker"
      },
      {
        "name": "SCHEDULER_OBJECT",
        "class_name": "TaskScheduler"
      }
    ]
  }
}
```

**Durable Object migrations**:

Required when adding or modifying Durable Objects.

```jsonc
{
  "migrations": [
    {
      "tag": "v1",
      "new_classes": ["TaskScheduler"] // Standard Durable Object
    },
    {
      "tag": "v2",
      "new_sqlite_classes": ["EventTracker"] // DO with SQLite storage
    }
  ]
}
```

**Durable Object definition**:

```typescript
import { DurableObject } from "cloudflare:workers";

export class EventTracker extends DurableObject {
  async fetch(request: Request) {
    // Strongly consistent storage
    let count = (await this.ctx.storage.get<number>("count")) || 0;
    count++;
    await this.ctx.storage.put("count", count);

    return new Response(count.toString());
  }

  // WebSocket connections
  async webSocketMessage(ws: WebSocket, message: string) {
    // Handle WebSocket messages
  }
}
```

**Access from worker**:

```typescript
// Create unique Durable Object instance
const id = env.TRACKER_OBJECT.idFromName("global-tracker");
const stub = env.TRACKER_OBJECT.get(id);

// Call Durable Object
const response = await stub.fetch(request);
```

**Use cases**: Chat rooms, collaborative editing, counters, rate limiting, session management.

## Environment-specific configuration

Override any top-level setting per environment.

```jsonc
{
  "name": "my-app",

  // Default configuration (used for wrangler dev)
  "vars": {
    "API_URL": "https://api-dev.example.com",
    "STRIPE_PRODUCT_ID": "price_dev_xxxxx"
  },
  "d1_databases": [
    {
      "binding": "DB",
      "database_id": "dev-database-id",
      "experimental_remote": true
    }
  ],

  // Environment-specific overrides
  "env": {
    "stage": {
      "vars": {
        "API_URL": "https://api-stage.example.com",
        "STRIPE_PRODUCT_ID": "price_staging_xxxxx"
      },
      "routes": [
        {
          "custom_domain": true,
          "pattern": "stage.example.com"
        }
      ],
      "d1_databases": [
        {
          "binding": "DB",
          "database_id": "stage-database-id",
          "experimental_remote": true
        }
      ],
      "kv_namespaces": [
        {
          "binding": "CACHE",
          "id": "stage-kv-namespace-id"
        }
      ],
      "r2_buckets": [
        {
          "binding": "BUCKET",
          "bucket_name": "my-storage-stage"
        }
      ],
      "services": [
        {
          "binding": "BACKEND_SERVICE",
          "service": "data-service-stage",
          "experimental_remote": true
        }
      ],
      "queues": {
        "consumers": [
          {
            "queue": "data-queue-stage",
            "dead_letter_queue": "data-dlq-stage"
          }
        ],
        "producers": [
          {
            "binding": "QUEUE",
            "queue": "data-queue-stage"
          }
        ]
      }
    },

    "production": {
      "vars": {
        "API_URL": "https://api.example.com",
        "STRIPE_PRODUCT_ID": "price_production_xxxxx"
      },
      "routes": [
        {
          "custom_domain": true,
          "pattern": "example.com"
        }
      ],
      "d1_databases": [
        {
          "binding": "DB",
          "database_id": "production-database-id"
        }
      ],
      "kv_namespaces": [
        {
          "binding": "CACHE",
          "id": "production-kv-namespace-id"
        }
      ],
      "r2_buckets": [
        {
          "binding": "BUCKET",
          "bucket_name": "my-storage-production"
        }
      ],
      "services": [
        {
          "binding": "BACKEND_SERVICE",
          "service": "data-service-production"
        }
      ],
      "queues": {
        "consumers": [
          {
            "queue": "data-queue-production",
            "dead_letter_queue": "data-dlq-production"
          }
        ],
        "producers": [
          {
            "binding": "QUEUE",
            "queue": "data-queue-production"
          }
        ]
      }
    }
  }
}
```

**Deploy to environment**:

```bash
# Deploy to staging
wrangler deploy --env stage

# Deploy to production
wrangler deploy --env production

# Local development (uses default config)
wrangler dev
```

**Environment-specific secrets**:

```bash
wrangler secret put DATABASE_PASSWORD --env stage
wrangler secret put DATABASE_PASSWORD --env production
```

## Analytics Engine

Custom analytics with high-cardinality data.

```jsonc
{
  "analytics_engine_datasets": [
    {
      "binding": "ANALYTICS"
    }
  ]
}
```

**Usage**:

```typescript
env.ANALYTICS.writeDataPoint({
  // String dimensions
  blobs: ["user_signup", "web"],

  // Numeric values
  doubles: [1, userId],

  // Indexed fields for querying
  indexes: [userId],
});
```

**Query in dashboard**: GraphQL API or SQL API for analytics queries.

## Version metadata

```jsonc
{
  "version_metadata": {
    "binding": "CF_VERSION_METADATA"
  }
}
```

**Usage**:

```typescript
const metadata = env.CF_VERSION_METADATA;

console.log(`Version ID: ${metadata.id}`);
console.log(`Deployed at: ${metadata.timestamp}`);
```

## Tail consumers

Process logs from other workers in real-time.

```jsonc
{
  "tail_consumers": [
    {
      "service": "log-aggregator"
    }
  ]
}
```

**Log consumer worker**:

```typescript
export default {
  async tail(events: TraceItem[], env: Env) {
    for (const event of events) {
      // Process logs, errors, traces
      await env.DB.prepare(
        "INSERT INTO logs (message, timestamp) VALUES (?, ?)"
      ).bind(event.logs[0].message, event.eventTimestamp).run();
    }
  }
}
```

## Dispatch namespaces (Workers for Platforms)

Deploy user-provided workers dynamically.

```jsonc
{
  "dispatch_namespaces": [
    {
      "binding": "DISPATCHER",
      "namespace": "user-workers"
    }
  ]
}
```

**Usage**:

```typescript
// Deploy user worker
await env.DISPATCHER.put("user-123-worker", workerScript);

// Execute user worker
const worker = env.DISPATCHER.get("user-123-worker");
const response = await worker.fetch(request);
```

**Use case**: SaaS platforms allowing customers to deploy custom workers.

## mTLS certificates

Mutual TLS for secure service-to-service communication.

```jsonc
{
  "mtls_certificates": [
    {
      "binding": "MTLS_CERT",
      "certificate_id": "cert-id"
    }
  ]
}
```

**Usage**:

```typescript
const response = await fetch("https://api.example.com", {
  mtls: {
    certificate_id: env.MTLS_CERT
  }
});
```

## Logpush integration

Stream logs to external destinations (S3, R2, HTTP).

```jsonc
{
  "logpush": true
}
```

Configure logpush destinations in Cloudflare dashboard.

## Rate limiting

```jsonc
{
  "limits": {
    "cpu_ms": 50
  }
}
```

Set CPU time limits per request (default 10ms on free tier, 50ms on paid).

## Cron triggers (scheduled events)

```jsonc
{
  "triggers": {
    "crons": [
      "0 0 * * *"  // Daily at midnight UTC
    ]
  }
}
```

**Handler**:

```typescript
export default {
  async scheduled(event: ScheduledEvent, env: Env, ctx: ExecutionContext) {
    // Run scheduled task
    await cleanupExpiredData(env.DB);
  }
}
```

## Additional resources

- **Wrangler CLI reference**: https://developers.cloudflare.com/workers/wrangler/
- **Bindings documentation**: https://developers.cloudflare.com/workers/runtime-apis/bindings/
- **Configuration schema**: https://developers.cloudflare.com/workers/wrangler/configuration/
- **Platform limits**: https://developers.cloudflare.com/workers/platform/limits/
