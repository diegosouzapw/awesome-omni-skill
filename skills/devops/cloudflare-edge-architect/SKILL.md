---
name: cloudflare-edge-architect
description: Advanced architecture skill for Cloudflare. Covers Workers, Durable Objects, R2, D1, and Edge-first data strategies.
version: 2.0.0
author: multi-game-engines contributors
---

# Cloudflare Edge Architect Skill

This skill (formerly `cloudflare-workers-ops`) focuses on building scalable, stateful, and performant architectures on the Cloudflare global network.

## Core Pillars

1.  **Stateful Coordination (Durable Objects)**: Use Durable Objects for real-time multiplayer coordination, game engines state synchronization, and WebSockets.
2.  **Edge Data (D1 & KV)**: Strategy for distributing relational data (D1) and low-latency configuration (KV).
3.  **Object Storage (R2)**: Serve game assets and large binaries efficiently without egress fees.
4.  **Compute-at-Edge (Workers)**: Offload API logic and game logic to the edge to minimize latency.
5.  **AI-at-Edge (Workers AI)**: Integrate on-device inference for features like move suggestions or content moderation.

## Architecture Patterns

### Real-time Multiplayer with Durable Objects

```typescript
// Durable Object implementation for Game Session
export class GameSession {
  state: DurableObjectState;
  constructor(state: DurableObjectState) {
    this.state = state;
  }
  async fetch(request: Request) {
    // Handle game state updates and WebSocket connections
  }
}
```

## Best Practices Checklist

- [ ] **Cross-Origin Isolation**: Set `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp` to enable `SharedArrayBuffer` for Wasm threads.
- [ ] **Smart Placement**: Enable Cloudflare Smart Placement to minimize latency between Workers and backends.
- [ ] **Bundling**: Use `esbuild` or `wrangler`'s built-in bundling to optimize script size.
- [ ] **Environment Secrets**: Securely manage keys for external APIs using `wrangler secret`.

## Advanced Commands

- `pnpm wrangler d1 migrations create <db_name> <description>`
- `pnpm wrangler r2 bucket create <bucket_name>`
- `pnpm wrangler dev --remote`: Test with real Cloudflare resources.

## Troubleshooting

- **"Durable Object not found"**: Ensure the namespace is correctly defined in `wrangler.toml`.
- **"Exceeded CPU limits"**: Optimize tight loops or offload heavy processing to asynchronous tasks.
