---
name: enterprise-architecture-patterns
description: Complete guide for enterprise architecture patterns including domain-driven design, event sourcing, CQRS, saga patterns, API gateway, service mesh, and scalability
tags: [enterprise-architecture, ddd, event-sourcing, cqrs, microservices, scalability, patterns]
tier: tier-1
---

# Enterprise Architecture Patterns

A comprehensive skill for mastering enterprise architecture patterns, distributed systems design, and scalable application development. This skill covers strategic and tactical patterns for building robust, maintainable, and scalable enterprise systems.

## When to Use This Skill

Use this skill when:

- Designing microservices architectures for distributed systems
- Implementing domain-driven design (DDD) in complex business domains
- Building event-driven architectures with event sourcing and CQRS
- Designing saga patterns for distributed transactions
- Implementing API gateways and service mesh architectures
- Scaling applications horizontally and vertically
- Building resilient systems with fault tolerance
- Migrating monoliths to microservices
- Designing multi-tenant SaaS architectures
- Implementing backend-for-frontend (BFF) patterns
- Building real-time systems with event streaming
- Architecting cloud-native applications

## Core Architectural Concepts

### System Design Fundamentals

**Separation of Concerns**
Divide systems into distinct sections where each section addresses a separate concern, reducing coupling and increasing cohesion.

**Modularity**
Design systems as collections of independent modules that can be developed, tested, and deployed separately.

**Abstraction**
Hide complex implementation details behind simple interfaces, making systems easier to understand and modify.

**Scalability Dimensions**
- Horizontal scaling: Add more machines/instances
- Vertical scaling: Add more resources to existing machines
- Data scaling: Partition data across multiple stores
- Functional scaling: Decompose by business capability

**Consistency Models**
- Strong consistency: All nodes see the same data at the same time
- Eventual consistency: All nodes will eventually see the same data
- Causal consistency: Related operations see consistent state
- Read-your-writes consistency: Users see their own updates immediately

**CAP Theorem**
In distributed systems, you can only guarantee two of three properties:
- Consistency: All nodes see the same data
- Availability: Every request receives a response
- Partition tolerance: System continues despite network failures

**Distributed Computing Fallacies**
1. The network is reliable
2. Latency is zero
3. Bandwidth is infinite
4. The network is secure
5. Topology doesn't change
6. There is one administrator
7. Transport cost is zero
8. The network is homogeneous

## Domain-Driven Design (DDD)

### Strategic Design Patterns

#### Bounded Context

A bounded context is an explicit boundary within which a domain model is consistent and valid. It defines the scope where particular terms, definitions, and rules apply.

**Key Principles:**
- Each bounded context has its own ubiquitous language
- Models within a context are consistent
- Cross-context integration requires translation
- Contexts align with business capabilities

**Implementation:**
```typescript
// Example: E-commerce system with multiple bounded contexts

// Sales Context
namespace Sales {
  class Customer {
    customerId: string;
    email: string;
    orderHistory: Order[];

    placeOrder(order: Order): void {
      // Sales-specific logic
    }
  }
}

// Billing Context
namespace Billing {
  class Customer {
    customerId: string;
    paymentMethods: PaymentMethod[];
    invoices: Invoice[];

    processPayment(invoice: Invoice): void {
      // Billing-specific logic
    }
  }
}

// Different models for Customer in different contexts
```

**Context Mapping Patterns:**

1. **Shared Kernel**: Two contexts share a subset of the domain model
   - Use when: Teams are closely coordinated
   - Risk: Changes affect multiple contexts

2. **Customer-Supplier**: Downstream context depends on upstream
   - Use when: Clear dependency direction exists
   - Pattern: Upstream provides defined API

3. **Conformist**: Downstream conforms to upstream model
   - Use when: Upstream is external/unchangeable
   - Pattern: Adapt to external API

4. **Anti-Corruption Layer**: Translate between contexts
   - Use when: Protecting from legacy or external systems
   - Pattern: Adapter/facade to translate models

5. **Separate Ways**: Contexts are completely independent
   - Use when: No integration needed
   - Pattern: Duplicate functionality if necessary

6. **Open Host Service**: Well-defined protocol for integration
   - Use when: Multiple consumers need access
   - Pattern: REST API, GraphQL, gRPC

7. **Published Language**: Shared, well-documented language
   - Use when: Industry standards exist
   - Pattern: XML schemas, JSON schemas, OpenAPI

#### Ubiquitous Language

A shared vocabulary between developers and domain experts used consistently in code, documentation, and conversations.

**Building Ubiquitous Language:**
```typescript
// Bad: Generic technical terms
class DataProcessor {
  processData(data: any): void {
    // Unclear what this does in business terms
  }
}

// Good: Business domain terms
class OrderFulfillment {
  fulfillOrder(order: Order): void {
    this.pickItems(order.items);
    this.packForShipment(order);
    this.scheduleDelivery(order);
  }

  private pickItems(items: OrderItem[]): void {
    // Business logic using domain language
  }
}
```

### Tactical Design Patterns

#### Entities

Objects with unique identity that persist over time, tracking continuity and lifecycle.

**Characteristics:**
- Unique identifier (ID)
- Mutable state
- Lifecycle (created, modified, deleted)
- Equality based on identity, not attributes

**Implementation:**
```typescript
class Order {
  private readonly orderId: string;
  private orderItems: OrderItem[];
  private status: OrderStatus;
  private orderDate: Date;
  private customerId: string;

  constructor(orderId: string, customerId: string) {
    this.orderId = orderId;
    this.customerId = customerId;
    this.orderItems = [];
    this.status = OrderStatus.Draft;
    this.orderDate = new Date();
  }

  // Business behavior
  addItem(product: Product, quantity: number): void {
    if (this.status !== OrderStatus.Draft) {
      throw new Error("Cannot modify confirmed order");
    }
    this.orderItems.push(new OrderItem(product, quantity));
  }

  confirm(): void {
    if (this.orderItems.length === 0) {
      throw new Error("Cannot confirm empty order");
    }
    this.status = OrderStatus.Confirmed;
  }

  // Identity-based equality
  equals(other: Order): boolean {
    return this.orderId === other.orderId;
  }
}
```

#### Value Objects

Immutable objects defined by their attributes rather than identity, representing descriptive aspects of the domain.

**Characteristics:**
- No unique identifier
- Immutable (cannot change state)
- Equality based on all attributes
- Often passed by value
- Can be shared safely

**Implementation:**
```typescript
class Money {
  readonly amount: number;
  readonly currency: string;

  constructor(amount: number, currency: string) {
    if (amount < 0) {
      throw new Error("Amount cannot be negative");
    }
    this.amount = amount;
    this.currency = currency;
  }

  // Operations return new instances
  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error("Cannot add different currencies");
    }
    return new Money(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  // Value-based equality
  equals(other: Money): boolean {
    return this.amount === other.amount &&
           this.currency === other.currency;
  }
}

class Address {
  readonly street: string;
  readonly city: string;
  readonly state: string;
  readonly zipCode: string;
  readonly country: string;

  constructor(
    street: string,
    city: string,
    state: string,
    zipCode: string,
    country: string
  ) {
    this.street = street;
    this.city = city;
    this.state = state;
    this.zipCode = zipCode;
    this.country = country;
  }

  equals(other: Address): boolean {
    return this.street === other.street &&
           this.city === other.city &&
           this.state === other.state &&
           this.zipCode === other.zipCode &&
           this.country === other.country;
  }
}
```

