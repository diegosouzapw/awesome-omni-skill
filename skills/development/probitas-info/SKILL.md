---
name: probitas-info
description: Information about Probitas framework. Use when asked "what is Probitas", explaining its purpose, features, or comparing with other test frameworks.
---

## What is Probitas?

Scenario-based E2E testing framework for backend services (APIs, databases,
message queues).

## Key Features

| Feature           | Description                              |
| ----------------- | ---------------------------------------- |
| Scenario-Based    | Tests as readable scenarios with steps   |
| Built-in Clients  | HTTP, gRPC, GraphQL, SQL, Redis, MongoDB |
| Fluent Assertions | Unified `expect()` with chainable checks |
| Auto Cleanup      | Resources with automatic cleanup         |
| Batteries         | faker, FakeTime, spy, stub included      |

## Quick Example

```typescript
import { client, expect, scenario } from "jsr:@probitas/probitas";

export default scenario("API Test", { tags: ["http"] })
  .resource("http", () =>
    client.http.createHttpClient({
      url: Deno.env.get("API_URL") ?? "http://localhost:8080",
    }))
  .step("GET /users", async (ctx) => {
    const res = await ctx.resources.http.get("/users");
    expect(res).toBeOk().toHaveStatus(200);
  })
  .build();
```

## Available Clients

| Client     | Factory Function                             | Use Case             |
| ---------- | -------------------------------------------- | -------------------- |
| HTTP       | `client.http.createHttpClient()`             | REST APIs, webhooks  |
| PostgreSQL | `client.sql.postgres.createPostgresClient()` | PostgreSQL databases |
| MySQL      | `client.sql.mysql.createMySqlClient()`       | MySQL databases      |
| SQLite     | `client.sql.sqlite.createSqliteClient()`     | Embedded databases   |
| DuckDB     | `client.sql.duckdb.createDuckDbClient()`     | Analytics databases  |
| gRPC       | `client.grpc.createGrpcClient()`             | gRPC services        |
| ConnectRPC | `client.connectrpc.createConnectRpcClient()` | Connect/gRPC-Web     |
| GraphQL    | `client.graphql.createGraphqlClient()`       | GraphQL APIs         |
| Redis      | `client.redis.createRedisClient()`           | Cache, pub/sub       |
| MongoDB    | `client.mongodb.createMongoClient()`         | Document databases   |
| Deno KV    | `client.deno_kv.createDenoKvClient()`        | Deno KV store        |
| RabbitMQ   | `client.rabbitmq.createRabbitMqClient()`     | AMQP message queues  |
| SQS        | `client.sqs.createSqsClient()`               | AWS message queues   |

## API Reference

Use `deno doc` to look up API:

```bash
deno doc jsr:@probitas/probitas
deno doc jsr:@probitas/probitas/client/http
deno doc jsr:@probitas/probitas/client/grpc
deno doc jsr:@probitas/probitas/client/graphql
```

## Documentation

- LLM sitemap: https://probitas-test.github.io/documents/llms.txt
