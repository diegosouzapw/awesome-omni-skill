---
name: architecture
description: Master architectural design with SOLID principles, design patterns, microservices, and event-driven systems. Learn to design scalable backend systems.
sasmp_version: "2.0.0"
bonded_agent: 04-architecture-patterns
bond_type: PRIMARY_BOND

# === PRODUCTION-GRADE SKILL CONFIG (SASMP v2.0.0) ===

atomic_operations:
  - PATTERN_SELECTION
  - ARCHITECTURE_DESIGN
  - SERVICE_DECOMPOSITION
  - TRADE_OFF_ANALYSIS

parameter_validation:
  query:
    type: string
    required: true
    minLength: 5
    maxLength: 3000
  architecture_type:
    type: string
    enum: [monolith, microservices, serverless, event-driven]
    required: false
  scale:
    type: string
    enum: [startup, growth, enterprise]
    required: false

retry_logic:
  max_attempts: 2
  backoff: exponential
  initial_delay_ms: 2000

logging_hooks:
  on_invoke: "skill.architecture.invoked"
  on_success: "skill.architecture.completed"
  on_error: "skill.architecture.failed"

exit_codes:
  SUCCESS: 0
  INVALID_INPUT: 1
  OVER_ENGINEERING: 2
  ANTI_PATTERN_DETECTED: 3
---

# System Architecture Skill

**Bonded to:** `architecture-patterns-agent`

---

## Quick Start

```bash
# Invoke architecture skill
"Help me decompose this monolith into microservices"
"Which design pattern should I use for notifications?"
"Design an event-driven architecture for order processing"
```

---

## Instructions

1. **Analyze Requirements**: Understand system needs and constraints
2. **Apply SOLID**: Use SOLID principles for clean design
3. **Select Patterns**: Choose appropriate design patterns
4. **Design Architecture**: Select monolithic or distributed approach
5. **Validate Design**: Review for scalability and maintainability

---

## Architecture Selection Matrix

| Style | Best For | Team Size | Complexity | Scale |
|-------|----------|-----------|------------|-------|
| Monolith | MVPs, small teams | 1-10 | Low | Vertical |
| Modular Monolith | Growing apps | 5-20 | Medium | Vertical |
| Microservices | Large teams | 20+ | High | Horizontal |
| Serverless | Variable load | 1-15 | Medium | Auto |
| Event-Driven | Real-time, async | 10+ | High | Horizontal |

---

## Decision Tree

```
Team size & complexity?
    │
    ├─→ Small team (1-5) → Monolith
    │
    ├─→ Growing team (5-20)
    │     ├─→ Clear domain boundaries → Modular Monolith
    │     └─→ Variable load → Serverless
    │
    └─→ Large team (20+)
          ├─→ Real-time/async heavy → Event-Driven
          └─→ Independent scaling needed → Microservices
```

---

## SOLID Principles

| Principle | Description | Violation Sign |
|-----------|-------------|----------------|
| **S**ingle Responsibility | One reason to change | Class does too many things |
| **O**pen/Closed | Open for extension, closed for mod | Editing existing code for new features |
| **L**iskov Substitution | Subtypes substitutable | Subclass changes behavior unexpectedly |
| **I**nterface Segregation | Small, specific interfaces | Clients implement unused methods |
| **D**ependency Inversion | Depend on abstractions | High-level depends on low-level |

---

## Design Patterns Quick Reference

### Creational
| Pattern | Use Case |
|---------|----------|
| Singleton | Global config, connection pools |
| Factory | Object creation with logic |
| Builder | Complex object construction |

### Structural
| Pattern | Use Case |
|---------|----------|
| Adapter | Interface compatibility |
| Decorator | Dynamic behavior extension |
| Facade | Simplified interface |
| Proxy | Access control, caching |

### Behavioral
| Pattern | Use Case |
|---------|----------|
| Observer | Event notifications |
| Strategy | Algorithm selection |
| Command | Action encapsulation |
| State | State machines |

---

## Examples

### Example 1: Microservices Decomposition
```
E-commerce Monolith → Microservices

Services:
├── user-service        (auth, profiles)
├── product-service     (catalog, inventory)
├── order-service       (orders, checkout)
├── payment-service     (transactions)
├── notification-service (email, push)
└── api-gateway         (routing, auth)

Communication:
├── Sync: REST/gRPC for queries
└── Async: Event bus for commands/events
```

### Example 2: Observer Pattern
```python
from abc import ABC, abstractmethod
from typing import List

class Observer(ABC):
    @abstractmethod
    def update(self, event: dict) -> None:
        pass

class NotificationService:
    def __init__(self):
        self._observers: List[Observer] = []

    def subscribe(self, observer: Observer) -> None:
        self._observers.append(observer)

    def notify(self, event: dict) -> None:
        for observer in self._observers:
            observer.update(event)

class EmailHandler(Observer):
    def update(self, event: dict) -> None:
        send_email(event["user_id"], event["message"])

class PushHandler(Observer):
    def update(self, event: dict) -> None:
        send_push(event["user_id"], event["message"])
```

### Example 3: Event-Driven Order Processing
```python
# Saga pattern for order processing
class OrderSaga:
    async def process_order(self, order_id: str):
        try:
            # Step 1: Reserve inventory
            await self.inventory_service.reserve(order_id)

            # Step 2: Process payment
            await self.payment_service.charge(order_id)

            # Step 3: Confirm order
            await self.order_service.confirm(order_id)

        except PaymentError:
            # Compensating transaction
            await self.inventory_service.release(order_id)
            await self.order_service.cancel(order_id)
            raise
```

---

## Troubleshooting

### Anti-Pattern Detection

| Anti-Pattern | Sign | Solution |
|--------------|------|----------|
| Distributed Monolith | Services too coupled | Define bounded contexts |
| God Class | One class does everything | Extract by responsibility |
| Circular Dependencies | A→B→C→A | Introduce interfaces |
| Shared Database | Multiple services, one DB | Database per service |

### Debug Checklist

1. Review dependency graph: Check for cycles
2. Analyze coupling: Services should be loosely coupled
3. Check cohesion: Related functionality together
4. Validate boundaries: Clear service boundaries
5. Test in isolation: Services deployable independently

---

## Test Template

```python
# tests/test_architecture.py
import pytest

class TestServiceBoundaries:
    def test_services_are_independent(self):
        # Each service should work independently
        user_service = UserService()
        order_service = OrderService()

        # Services communicate via events/APIs, not direct calls
        assert not hasattr(order_service, 'user_repository')

    def test_no_circular_dependencies(self):
        from app.services import dependency_graph
        cycles = dependency_graph.find_cycles()
        assert len(cycles) == 0
```

---

## References

See `references/` directory for:
- `ARCHITECTURE_GUIDE.md` - Detailed patterns
- `design-patterns.yaml` - Pattern catalog

---

## Resources

- [Martin Fowler - Patterns](https://martinfowler.com/eaaCatalog/)
- [Microsoft Cloud Patterns](https://docs.microsoft.com/en-us/azure/architecture/patterns/)
- [DDD Reference](https://www.domainlanguage.com/ddd/reference/)