#### Aggregates

Clusters of entities and value objects with clear consistency boundaries, accessed through a single root entity.

**Key Principles:**
- One aggregate = one transaction boundary
- External references only to aggregate root
- Root enforces all invariants
- Small aggregates perform better
- Eventual consistency between aggregates

**Design Rules:**
1. Model true invariants in consistency boundaries
2. Design small aggregates
3. Reference other aggregates by identity only
4. Update other aggregates using eventual consistency
5. Use repositories to retrieve aggregates

**Implementation:**
```typescript
// Aggregate Root
class Order {
  private readonly orderId: string;
  private readonly customerId: string; // Reference by ID only
  private orderItems: OrderItem[] = [];
  private shippingAddress: Address;
  private status: OrderStatus;
  private totalAmount: Money;

  constructor(orderId: string, customerId: string, shippingAddress: Address) {
    this.orderId = orderId;
    this.customerId = customerId;
    this.shippingAddress = shippingAddress;
    this.status = OrderStatus.Draft;
    this.totalAmount = new Money(0, "USD");
  }

  // Public methods enforce invariants
  addItem(product: Product, quantity: number): void {
    if (this.status !== OrderStatus.Draft) {
      throw new Error("Cannot modify confirmed order");
    }

    if (quantity <= 0) {
      throw new Error("Quantity must be positive");
    }

    const existingItem = this.findItem(product.id);
    if (existingItem) {
      existingItem.increaseQuantity(quantity);
    } else {
      this.orderItems.push(new OrderItem(product, quantity));
    }

    this.recalculateTotal();
  }

  removeItem(productId: string): void {
    if (this.status !== OrderStatus.Draft) {
      throw new Error("Cannot modify confirmed order");
    }

    this.orderItems = this.orderItems.filter(
      item => item.productId !== productId
    );
    this.recalculateTotal();
  }

  confirm(): void {
    if (this.orderItems.length === 0) {
      throw new Error("Cannot confirm empty order");
    }

    if (!this.shippingAddress) {
      throw new Error("Shipping address required");
    }

    this.status = OrderStatus.Confirmed;
  }

  private recalculateTotal(): void {
    this.totalAmount = this.orderItems.reduce(
      (total, item) => total.add(item.subtotal),
      new Money(0, "USD")
    );
  }

  private findItem(productId: string): OrderItem | undefined {
    return this.orderItems.find(item => item.productId === productId);
  }

  // Getters for read-only access
  get id(): string { return this.orderId; }
  get total(): Money { return this.totalAmount; }
  get items(): readonly OrderItem[] { return this.orderItems; }
}

// Entity within aggregate
class OrderItem {
  readonly productId: string;
  readonly productName: string;
  readonly unitPrice: Money;
  private quantity: number;

  constructor(product: Product, quantity: number) {
    this.productId = product.id;
    this.productName = product.name;
    this.unitPrice = product.price;
    this.quantity = quantity;
  }

  increaseQuantity(amount: number): void {
    this.quantity += amount;
  }

  get subtotal(): Money {
    return this.unitPrice.multiply(this.quantity);
  }
}
```

#### Domain Events

Events that represent something significant that happened in the domain, enabling loose coupling and eventual consistency.

**Characteristics:**
- Past tense naming (OrderPlaced, PaymentProcessed)
- Immutable
- Include all necessary information
- Timestamped
- Often include aggregate ID

**Implementation:**
```typescript
interface DomainEvent {
  eventId: string;
  occurredAt: Date;
  aggregateId: string;
  eventType: string;
}

class OrderPlacedEvent implements DomainEvent {
  readonly eventId: string;
  readonly occurredAt: Date;
  readonly aggregateId: string;
  readonly eventType = "OrderPlaced";

  readonly orderId: string;
  readonly customerId: string;
  readonly totalAmount: Money;
  readonly items: OrderItemDto[];

  constructor(order: Order) {
    this.eventId = generateId();
    this.occurredAt = new Date();
    this.aggregateId = order.id;
    this.orderId = order.id;
    this.customerId = order.customerId;
    this.totalAmount = order.total;
    this.items = order.items.map(item => ({
      productId: item.productId,
      quantity: item.quantity,
      price: item.unitPrice
    }));
  }
}

// Domain event publisher
class DomainEventPublisher {
  private handlers: Map<string, Function[]> = new Map();

  subscribe(eventType: string, handler: Function): void {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }
    this.handlers.get(eventType)!.push(handler);
  }

  async publish(event: DomainEvent): Promise<void> {
    const handlers = this.handlers.get(event.eventType) || [];
    await Promise.all(handlers.map(handler => handler(event)));
  }
}

// Usage
class OrderService {
  constructor(
    private orderRepository: OrderRepository,
    private eventPublisher: DomainEventPublisher
  ) {}

  async placeOrder(order: Order): Promise<void> {
    order.confirm();
    await this.orderRepository.save(order);

    const event = new OrderPlacedEvent(order);
    await this.eventPublisher.publish(event);
  }
}
```

#### Repositories

Abstraction for accessing aggregates, providing collection-like interface while hiding persistence details.

**Principles:**
- One repository per aggregate root
- Collection-oriented interface
- Hide database implementation
- Return fully-formed aggregates
- Support querying by ID and business criteria

**Implementation:**
```typescript
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(orderId: string): Promise<Order | null>;
  findByCustomer(customerId: string): Promise<Order[]>;
  findByStatus(status: OrderStatus): Promise<Order[]>;
  delete(orderId: string): Promise<void>;
}

class OrderRepositoryImpl implements OrderRepository {
  constructor(private db: Database) {}

  async save(order: Order): Promise<void> {
    const data = this.toDataModel(order);
    await this.db.orders.upsert(data);
  }

  async findById(orderId: string): Promise<Order | null> {
    const data = await this.db.orders.findOne({ id: orderId });
    if (!data) return null;
    return this.toDomainModel(data);
  }

  async findByCustomer(customerId: string): Promise<Order[]> {
    const results = await this.db.orders.find({ customerId });
    return results.map(data => this.toDomainModel(data));
  }

  async findByStatus(status: OrderStatus): Promise<Order[]> {
    const results = await this.db.orders.find({ status });
    return results.map(data => this.toDomainModel(data));
  }

  async delete(orderId: string): Promise<void> {
    await this.db.orders.delete({ id: orderId });
  }

  private toDataModel(order: Order): any {
    // Convert domain model to database model
    return {
      id: order.id,
      customerId: order.customerId,
      items: order.items.map(item => ({
        productId: item.productId,
        quantity: item.quantity,
        price: item.unitPrice.amount
      })),
      total: order.total.amount,
      status: order.status
    };
  }

  private toDomainModel(data: any): Order {
    // Reconstruct domain model from database data
    const order = new Order(data.id, data.customerId, data.shippingAddress);
    // Restore items and state
    return order;
  }
}
```

#### Domain Services

Operations that don't naturally belong to any entity or value object, encapsulating domain logic that involves multiple aggregates.

**When to Use:**
- Logic spans multiple aggregates
- Operation is a significant domain concept
- Behavior doesn't fit naturally in entity or value object

