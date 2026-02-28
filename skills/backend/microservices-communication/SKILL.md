---
name: Microservices Communication
description: "Thiết kế kiến trúc giao tiếp Microservices (gRPC, message queues, event-driven pattern)."
category: Backend Architecture
difficulty: Advanced
last_updated: 2026-01-16
---

# Microservices Communication Patterns

Expert patterns for modern microservices architectures and distributed systems.

---

## When to Use This Skill

- Building microservices architectures
- Implementing async communication
- Designing event-driven systems
- Service mesh implementation
- API Gateway patterns
- Distributed transactions

---

## Content Map

### Communication Patterns
- **synchronous.md** - REST, gRPC, GraphQL
- **asynchronous.md** - Message queues, pub/sub, event streaming
- **hybrid.md** - Mixed patterns, CQRS

### Message Queues
- **rabbitmq.md** - RabbitMQ patterns (work queues, pub/sub, RPC)
- **kafka.md** - Apache Kafka (event streaming, topics, partitions)
- **sqs-sns.md** - AWS SQS/SNS patterns

### gRPC
- **grpc-basics.md** - Protobuf, service definition
- **grpc-streaming.md** - Server/client/bidirectional streaming
- **grpc-production.md** - Load balancing, health checks, interceptors

### Event-Driven
- **event-sourcing.md** - Event store, projections, replay
- **cqrs.md** - Command Query Responsibility Segregation
- **saga-pattern.md** - Distributed transactions, choreography vs orchestration

### Service Mesh
- **service-mesh-intro.md** - Istio, Linkerd, Consul
- **traffic-management.md** - Load balancing, circuit breaking, retries
- **security.md** - mTLS, authorization

### API Gateway
- **gateway-patterns.md** - Kong, Nginx, AWS API Gateway
- **rate-limiting.md** - Throttling, quotas
- **authentication.md** - OAuth, JWT validation

---

## Quick Reference

### Communication Pattern Decision Tree

```
Choose Communication Pattern:

Real-time request-response needed?
├── YES → Synchronous (REST/gRPC)
│   ├── High performance? → gRPC
│   └── Simple/ubiquitous? → REST
│
└── NO → Asynchronous (Message Queue)
    ├── Fan-out to many consumers? → Pub/Sub (SNS, Kafka)
    ├── Work queue? → RabbitMQ, SQS
    └── Event streaming? → Kafka, Kinesis
```

---

### gRPC Example

```protobuf
// user.proto
syntax = "proto3";

service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
  rpc StreamUsers (Empty) returns (stream UserResponse);
}

message UserRequest {
  string user_id = 1;
}

message UserResponse {
  string user_id = 1;
  string name = 2;
  string email = 3;
}
```

```javascript
// Node.js gRPC Server
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const packageDefinition = protoLoader.loadSync('user.proto');
const proto = grpc.loadPackageDefinition(packageDefinition);

const server = new grpc.Server();

server.addService(proto.UserService.service, {
  getUser: (call, callback) => {
    const userId = call.request.user_id;
    callback(null, {
      user_id: userId,
      name: 'John Doe',
      email: 'john@example.com'
    });
  },
  streamUsers: (call) => {
    // Stream users
    users.forEach(user => call.write(user));
    call.end();
  }
});

server.bindAsync('0.0.0.0:50051', 
  grpc.ServerCredentials.createInsecure(), 
  () => server.start()
);
```

---

### Message Queue Patterns

#### RabbitMQ Work Queue

```javascript
// Producer
const amqp = require('amqplib');

async function sendTask(task) {
  const connection = await amqp.connect('amqp://localhost');
  const channel = await connection.createChannel();
  
  await channel.assertQueue('tasks', { durable: true });
  channel.sendToQueue('tasks', Buffer.from(JSON.stringify(task)), {
    persistent: true
  });
}
```

```javascript
// Consumer
async function consumeTasks() {
  const connection = await amqp.connect('amqp://localhost');
  const channel = await connection.createChannel();
  
  await channel.assertQueue('tasks', { durable: true });
  channel.prefetch(1);
  
  channel.consume('tasks', async (msg) => {
    const task = JSON.parse(msg.content.toString());
    await processTask(task);
    channel.ack(msg);
  });
}
```

