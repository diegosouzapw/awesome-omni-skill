---
name: asyncapi-design
description: Event-driven API specification with AsyncAPI 3.0 for message-based architectures
allowed-tools: Read, Glob, Grep, Write, Edit, mcp__perplexity__search, mcp__context7__resolve-library-id, mcp__context7__query-docs
---

# AsyncAPI Design Skill

## When to Use This Skill

Use this skill when:

- **Designing event-driven APIs** - AsyncAPI 3.0 for message-based architectures
- **Configuring message brokers** - Kafka, RabbitMQ, MQTT, WebSocket bindings
- **Implementing in C#** - Event contracts, MassTransit, Confluent Kafka
- **Event versioning** - Schema evolution and backward compatibility

## MANDATORY: Documentation-First Approach

Before creating AsyncAPI specifications:

1. **Invoke `docs-management` skill** for event-driven patterns
2. **Verify AsyncAPI 3.0 syntax** via MCP servers (context7 for latest spec)
3. **Base all guidance on AsyncAPI 3.0 specification**

## AsyncAPI vs OpenAPI

| Aspect | OpenAPI | AsyncAPI |
|--------|---------|----------|
| Communication | Request/Response | Event-Driven |
| Protocol | HTTP/HTTPS | Kafka, RabbitMQ, MQTT, WebSocket |
| Initiator | Client requests | Publisher emits |
| Pattern | Synchronous | Asynchronous |
| Use Case | REST APIs | Message queues, streaming, IoT |

## AsyncAPI 3.0 Structure Overview

```yaml
asyncapi: 3.0.0
info:
  title: API Title
  version: 1.0.0

servers:
  production:
    host: kafka.example.com:9092
    protocol: kafka

channels:
  orderCreated:
    address: orders.created
    messages:
      orderCreatedMessage:
        $ref: '#/components/messages/OrderCreated'

operations:
  publishOrderCreated:
    action: send
    channel:
      $ref: '#/channels/orderCreated'

components:
  messages: { }
  schemas: { }
  securitySchemes: { }
```

**For complete template**: See [basic-template.md](references/basic-template.md)

## Quick Reference

### Supported Protocols

| Protocol | Use Case | Binding Version |
|----------|----------|-----------------|
| Kafka | High-throughput streaming | 0.5.0 |
| AMQP (RabbitMQ) | Message queuing | 0.3.0 |
| MQTT | IoT, lightweight messaging | 0.2.0 |
| WebSocket | Real-time browser comms | - |

**For protocol-specific patterns**: See [protocol-patterns.md](references/protocol-patterns.md)

### Channel Naming Convention

```text
{domain}.{entity}.{action}.{version}
Example: orders.order.created.v1
```

## Workflow

1. **Identify events** - What significant occurrences need to be communicated?
2. **Define channels** - What topics/queues will carry these events?
3. **Design messages** - What data does each event contain?
4. **Choose protocol** - Kafka, RabbitMQ, MQTT, etc.?
5. **Add bindings** - Protocol-specific configuration
6. **Document security** - Authentication and authorization
7. **Version strategy** - How will events evolve?
8. **Generate code** - Use AsyncAPI generator for clients/handlers

## References

Load on-demand based on need:

| Reference | Load When |
|-----------|-----------|
| [basic-template.md](references/basic-template.md) | Creating a new AsyncAPI spec from scratch |
| [protocol-patterns.md](references/protocol-patterns.md) | Configuring Kafka, RabbitMQ, MQTT, WebSocket |
| [csharp-implementation.md](references/csharp-implementation.md) | Implementing in .NET/C# with MassTransit or Confluent |
| [event-design-patterns.md](references/event-design-patterns.md) | Event envelopes, versioning, best practices |

## Related Skills (Cross-Plugin)

| Phase | Skill | Plugin | Purpose |
| ----- | ----- | ------ | ------- |
| DESIGN | `asyncapi-design` (this skill) | formal-specification | Architecture research, pattern selection |
| AUTHORING | `asyncapi-authoring` | spec-driven-development | Concrete YAML file creation |

**Workflow:** Design (research event patterns) → Author (create YAML) → Implement (generate code)

## MCP Research

For current AsyncAPI patterns and tools:

```text
perplexity: "AsyncAPI 3.0 specification" "event-driven API design patterns"
context7: "asyncapi" (for official documentation)
ref: "AsyncAPI spec examples" "Kafka binding patterns"
```

## Version History

- **v2.0.0** (2026-01-17): Refactored to progressive disclosure pattern
  - Extracted 4 reference files (~650 lines)
  - Hub reduced from 789 to ~130 lines
- **v1.0.0** (2025-12-26): Initial release

---

**Last Updated:** 2026-01-17