**Implementation:**
```typescript
class PricingService {
  calculateOrderTotal(
    items: OrderItem[],
    customer: Customer,
    promotions: Promotion[]
  ): Money {
    let total = items.reduce(
      (sum, item) => sum.add(item.subtotal),
      new Money(0, "USD")
    );

    // Apply customer discount
    if (customer.isPremium) {
      total = total.multiply(0.9); // 10% discount
    }

    // Apply promotions
    for (const promo of promotions) {
      total = promo.apply(total);
    }

    return total;
  }
}

class TransferService {
  transfer(
    fromAccount: Account,
    toAccount: Account,
    amount: Money
  ): void {
    if (!fromAccount.canWithdraw(amount)) {
      throw new Error("Insufficient funds");
    }

    fromAccount.withdraw(amount);
    toAccount.deposit(amount);
  }
}
```

## Event Sourcing

Event sourcing persists the state of a system as a sequence of events rather than storing current state. The current state is derived by replaying events.

### Core Concepts

**Event Store**
Append-only log of all events that have occurred in the system.

**Event Stream**
Sequence of events for a specific aggregate, ordered by time.

**Projection**
Read model built by processing events, optimized for queries.

**Snapshot**
Cached state at a point in time to avoid replaying all events.

### Implementation

```typescript
// Event interface
interface Event {
  eventId: string;
  eventType: string;
  aggregateId: string;
  aggregateType: string;
  version: number;
  timestamp: Date;
  data: any;
  metadata?: any;
}

// Account aggregate with event sourcing
class Account {
  private accountId: string;
  private balance: number = 0;
  private isActive: boolean = true;
  private version: number = 0;
  private uncommittedEvents: Event[] = [];

  constructor(accountId: string) {
    this.accountId = accountId;
  }

  // Command handlers
  open(initialBalance: number): void {
    if (this.version > 0) {
      throw new Error("Account already opened");
    }
    this.applyEvent({
      eventType: "AccountOpened",
      data: { accountId: this.accountId, initialBalance }
    });
  }

  deposit(amount: number): void {
    if (!this.isActive) {
      throw new Error("Account is closed");
    }
    if (amount <= 0) {
      throw new Error("Amount must be positive");
    }
    this.applyEvent({
      eventType: "MoneyDeposited",
      data: { amount }
    });
  }

  withdraw(amount: number): void {
    if (!this.isActive) {
      throw new Error("Account is closed");
    }
    if (amount <= 0) {
      throw new Error("Amount must be positive");
    }
    if (this.balance < amount) {
      throw new Error("Insufficient funds");
    }
    this.applyEvent({
      eventType: "MoneyWithdrawn",
      data: { amount }
    });
  }

  close(): void {
    if (!this.isActive) {
      throw new Error("Account already closed");
    }
    if (this.balance > 0) {
      throw new Error("Cannot close account with positive balance");
    }
    this.applyEvent({
      eventType: "AccountClosed",
      data: {}
    });
  }

  // Event application
  private applyEvent(eventData: Partial<Event>): void {
    const event: Event = {
      eventId: generateId(),
      eventType: eventData.eventType!,
      aggregateId: this.accountId,
      aggregateType: "Account",
      version: this.version + 1,
      timestamp: new Date(),
      data: eventData.data,
      metadata: eventData.metadata
    };

    this.apply(event);
    this.uncommittedEvents.push(event);
  }

  // Event handlers (state mutations)
  private apply(event: Event): void {
    switch (event.eventType) {
      case "AccountOpened":
        this.balance = event.data.initialBalance;
        this.isActive = true;
        break;

      case "MoneyDeposited":
        this.balance += event.data.amount;
        break;

      case "MoneyWithdrawn":
        this.balance -= event.data.amount;
        break;

      case "AccountClosed":
        this.isActive = false;
        break;

      default:
        throw new Error(`Unknown event type: ${event.eventType}`);
    }

    this.version = event.version;
  }

  // Replay events to rebuild state
  static fromEvents(events: Event[]): Account {
    if (events.length === 0) {
      throw new Error("Cannot create account from empty event stream");
    }

    const account = new Account(events[0].aggregateId);
    events.forEach(event => account.apply(event));
    return account;
  }

  getUncommittedEvents(): Event[] {
    return this.uncommittedEvents;
  }

  markEventsAsCommitted(): void {
    this.uncommittedEvents = [];
  }
}

// Event store interface
interface EventStore {
  append(events: Event[]): Promise<void>;
  getEvents(aggregateId: string, fromVersion?: number): Promise<Event[]>;
  getAllEvents(fromTimestamp?: Date): Promise<Event[]>;
}

// Event store implementation
class InMemoryEventStore implements EventStore {
  private events: Map<string, Event[]> = new Map();
  private allEvents: Event[] = [];

  async append(events: Event[]): Promise<void> {
    for (const event of events) {
      // Store in aggregate stream
      if (!this.events.has(event.aggregateId)) {
        this.events.set(event.aggregateId, []);
      }
      this.events.get(event.aggregateId)!.push(event);

      // Store in global stream
      this.allEvents.push(event);
    }
  }

  async getEvents(
    aggregateId: string,
    fromVersion: number = 0
  ): Promise<Event[]> {
    const events = this.events.get(aggregateId) || [];
    return events.filter(e => e.version > fromVersion);
  }

  async getAllEvents(fromTimestamp?: Date): Promise<Event[]> {
    if (!fromTimestamp) {
      return this.allEvents;
    }
    return this.allEvents.filter(e => e.timestamp >= fromTimestamp);
  }
}

// Repository with event sourcing
class EventSourcedAccountRepository {
  constructor(private eventStore: EventStore) {}

  async save(account: Account): Promise<void> {
    const events = account.getUncommittedEvents();
    if (events.length > 0) {
      await this.eventStore.append(events);
      account.markEventsAsCommitted();
    }
  }

  async findById(accountId: string): Promise<Account | null> {
    const events = await this.eventStore.getEvents(accountId);
    if (events.length === 0) {
      return null;
    }
    return Account.fromEvents(events);
  }
}
```

### Snapshots

Optimize performance by periodically saving aggregate state:

```typescript
interface Snapshot {
  aggregateId: string;
  version: number;
  timestamp: Date;
  state: any;
}

class SnapshotStore {
  private snapshots: Map<string, Snapshot> = new Map();

  async save(snapshot: Snapshot): Promise<void> {
    this.snapshots.set(snapshot.aggregateId, snapshot);
  }

  async getLatest(aggregateId: string): Promise<Snapshot | null> {
    return this.snapshots.get(aggregateId) || null;
  }
}

class EventSourcedAccountRepositoryWithSnapshots {
  constructor(
    private eventStore: EventStore,
    private snapshotStore: SnapshotStore,
    private snapshotInterval: number = 100
  ) {}

  async save(account: Account): Promise<void> {
    const events = account.getUncommittedEvents();
    await this.eventStore.append(events);
    account.markEventsAsCommitted();

    // Create snapshot every N events
    if (account.version % this.snapshotInterval === 0) {
      await this.snapshotStore.save({
        aggregateId: account.id,
        version: account.version,
        timestamp: new Date(),
        state: account.toSnapshot()
      });
    }
  }

  async findById(accountId: string): Promise<Account | null> {
    // Try to load from snapshot
    const snapshot = await this.snapshotStore.getLatest(accountId);
    let account: Account;
    let fromVersion = 0;

    if (snapshot) {
      account = Account.fromSnapshot(snapshot.state);
      fromVersion = snapshot.version;
    } else {
      account = new Account(accountId);
    }

    // Apply events after snapshot
    const events = await this.eventStore.getEvents(accountId, fromVersion);
    events.forEach(event => account.apply(event));

    return account;
  }
}
```