#### Kafka Event Streaming

```javascript
// Kafka Producer
const { Kafka } = require('kafkajs');

const kafka = new Kafka({ brokers: ['localhost:9092'] });
const producer = kafka.producer();

await producer.connect();
await producer.send({
  topic: 'user-events',
  messages: [
    { key: userId, value: JSON.stringify(event) }
  ]
});
```

```javascript
// Kafka Consumer
const consumer = kafka.consumer({ groupId: 'user-service' });

await consumer.connect();
await consumer.subscribe({ topic: 'user-events' });

await consumer.run({
  eachMessage: async ({ topic, partition, message }) => {
    const event = JSON.parse(message.value.toString());
    await handleEvent(event);
  }
});
```

---

### Event-Driven Architecture

#### Event Sourcing

```typescript
// Event Store
class EventStore {
  async appendEvent(streamId: string, event: Event) {
    await db.events.insert({
      streamId,
      eventType: event.type,
      data: event.data,
      timestamp: new Date()
    });
  }
  
  async getEvents(streamId: string): Promise<Event[]> {
    return db.events.find({ streamId }).sort({ timestamp: 1 });
  }
}

// Aggregate
class UserAggregate {
  private events: Event[] = [];
  
  apply(event: Event) {
    this.events.push(event);
    this.applyEvent(event);
  }
  
  private applyEvent(event: Event) {
    switch(event.type) {
      case 'UserCreated':
        this.id = event.data.id;
        this.name = event.data.name;
        break;
      case 'UserUpdated':
        this.name = event.data.name;
        break;
    }
  }
}
```

#### Saga Pattern (Orchestration)

```typescript
class OrderSaga {
  async execute(order: Order) {
    try {
      // Step 1: Reserve inventory
      await inventoryService.reserve(order.items);
      
      // Step 2: Process payment
      await paymentService.charge(order.total);
      
      // Step 3: Ship order
      await shippingService.ship(order);
      
      // Success
      await this.complete(order);
    } catch (error) {
      // Compensate (rollback)
      await this.compensate(order, error);
    }
  }
  
  private async compensate(order: Order, error: Error) {
    await inventoryService.release(order.items);
    await paymentService.refund(order.total);
    await this.markFailed(order, error);
  }
}
```

---

### Service Mesh Patterns

#### Circuit Breaker

```yaml
# Istio DestinationRule
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: user-service
spec:
  host: user-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
```

#### Retry Policy

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: user-service
spec:
  hosts:
  - user-service
  http:
  - retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset
    route:
    - destination:
        host: user-service
```

---

## Anti-Patterns

❌ **Synchronous coupling everywhere** → Use async for non-critical paths  
❌ **No idempotency** → Messages can be delivered multiple times  
❌ **No circuit breakers** → Cascading failures  
❌ **Chatty services** → Aggregate calls, use events  
❌ **No message versioning** → Breaking changes hurt consumers

---

## Best Practices

✅ **Idempotent consumers** - Handle duplicate messages  
✅ **Circuit breakers** - Fail fast, prevent cascades  
✅ **Message versioning** - Schema evolution  
✅ **Saga for distributed transactions** - No 2PC  
✅ **Service mesh for cross-cutting** - Don't reinvent  
✅ **Event sourcing when needed** - Audit, replay, projections

---

## Related Skills

- `api-patterns` - REST API design
- `graphql-patterns` - GraphQL schemas
- `kubernetes-patterns` - Deployment
- `monitoring-observability` - Distributed tracing

---

## Official Resources

- [gRPC Documentation](https://grpc.io/)
- [RabbitMQ Tutorials](https://www.rabbitmq.com/getstarted.html)
- [Apache Kafka](https://kafka.apache.org/)
- [Istio Service Mesh](https://istio.io/)
- [Microservices.io Patterns](https://microservices.io/patterns/)
