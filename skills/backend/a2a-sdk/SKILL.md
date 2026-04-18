---
name: a2a-sdk
description: "Comprehensive Agent2Agent (A2A) JavaScript SDK skill for building A2A-compliant agents and clients. Use when implementing or integrating @a2a-js/sdk, creating A2A servers/clients, using JSON-RPC/REST/gRPC transports, streaming task updates, handling tasks/artifacts, authentication, or push notifications."
---

# A2A JavaScript SDK Skill

## Core Workflows

1. Select transport: JSON-RPC (default), REST, or gRPC.
2. Define an AgentCard and AgentExecutor for server implementations.
3. Verify server responds to agent card discovery before adding task handling.
4. Use ClientFactory for client creation and message exchange.
5. Add task handling and artifacts for long-running operations.
6. Use streaming when real-time updates are required.
7. Add authentication or push notifications when needed.

## Quick Start: Minimal Server

```typescript
import { AgentCard, A2AServer, AgentExecutor, MessageRequest } from '@a2a-js/sdk';

const agentCard: AgentCard = {
  name: 'my-agent',
  description: 'A minimal A2A agent',
  url: 'http://localhost:3000',
  version: '0.3.0',
  capabilities: { streaming: false },
  skills: [{ id: 'echo', name: 'Echo', description: 'Echoes input back' }],
};

const executor: AgentExecutor = {
  async execute(request: MessageRequest) {
    return { type: 'message', content: request.message.parts };
  },
};

const server = new A2AServer(agentCard, executor);
server.start(3000);
```

## Bundled Resources

- `README.md`: Quick overview and basic usage patterns.
- `REFERENCE.md`: Full API reference and advanced features.
- `HELPERS.md`: Helper utilities and common patterns.
- `templates/server/basic-server.ts`: Minimal server template.
- `templates/server/task-server.ts`: Task-based server template.
- `templates/client/basic-client.ts`: Minimal client template.
- `templates/client/streaming-client.ts`: Streaming client template.
- `examples/authentication.md`: Authentication server/client examples.
- `examples/streaming.md`: Streaming server/client examples.

## Implementation Guidance

- Keep AgentCard fields aligned with protocol v0.3.0.
- Prefer direct message responses for simple requests.
- Use tasks for long-running work and publish status updates and artifacts.
- Use streaming for real-time task progress updates.
- Include authentication via custom user builders or authenticated transports when required.