## CQRS (Command Query Responsibility Segregation)

Separate read and write operations into different models, optimizing each for its specific use case.

### Architecture

```
Commands → Command Handlers → Aggregates → Events → Event Store
                                                   ↓
                                              Event Bus
                                                   ↓
                                           Projections → Read Models → Queries
```

### Implementation

```typescript
// Commands (write operations)
interface Command {
  commandId: string;
  timestamp: Date;
}

class CreateAccountCommand implements Command {
  commandId: string;
  timestamp: Date;
  accountId: string;
  initialBalance: number;

  constructor(accountId: string, initialBalance: number) {
    this.commandId = generateId();
    this.timestamp = new Date();
    this.accountId = accountId;
    this.initialBalance = initialBalance;
  }
}

class DepositMoneyCommand implements Command {
  commandId: string;
  timestamp: Date;
  accountId: string;
  amount: number;

  constructor(accountId: string, amount: number) {
    this.commandId = generateId();
    this.timestamp = new Date();
    this.accountId = accountId;
    this.amount = amount;
  }
}

// Command handlers
class AccountCommandHandler {
  constructor(
    private repository: EventSourcedAccountRepository,
    private eventBus: EventBus
  ) {}

  async handle(command: Command): Promise<void> {
    if (command instanceof CreateAccountCommand) {
      await this.handleCreateAccount(command);
    } else if (command instanceof DepositMoneyCommand) {
      await this.handleDepositMoney(command);
    }
  }

  private async handleCreateAccount(
    command: CreateAccountCommand
  ): Promise<void> {
    const account = new Account(command.accountId);
    account.open(command.initialBalance);
    await this.repository.save(account);

    // Publish events
    const events = account.getUncommittedEvents();
    await this.eventBus.publish(events);
  }

  private async handleDepositMoney(
    command: DepositMoneyCommand
  ): Promise<void> {
    const account = await this.repository.findById(command.accountId);
    if (!account) {
      throw new Error("Account not found");
    }

    account.deposit(command.amount);
    await this.repository.save(account);

    const events = account.getUncommittedEvents();
    await this.eventBus.publish(events);
  }
}

// Read models (optimized for queries)
interface AccountReadModel {
  accountId: string;
  balance: number;
  status: string;
  lastActivity: Date;
  transactionCount: number;
}

interface AccountSummaryReadModel {
  accountId: string;
  balance: number;
  status: string;
}

// Projections (build read models from events)
class AccountProjection {
  constructor(private db: ReadDatabase) {}

  async handleEvent(event: Event): Promise<void> {
    switch (event.eventType) {
      case "AccountOpened":
        await this.handleAccountOpened(event);
        break;
      case "MoneyDeposited":
        await this.handleMoneyDeposited(event);
        break;
      case "MoneyWithdrawn":
        await this.handleMoneyWithdrawn(event);
        break;
      case "AccountClosed":
        await this.handleAccountClosed(event);
        break;
    }
  }

  private async handleAccountOpened(event: Event): Promise<void> {
    await this.db.accounts.insert({
      accountId: event.aggregateId,
      balance: event.data.initialBalance,
      status: "Active",
      lastActivity: event.timestamp,
      transactionCount: 0
    });
  }

  private async handleMoneyDeposited(event: Event): Promise<void> {
    await this.db.accounts.update(
      { accountId: event.aggregateId },
      {
        $inc: { balance: event.data.amount, transactionCount: 1 },
        $set: { lastActivity: event.timestamp }
      }
    );
  }

  private async handleMoneyWithdrawn(event: Event): Promise<void> {
    await this.db.accounts.update(
      { accountId: event.aggregateId },
      {
        $inc: { balance: -event.data.amount, transactionCount: 1 },
        $set: { lastActivity: event.timestamp }
      }
    );
  }

  private async handleAccountClosed(event: Event): Promise<void> {
    await this.db.accounts.update(
      { accountId: event.aggregateId },
      {
        $set: {
          status: "Closed",
          lastActivity: event.timestamp
        }
      }
    );
  }
}

// Query service (read-only)
class AccountQueryService {
  constructor(private db: ReadDatabase) {}

  async getAccount(accountId: string): Promise<AccountReadModel | null> {
    return await this.db.accounts.findOne({ accountId });
  }

  async getAccountsByStatus(status: string): Promise<AccountSummaryReadModel[]> {
    return await this.db.accounts.find(
      { status },
      { projection: { accountId: 1, balance: 1, status: 1 } }
    );
  }

  async getHighBalanceAccounts(
    minBalance: number
  ): Promise<AccountSummaryReadModel[]> {
    return await this.db.accounts.find(
      { balance: { $gte: minBalance } },
      { projection: { accountId: 1, balance: 1, status: 1 } }
    );
  }
}

// Event bus for publishing events
class EventBus {
  private subscribers: Map<string, Function[]> = new Map();

  subscribe(eventType: string, handler: Function): void {
    if (!this.subscribers.has(eventType)) {
      this.subscribers.set(eventType, []);
    }
    this.subscribers.get(eventType)!.push(handler);
  }

  subscribeToAll(handler: Function): void {
    this.subscribe("*", handler);
  }

  async publish(events: Event[]): Promise<void> {
    for (const event of events) {
      // Call specific handlers
      const handlers = this.subscribers.get(event.eventType) || [];
      await Promise.all(handlers.map(h => h(event)));

      // Call wildcard handlers
      const allHandlers = this.subscribers.get("*") || [];
      await Promise.all(allHandlers.map(h => h(event)));
    }
  }
}
```

## Saga Pattern

Manage distributed transactions across multiple services using a sequence of local transactions coordinated by a saga.

### Orchestration-Based Saga

A central orchestrator coordinates all saga participants.

```typescript
// Saga state
enum SagaStatus {
  Started = "Started",
  Completed = "Completed",
  Compensating = "Compensating",
  Compensated = "Compensated",
  Failed = "Failed"
}

interface SagaStep {
  name: string;
  action: () => Promise<void>;
  compensation: () => Promise<void>;
}

class OrderSaga {
  private sagaId: string;
  private status: SagaStatus;
  private completedSteps: string[] = [];
  private currentStep: number = 0;

  private steps: SagaStep[] = [
    {
      name: "CreateOrder",
      action: async () => await this.createOrder(),
      compensation: async () => await this.cancelOrder()
    },
    {
      name: "ReserveInventory",
      action: async () => await this.reserveInventory(),
      compensation: async () => await this.releaseInventory()
    },
    {
      name: "ProcessPayment",
      action: async () => await this.processPayment(),
      compensation: async () => await this.refundPayment()
    },
    {
      name: "ArrangeShipment",
      action: async () => await this.arrangeShipment(),
      compensation: async () => await this.cancelShipment()
    }
  ];

  constructor(
    private orderId: string,
    private customerId: string,
    private items: OrderItem[]
  ) {
    this.sagaId = generateId();
    this.status = SagaStatus.Started;
  }

  async execute(): Promise<void> {
    try {
      // Execute each step
      for (let i = 0; i < this.steps.length; i++) {
        this.currentStep = i;
        const step = this.steps[i];

        console.log(`Executing step: ${step.name}`);
        await step.action();
        this.completedSteps.push(step.name);
      }

      this.status = SagaStatus.Completed;
      console.log("Saga completed successfully");

    } catch (error) {
      console.error(`Saga failed at step ${this.currentStep}:`, error);
      await this.compensate();
    }
  }

  private async compensate(): Promise<void> {
    this.status = SagaStatus.Compensating;
    console.log("Starting compensation");

    // Compensate in reverse order
    for (let i = this.completedSteps.length - 1; i >= 0; i--) {
      const stepName = this.completedSteps[i];
      const step = this.steps.find(s => s.name === stepName);

      if (step) {
        try {
          console.log(`Compensating step: ${step.name}`);
          await step.compensation();
        } catch (error) {
          console.error(`Compensation failed for ${step.name}:`, error);
          // Log for manual intervention
        }
      }
    }

    this.status = SagaStatus.Compensated;
    console.log("Compensation completed");
  }

  // Step implementations
  private async createOrder(): Promise<void> {
    await orderService.create({
      orderId: this.orderId,
      customerId: this.customerId,
      items: this.items
    });
  }

  private async cancelOrder(): Promise<void> {
    await orderService.cancel(this.orderId);
  }

  private async reserveInventory(): Promise<void> {
    await inventoryService.reserve(this.orderId, this.items);
  }

  private async releaseInventory(): Promise<void> {
    await inventoryService.release(this.orderId);
  }

  private async processPayment(): Promise<void> {
    const total = this.calculateTotal();
    await paymentService.charge(this.customerId, total);
  }

  private async refundPayment(): Promise<void> {
    const total = this.calculateTotal();
    await paymentService.refund(this.customerId, total);
  }

  private async arrangeShipment(): Promise<void> {
    await shippingService.createShipment(this.orderId);
  }

  private async cancelShipment(): Promise<void> {
    await shippingService.cancelShipment(this.orderId);
  }

  private calculateTotal(): Money {
    return this.items.reduce(
      (sum, item) => sum.add(item.price.multiply(item.quantity)),
      new Money(0, "USD")
    );
  }
}

// Orchestrator service
class SagaOrchestrator {
  private activeSagas: Map<string, OrderSaga> = new Map();

  async startOrderSaga(
    orderId: string,
    customerId: string,
    items: OrderItem[]
  ): Promise<void> {
    const saga = new OrderSaga(orderId, customerId, items);
    this.activeSagas.set(saga.sagaId, saga);

    try {
      await saga.execute();
    } finally {
      this.activeSagas.delete(saga.sagaId);
    }
  }

  getSagaStatus(sagaId: string): SagaStatus | null {
    const saga = this.activeSagas.get(sagaId);
    return saga ? saga.status : null;
  }
}
```

### Choreography-Based Saga

Services coordinate through events without central orchestrator.

```typescript
// Event-driven saga with choreography
class OrderCreatedEvent {
  constructor(
    public orderId: string,
    public customerId: string,
    public items: OrderItem[]
  ) {}
}

class InventoryReservedEvent {
  constructor(
    public orderId: string,
    public reservationId: string
  ) {}
}

class PaymentProcessedEvent {
  constructor(
    public orderId: string,
    public paymentId: string
  ) {}
}

class ShipmentArrangedEvent {
  constructor(
    public orderId: string,
    public shipmentId: string
  ) {}
}

// Compensation events
class InventoryReservationFailedEvent {
  constructor(
    public orderId: string,
    public reason: string
  ) {}
}

class PaymentFailedEvent {
  constructor(
    public orderId: string,
    public reason: string
  ) {}
}

// Order service
class OrderService {
  constructor(private eventBus: EventBus) {
    // Subscribe to compensation events
    eventBus.subscribe("InventoryReservationFailed",
      this.handleInventoryReservationFailed.bind(this));
    eventBus.subscribe("PaymentFailed",
      this.handlePaymentFailed.bind(this));
  }

  async createOrder(order: CreateOrderRequest): Promise<void> {
    // Create order in pending state
    await this.repository.save({
      ...order,
      status: "Pending"
    });

    // Publish event to trigger next step
    await this.eventBus.publish(
      new OrderCreatedEvent(order.orderId, order.customerId, order.items)
    );
  }

  private async handleInventoryReservationFailed(
    event: InventoryReservationFailedEvent
  ): Promise<void> {
    await this.repository.updateStatus(event.orderId, "Cancelled");
    console.log(`Order ${event.orderId} cancelled: ${event.reason}`);
  }

  private async handlePaymentFailed(event: PaymentFailedEvent): Promise<void> {
    await this.repository.updateStatus(event.orderId, "PaymentFailed");
    // Trigger inventory release
    await this.eventBus.publish(
      new ReleaseInventoryCommand(event.orderId)
    );
  }
}

// Inventory service
class InventoryService {
  constructor(private eventBus: EventBus) {
    eventBus.subscribe("OrderCreated",
      this.handleOrderCreated.bind(this));
    eventBus.subscribe("ReleaseInventory",
      this.handleReleaseInventory.bind(this));
  }

  private async handleOrderCreated(event: OrderCreatedEvent): Promise<void> {
    try {
      // Reserve inventory
      const reservationId = await this.reserveItems(event.items);

      // Publish success event
      await this.eventBus.publish(
        new InventoryReservedEvent(event.orderId, reservationId)
      );
    } catch (error) {
      // Publish failure event
      await this.eventBus.publish(
        new InventoryReservationFailedEvent(
          event.orderId,
          error.message
        )
      );
    }
  }

  private async handleReleaseInventory(
    command: ReleaseInventoryCommand
  ): Promise<void> {
    await this.releaseReservation(command.orderId);
  }
}

// Payment service
class PaymentService {
  constructor(private eventBus: EventBus) {
    eventBus.subscribe("InventoryReserved",
      this.handleInventoryReserved.bind(this));
    eventBus.subscribe("RefundPayment",
      this.handleRefundPayment.bind(this));
  }

  private async handleInventoryReserved(
    event: InventoryReservedEvent
  ): Promise<void> {
    try {
      // Process payment
      const paymentId = await this.chargeCustomer(event.orderId);

      // Publish success event
      await this.eventBus.publish(
        new PaymentProcessedEvent(event.orderId, paymentId)
      );
    } catch (error) {
      // Publish failure event
      await this.eventBus.publish(
        new PaymentFailedEvent(event.orderId, error.message)
      );
    }
  }

  private async handleRefundPayment(
    command: RefundPaymentCommand
  ): Promise<void> {
    await this.refund(command.orderId);
  }
}
```

## API Gateway Pattern

Single entry point for clients, routing requests to appropriate microservices and handling cross-cutting concerns.

### Implementation

```typescript
class APIGateway {
  constructor(
    private router: Router,
    private authService: AuthService,
    private rateLimiter: RateLimiter,
    private circuitBreaker: CircuitBreaker,
    private loadBalancer: LoadBalancer
  ) {
    this.setupRoutes();
  }

  private setupRoutes(): void {
    // User service routes
    this.router.get("/api/users/:id",
      this.authenticate.bind(this),
      this.rateLimit.bind(this),
      this.getUserHandler.bind(this)
    );

    // Order service routes
    this.router.post("/api/orders",
      this.authenticate.bind(this),
      this.rateLimit.bind(this),
      this.createOrderHandler.bind(this)
    );

    // Product service routes
    this.router.get("/api/products",
      this.rateLimit.bind(this),
      this.getProductsHandler.bind(this)
    );
  }

  // Middleware: Authentication
  private async authenticate(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const token = req.headers.authorization?.replace("Bearer ", "");
      if (!token) {
        res.status(401).json({ error: "Unauthorized" });
        return;
      }

      const user = await this.authService.validateToken(token);
      req.user = user;
      next();
    } catch (error) {
      res.status(401).json({ error: "Invalid token" });
    }
  }

  // Middleware: Rate limiting
  private async rateLimit(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    const clientId = req.ip || req.user?.id;

    if (await this.rateLimiter.isAllowed(clientId)) {
      next();
    } else {
      res.status(429).json({ error: "Too many requests" });
    }
  }

  // Handler: Get user
  private async getUserHandler(req: Request, res: Response): Promise<void> {
    try {
      const userId = req.params.id;

      // Call user service with circuit breaker
      const user = await this.circuitBreaker.execute(
        "user-service",
        async () => {
          const instance = this.loadBalancer.getInstance("user-service");
          return await instance.getUser(userId);
        }
      );

      res.json(user);
    } catch (error) {
      res.status(500).json({ error: "Internal server error" });
    }
  }

  // Handler: Create order
  private async createOrderHandler(
    req: Request,
    res: Response
  ): Promise<void> {
    try {
      // Aggregate data from multiple services
      const [user, products, inventory] = await Promise.all([
        this.callUserService(req.user.id),
        this.callProductService(req.body.items),
        this.callInventoryService(req.body.items)
      ]);

      // Call order service
      const order = await this.callOrderService({
        user,
        items: req.body.items,
        inventory
      });

      res.status(201).json(order);
    } catch (error) {
      res.status(500).json({ error: "Internal server error" });
    }
  }

  private async callUserService(userId: string): Promise<User> {
    return await this.circuitBreaker.execute("user-service", async () => {
      const instance = this.loadBalancer.getInstance("user-service");
      return await instance.getUser(userId);
    });
  }

  private async callProductService(items: any[]): Promise<Product[]> {
    return await this.circuitBreaker.execute("product-service", async () => {
      const instance = this.loadBalancer.getInstance("product-service");
      return await instance.getProducts(items.map(i => i.productId));
    });
  }
}

// Rate limiter implementation
class RateLimiter {
  private requests: Map<string, number[]> = new Map();

  constructor(
    private maxRequests: number = 100,
    private windowMs: number = 60000 // 1 minute
  ) {}

  async isAllowed(clientId: string): Promise<boolean> {
    const now = Date.now();
    const windowStart = now - this.windowMs;

    // Get existing requests
    const clientRequests = this.requests.get(clientId) || [];

    // Filter out old requests
    const recentRequests = clientRequests.filter(t => t > windowStart);

    // Check if under limit
    if (recentRequests.length < this.maxRequests) {
      recentRequests.push(now);
      this.requests.set(clientId, recentRequests);
      return true;
    }

    return false;
  }
}

// Load balancer
class LoadBalancer {
  private services: Map<string, ServiceInstance[]> = new Map();
  private currentIndex: Map<string, number> = new Map();

  registerService(name: string, instance: ServiceInstance): void {
    if (!this.services.has(name)) {
      this.services.set(name, []);
      this.currentIndex.set(name, 0);
    }
    this.services.get(name)!.push(instance);
  }

  getInstance(serviceName: string): ServiceInstance {
    const instances = this.services.get(serviceName);
    if (!instances || instances.length === 0) {
      throw new Error(`No instances available for ${serviceName}`);
    }

    // Round-robin selection
    const index = this.currentIndex.get(serviceName)!;
    const instance = instances[index];

    // Update index for next call
    this.currentIndex.set(
      serviceName,
      (index + 1) % instances.length
    );

    return instance;
  }
}
```

### Backend for Frontend (BFF) Pattern

```typescript
// Separate BFFs for different clients
class WebBFF {
  constructor(
    private userService: UserService,
    private productService: ProductService,
    private orderService: OrderService
  ) {}

  // Optimized for web client needs
  async getHomePage(userId: string): Promise<WebHomePageData> {
    const [user, recommendations, recentOrders] = await Promise.all([
      this.userService.getUser(userId),
      this.productService.getRecommendations(userId, 10),
      this.orderService.getRecentOrders(userId, 5)
    ]);

    return {
      user: {
        name: user.name,
        email: user.email,
        avatar: user.avatarUrl
      },
      recommendations: recommendations.map(p => ({
        id: p.id,
        name: p.name,
        price: p.price,
        imageUrl: p.images[0], // Full images for web
        rating: p.averageRating
      })),
      recentOrders: recentOrders.map(o => ({
        orderId: o.id,
        date: o.createdAt,
        total: o.totalAmount,
        status: o.status,
        itemCount: o.items.length
      }))
    };
  }
}

class MobileBFF {
  constructor(
    private userService: UserService,
    private productService: ProductService,
    private orderService: OrderService
  ) {}

  // Optimized for mobile client needs
  async getHomePage(userId: string): Promise<MobileHomePageData> {
    const [user, recommendations, recentOrders] = await Promise.all([
      this.userService.getUser(userId),
      this.productService.getRecommendations(userId, 5), // Fewer items
      this.orderService.getRecentOrders(userId, 3)
    ]);

    return {
      user: {
        name: user.name,
        avatar: user.avatarThumbnailUrl // Smaller images for mobile
      },
      recommendations: recommendations.map(p => ({
        id: p.id,
        name: p.name,
        price: p.price,
        thumbnail: p.thumbnails.small, // Optimized image size
        rating: Math.round(p.averageRating) // Simplified rating
      })),
      recentOrders: recentOrders.map(o => ({
        id: o.id,
        date: o.createdAt.toISOString(),
        total: o.totalAmount,
        status: o.status
      }))
    };
  }
}
```

## Service Mesh Pattern

Infrastructure layer for service-to-service communication providing observability, traffic management, and security.

### Key Features

```typescript
// Service mesh configuration example (Istio)
const serviceMeshConfig = {
  // Traffic management
  virtualService: {
    name: "product-service",
    hosts: ["product-service"],
    http: [
      {
        match: [{ uri: { prefix: "/api/v1" } }],
        route: [
          {
            destination: {
              host: "product-service",
              subset: "v1"
            },
            weight: 90
          },
          {
            destination: {
              host: "product-service",
              subset: "v2"
            },
            weight: 10 // Canary deployment
          }
        ],
        retries: {
          attempts: 3,
          perTryTimeout: "2s"
        },
        timeout: "10s"
      }
    ]
  },

  // Destination rules
  destinationRule: {
    name: "product-service",
    host: "product-service",
    trafficPolicy: {
      connectionPool: {
        tcp: {
          maxConnections: 100
        },
        http: {
          http1MaxPendingRequests: 50,
          http2MaxRequests: 100,
          maxRequestsPerConnection: 2
        }
      },
      loadBalancer: {
        simple: "ROUND_ROBIN"
      },
      outlierDetection: {
        consecutive5xxErrors: 5,
        interval: "30s",
        baseEjectionTime: "30s",
        maxEjectionPercent: 50
      }
    },
    subsets: [
      {
        name: "v1",
        labels: { version: "v1" }
      },
      {
        name: "v2",
        labels: { version: "v2" }
      }
    ]
  },

  // Circuit breaker
  circuitBreaker: {
    consecutiveErrors: 5,
    interval: "30s",
    baseEjectionTime: "30s",
    maxEjectionPercent: 50
  }
};
```

## Resilience Patterns

### Circuit Breaker

Prevent cascading failures by stopping requests to failing services.

```typescript
enum CircuitState {
  Closed = "Closed",     // Normal operation
  Open = "Open",         // Blocking requests
  HalfOpen = "HalfOpen"  // Testing if service recovered
}

class CircuitBreaker {
  private state: CircuitState = CircuitState.Closed;
  private failureCount: number = 0;
  private successCount: number = 0;
  private lastFailureTime: number = 0;

  constructor(
    private failureThreshold: number = 5,
    private successThreshold: number = 2,
    private timeout: number = 60000 // 1 minute
  ) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.Open) {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = CircuitState.HalfOpen;
        this.successCount = 0;
      } else {
        throw new Error("Circuit breaker is OPEN");
      }
    }

    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess(): void {
    this.failureCount = 0;

    if (this.state === CircuitState.HalfOpen) {
      this.successCount++;
      if (this.successCount >= this.successThreshold) {
        this.state = CircuitState.Closed;
        this.successCount = 0;
      }
    }
  }

  private onFailure(): void {
    this.failureCount++;
    this.lastFailureTime = Date.now();

    if (this.failureCount >= this.failureThreshold) {
      this.state = CircuitState.Open;
    }
  }

  getState(): CircuitState {
    return this.state;
  }
}
```

### Retry Pattern

Automatically retry failed operations with exponential backoff.

```typescript
class RetryPolicy {
  constructor(
    private maxRetries: number = 3,
    private initialDelayMs: number = 100,
    private maxDelayMs: number = 5000,
    private backoffMultiplier: number = 2
  ) {}

  async execute<T>(
    operation: () => Promise<T>,
    isRetryable: (error: Error) => boolean = () => true
  ): Promise<T> {
    let lastError: Error;

    for (let attempt = 0; attempt <= this.maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error as Error;

        if (attempt === this.maxRetries || !isRetryable(lastError)) {
          throw lastError;
        }

        const delay = this.calculateDelay(attempt);
        await this.sleep(delay);
      }
    }

    throw lastError!;
  }

  private calculateDelay(attempt: number): number {
    const delay = this.initialDelayMs * Math.pow(this.backoffMultiplier, attempt);
    return Math.min(delay, this.maxDelayMs);
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage
const retryPolicy = new RetryPolicy(3, 100, 5000, 2);

await retryPolicy.execute(
  async () => await apiClient.get("/users/123"),
  (error) => error.statusCode >= 500 // Only retry server errors
);
```

### Bulkhead Pattern

Isolate resources to prevent failures from affecting entire system.

```typescript
class Bulkhead {
  private activeRequests: number = 0;
  private queue: Array<{
    resolve: (value: any) => void;
    reject: (error: any) => void;
    operation: () => Promise<any>;
  }> = [];

  constructor(
    private maxConcurrent: number = 10,
    private maxQueueSize: number = 100
  ) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.activeRequests < this.maxConcurrent) {
      return await this.executeOperation(operation);
    }

    if (this.queue.length >= this.maxQueueSize) {
      throw new Error("Bulkhead queue full");
    }

    return new Promise((resolve, reject) => {
      this.queue.push({ resolve, reject, operation });
    });
  }

  private async executeOperation<T>(
    operation: () => Promise<T>
  ): Promise<T> {
    this.activeRequests++;

    try {
      const result = await operation();
      this.processQueue();
      return result;
    } catch (error) {
      this.processQueue();
      throw error;
    } finally {
      this.activeRequests--;
    }
  }

  private processQueue(): void {
    if (this.queue.length > 0 &&
        this.activeRequests < this.maxConcurrent) {
      const { resolve, reject, operation } = this.queue.shift()!;

      this.executeOperation(operation)
        .then(resolve)
        .catch(reject);
    }
  }
}
```

### Timeout Pattern

Prevent indefinite waits by setting time limits.

```typescript
class TimeoutPolicy {
  constructor(private timeoutMs: number = 30000) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    return Promise.race([
      operation(),
      this.timeout()
    ]);
  }

  private timeout(): Promise<never> {
    return new Promise((_, reject) => {
      setTimeout(() => {
        reject(new Error(`Operation timed out after ${this.timeoutMs}ms`));
      }, this.timeoutMs);
    });
  }
}
```

### Fallback Pattern

Provide alternative response when operation fails.

```typescript
class FallbackPolicy<T> {
  constructor(private fallbackFn: () => Promise<T>) {}

  async execute(operation: () => Promise<T>): Promise<T> {
    try {
      return await operation();
    } catch (error) {
      console.warn("Operation failed, using fallback:", error);
      return await this.fallbackFn();
    }
  }
}

// Usage
const getUserWithFallback = new FallbackPolicy(async () => ({
  id: "default",
  name: "Guest User",
  email: "guest@example.com"
}));

const user = await getUserWithFallback.execute(
  async () => await userService.getUser(userId)
);
```

### Combined Resilience Strategy

```typescript
class ResilientClient {
  private circuitBreaker: CircuitBreaker;
  private retryPolicy: RetryPolicy;
  private timeoutPolicy: TimeoutPolicy;
  private fallbackPolicy: FallbackPolicy<any>;
  private bulkhead: Bulkhead;

  constructor() {
    this.circuitBreaker = new CircuitBreaker(5, 2, 60000);
    this.retryPolicy = new RetryPolicy(3, 100, 5000, 2);
    this.timeoutPolicy = new TimeoutPolicy(10000);
    this.fallbackPolicy = new FallbackPolicy(async () => null);
    this.bulkhead = new Bulkhead(10, 100);
  }

  async call<T>(
    operation: () => Promise<T>,
    options?: {
      timeout?: number;
      retries?: number;
      fallback?: () => Promise<T>;
    }
  ): Promise<T> {
    const timeoutPolicy = options?.timeout
      ? new TimeoutPolicy(options.timeout)
      : this.timeoutPolicy;

    const fallbackPolicy = options?.fallback
      ? new FallbackPolicy(options.fallback)
      : this.fallbackPolicy;

    return await fallbackPolicy.execute(async () => {
      return await this.bulkhead.execute(async () => {
        return await this.circuitBreaker.execute(async () => {
          return await this.retryPolicy.execute(async () => {
            return await timeoutPolicy.execute(operation);
          });
        });
      });
    });
  }
}
```

## Scalability Patterns

### Horizontal Scaling (Scale Out)

Add more instances to handle increased load.

```typescript
// Load balancer for horizontal scaling
class RoundRobinLoadBalancer {
  private instances: string[] = [];
  private currentIndex: number = 0;

  addInstance(url: string): void {
    this.instances.push(url);
  }

  removeInstance(url: string): void {
    this.instances = this.instances.filter(i => i !== url);
  }

  getNextInstance(): string {
    if (this.instances.length === 0) {
      throw new Error("No instances available");
    }

    const instance = this.instances[this.currentIndex];
    this.currentIndex = (this.currentIndex + 1) % this.instances.length;
    return instance;
  }
}

// Auto-scaling based on metrics
class AutoScaler {
  constructor(
    private minInstances: number = 2,
    private maxInstances: number = 10,
    private targetCPU: number = 70
  ) {}

  async scale(currentInstances: number, currentCPU: number): Promise<number> {
    if (currentCPU > this.targetCPU) {
      // Scale up
      const desiredInstances = Math.ceil(
        currentInstances * (currentCPU / this.targetCPU)
      );
      return Math.min(desiredInstances, this.maxInstances);
    } else if (currentCPU < this.targetCPU * 0.5) {
      // Scale down
      const desiredInstances = Math.floor(
        currentInstances * (currentCPU / this.targetCPU)
      );
      return Math.max(desiredInstances, this.minInstances);
    }

    return currentInstances;
  }
}
```

### Caching Strategies

```typescript
// Cache-aside pattern
class CacheAsideRepository {
  constructor(
    private cache: Cache,
    private database: Database,
    private ttl: number = 3600
  ) {}

  async get(id: string): Promise<any> {
    // Try cache first
    const cached = await this.cache.get(id);
    if (cached) {
      return cached;
    }

    // Cache miss - get from database
    const data = await this.database.findById(id);
    if (data) {
      await this.cache.set(id, data, this.ttl);
    }

    return data;
  }

  async update(id: string, data: any): Promise<void> {
    // Update database
    await this.database.update(id, data);

    // Invalidate cache
    await this.cache.delete(id);
  }
}

// Write-through cache
class WriteThroughCache {
  constructor(
    private cache: Cache,
    private database: Database
  ) {}

  async write(id: string, data: any): Promise<void> {
    // Write to both cache and database
    await Promise.all([
      this.cache.set(id, data),
      this.database.save(id, data)
    ]);
  }
}

// Write-behind cache
class WriteBehindCache {
  private writeQueue: Map<string, any> = new Map();

  constructor(
    private cache: Cache,
    private database: Database,
    private flushInterval: number = 5000
  ) {
    this.startFlushInterval();
  }

  async write(id: string, data: any): Promise<void> {
    // Write to cache immediately
    await this.cache.set(id, data);

    // Queue for database write
    this.writeQueue.set(id, data);
  }

  private startFlushInterval(): void {
    setInterval(async () => {
      await this.flush();
    }, this.flushInterval);
  }

  private async flush(): Promise<void> {
    const entries = Array.from(this.writeQueue.entries());
    this.writeQueue.clear();

    await Promise.all(
      entries.map(([id, data]) => this.database.save(id, data))
    );
  }
}
```

### Database Sharding

```typescript
// Shard key-based routing
class ShardRouter {
  private shards: Map<number, Database> = new Map();
  private totalShards: number;

  constructor(shards: Database[]) {
    this.totalShards = shards.length;
    shards.forEach((shard, index) => {
      this.shards.set(index, shard);
    });
  }

  private getShardIndex(key: string): number {
    // Hash-based sharding
    const hash = this.hashCode(key);
    return Math.abs(hash) % this.totalShards;
  }

  private hashCode(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32-bit integer
    }
    return hash;
  }

  getShard(key: string): Database {
    const index = this.getShardIndex(key);
    return this.shards.get(index)!;
  }

  async save(key: string, data: any): Promise<void> {
    const shard = this.getShard(key);
    await shard.save(key, data);
  }

  async find(key: string): Promise<any> {
    const shard = this.getShard(key);
    return await shard.findById(key);
  }
}
```

## Best Practices

### Architecture Design

1. **Start with the domain**: Understand business requirements before choosing patterns
2. **Keep it simple**: Don't over-engineer; add complexity only when needed
3. **Design for failure**: Assume services will fail; build resilience
4. **Loose coupling**: Services should be independent and deployable separately
5. **High cohesion**: Group related functionality together
6. **API-first design**: Define contracts before implementation
7. **Versioning strategy**: Plan for API evolution from the start
8. **Observability**: Build in logging, metrics, and tracing from day one

### Domain-Driven Design

1. **Collaborate with domain experts**: Build shared understanding
2. **Use ubiquitous language**: Consistent terminology everywhere
3. **Model bounded contexts**: Clear boundaries prevent model confusion
4. **Small aggregates**: Better performance and clearer boundaries
5. **Reference by ID**: Aggregates reference others by identity
6. **Protect invariants**: Aggregate roots enforce all business rules
7. **Domain events**: Capture important business occurrences
8. **Repository per aggregate**: One repository per aggregate root

### Event Sourcing & CQRS

1. **Event naming**: Use past tense, business-meaningful names
2. **Event immutability**: Never modify published events
3. **Event versioning**: Plan for event schema evolution
4. **Snapshots**: Use for long event streams
5. **Idempotent handlers**: Events may be processed multiple times
6. **Separate concerns**: Different models for read and write
7. **Eventual consistency**: Accept and communicate delay
8. **Monitoring**: Track projection lag and event processing

### Microservices

1. **Service size**: Small enough to understand, large enough to provide value
2. **Data ownership**: Each service owns its data
3. **Asynchronous communication**: Prefer events over synchronous calls
4. **Service discovery**: Dynamic service location
5. **Configuration management**: Centralized, environment-specific config
6. **Deployment independence**: Services deploy without coordinating
7. **Failure isolation**: Circuit breakers and bulkheads
8. **Distributed tracing**: Correlation IDs across service calls

### Performance & Scalability

1. **Measure first**: Profile before optimizing
2. **Cache strategically**: Right layer, right data, right TTL
3. **Async processing**: Move slow operations to background
4. **Connection pooling**: Reuse database/HTTP connections
5. **Pagination**: Never return unbounded result sets
6. **Compression**: Reduce network transfer size
7. **CDN usage**: Serve static assets from edge locations
8. **Database indexes**: Index query patterns, not all columns

### Security

1. **Defense in depth**: Multiple security layers
2. **Least privilege**: Minimal permissions necessary
3. **Encrypt in transit**: TLS for all network communication
4. **Encrypt at rest**: Sensitive data encrypted in storage
5. **Input validation**: Validate and sanitize all inputs
6. **Authentication**: Verify identity (JWT, OAuth)
7. **Authorization**: Verify permissions (RBAC, ABAC)
8. **Audit logging**: Track security-relevant events

### Testing

1. **Test pyramid**: Many unit tests, fewer integration, few E2E
2. **Test behavior**: Focus on business logic, not implementation
3. **Contract testing**: Verify API contracts between services
4. **Chaos engineering**: Test failure scenarios
5. **Performance testing**: Load test before production
6. **Security testing**: Automated vulnerability scanning
7. **Smoke tests**: Quick validation after deployment
8. **Canary deployments**: Gradual rollout to detect issues

### Monitoring & Observability

1. **Structured logging**: JSON logs with context
2. **Metrics collection**: RED metrics (Rate, Errors, Duration)
3. **Distributed tracing**: Request flow across services
4. **Health checks**: Liveness and readiness endpoints
5. **Alerting**: Alert on symptoms, not causes
6. **Dashboards**: Key metrics visible at a glance
7. **SLO/SLA**: Define and track service levels
8. **Incident response**: Runbooks for common issues

## Resources

### Books
- "Domain-Driven Design" by Eric Evans
- "Implementing Domain-Driven Design" by Vaughn Vernon
- "Microservices Patterns" by Chris Richardson
- "Building Microservices" by Sam Newman
- "Designing Data-Intensive Applications" by Martin Kleppmann

### Online Resources
- https://microservices.io/patterns - Microservices pattern catalog
- https://martinfowler.com - Architecture articles and patterns
- https://learn.microsoft.com/en-us/azure/architecture - Azure Architecture Center
- https://aws.amazon.com/architecture - AWS Architecture resources
- https://cloud.google.com/architecture - Google Cloud Architecture

---

**Skill Version**: 1.0.0
**Last Updated**: October 2025
**Skill Category**: Enterprise Architecture, System Design, Distributed Systems
