---
name: idempotency
description: Idempotency concepts, patterns for APIs REST, database operations, and implementation strategies. Use when implementing idempotent operations, handling retries, or when ensuring operations can be safely repeated in any language.
---

# Idempotency Best Practices

This skill provides idempotency concepts, patterns, and implementation strategies applicable across multiple programming languages and frameworks.

## Core Concepts

### What is Idempotency?

An operation is **idempotent** if performing it multiple times produces the same result as performing it once.

**Mathematical Example:**
- `f(f(x)) = f(x)`
- Example: `abs(abs(-5)) = abs(-5) = 5`

**Programming Example:**
- GET request: Always returns same result
- DELETE request: Deleting twice has same effect as once
- PUT request: Replacing resource multiple times has same effect

## Why Idempotency Matters

1. **Retry Safety**: Network failures can cause retries - operations must handle this
2. **Distributed Systems**: Multiple services may attempt same operation
3. **User Experience**: Users may click "Submit" multiple times
4. **Reliability**: Ensures system consistency under failures

## HTTP Methods and Idempotency

### Idempotent Methods

**GET**: Always safe and idempotent
- Reading data doesn't change state
- Can be called multiple times safely

**PUT**: Idempotent (when used correctly)
- Replacing resource with same data multiple times has same effect
- Must include complete resource representation

**DELETE**: Idempotent
- Deleting already-deleted resource has same effect (no-op)

### Non-Idempotent Methods

**POST**: Not idempotent
- Creates new resources
- Each call may create different resource

**PATCH**: Not idempotent (usually)
- Partial updates may have different effects on repeated calls

## API Idempotency Patterns

### Pattern 1: Idempotency Keys

Client provides unique key for each operation. Server uses key to detect duplicates.

```typescript
// Client sends request with idempotency key
POST /api/orders
Headers:
  Idempotency-Key: abc123-def456-ghi789

// Server implementation
const idempotencyKey = req.headers['idempotency-key'];

if (idempotencyKey) {
  // Check if we've seen this key before
  const cachedResponse = await cache.get(`idempotency:${idempotencyKey}`);
  if (cachedResponse) {
    return res.status(cachedResponse.status).json(cachedResponse.body);
  }
  
  // Process request
  const order = await createOrder(req.body);
  
  // Cache response
  await cache.set(`idempotency:${idempotencyKey}`, {
    status: 201,
    body: order,
  }, { ttl: 24 * 60 * 60 }); // 24 hours
  
  return res.status(201).json(order);
}
```

### Pattern 2: Request Deduplication

```typescript
class IdempotencyHandler {
  async handleRequest(
    idempotencyKey: string,
    handler: () => Promise<any>
  ): Promise<any> {
    // Check if request already processed
    const existing = await this.getResponse(idempotencyKey);
    if (existing) {
      return existing;
    }
    
    // Process request
    try {
      const result = await handler();
      await this.storeResponse(idempotencyKey, result);
      return result;
    } catch (error) {
      // Don't cache errors - allow retry
      throw error;
    }
  }
  
  private async getResponse(key: string) {
    return cache.get(`idempotency:${key}`);
  }
  
  private async storeResponse(key: string, response: any) {
    await cache.set(`idempotency:${key}`, response, { ttl: 86400 });
  }
}
```

## Database Idempotency Patterns

### Pattern 1: UPSERT Operations

```sql
-- PostgreSQL
INSERT INTO orders (id, user_id, total, status)
VALUES ($1, $2, $3, 'pending')
ON CONFLICT (id) DO UPDATE SET
  total = EXCLUDED.total,
  updated_at = NOW()
RETURNING *;
```

```typescript
// MongoDB
await db.orders.findOneAndUpdate(
  { id: orderId },
  { $set: { total, status: 'pending' } },
  { upsert: true, new: true }
);
```

### Pattern 2: Conditional Updates

```typescript
// Only update if version matches (optimistic locking)
async function updateOrder(orderId: string, data: Partial<Order>, expectedVersion: number) {
  const result = await db.orders.updateOne(
    { 
      id: orderId,
      version: expectedVersion, // Only update if version matches
    },
    {
      $set: { ...data, version: expectedVersion + 1 },
    }
  );
  
  if (result.matchedCount === 0) {
    throw new Error('Order was modified by another operation');
  }
  
  return result;
}
```

### Pattern 3: Unique Constraints

```sql
-- Prevent duplicate operations using unique constraint
CREATE UNIQUE INDEX idx_order_user_unique 
ON orders(user_id, order_number);

-- Insert will fail if duplicate exists
INSERT INTO orders (user_id, order_number, total)
VALUES ($1, $2, $3);
-- If duplicate, operation is idempotent (no change)
```

## Implementation Strategies

### Strategy 1: Versioning

Use version numbers or timestamps to detect changes.

```typescript
interface Resource {
  id: string;
  version: number;
  data: any;
}

async function updateResource(id: string, data: any, expectedVersion: number) {
  const resource = await getResource(id);
  
  if (resource.version !== expectedVersion) {
    throw new Error('Resource was modified');
  }
  
  return updateResourceWithVersion(id, data, expectedVersion + 1);
}
```

### Strategy 2: State Machines

Use state machines to ensure operations only happen in valid states.

```typescript
enum OrderStatus {
  PENDING = 'pending',
  PROCESSING = 'processing',
  COMPLETED = 'completed',
  CANCELLED = 'cancelled',
}

async function processOrder(orderId: string) {
  const order = await getOrder(orderId);
  
  // Only process if in pending state
  if (order.status !== OrderStatus.PENDING) {
    return order; // Idempotent: return current state
  }
  
  // Update to processing
  await updateOrderStatus(orderId, OrderStatus.PROCESSING);
  
  try {
    await fulfillOrder(order);
    await updateOrderStatus(orderId, OrderStatus.COMPLETED);
  } catch (error) {
    await updateOrderStatus(orderId, OrderStatus.CANCELLED);
    throw error;
  }
}
```

### Strategy 3: Tokens/Request IDs

Generate unique tokens for each operation attempt.

```typescript
async function createPayment(paymentData: PaymentData, requestId: string) {
  // Check if this request was already processed
  const existing = await db.payments.findOne({ requestId });
  if (existing) {
    return existing; // Return existing payment (idempotent)
  }
  
  // Create new payment with request ID
  const payment = await db.payments.create({
    ...paymentData,
    requestId,
    status: 'pending',
  });
  
  return payment;
}
```

## Best Practices

### ✅ DO:

1. **Always Return Same Result**: Same input → same output
2. **Use Idempotency Keys**: For POST/PATCH operations
3. **Check Before Creating**: Verify resource doesn't exist
4. **Use UPSERT**: For create-or-update operations
5. **Log Idempotent Operations**: Track for debugging
6. **Handle Retries**: Design for automatic retries
7. **Version Resources**: Use versioning for updates

### ❌ DON'T:

1. **Increment Counters**: Without checking current value
2. **Create Multiple Resources**: From same operation
3. **Side Effects**: Uncontrolled side effects
4. **Ignore Duplicates**: Always check for existing operations
5. **Cache Errors**: Don't cache error responses as idempotent

## Common Patterns

### Pattern: Idempotent Create

```typescript
async function createOrder(orderData: OrderData, idempotencyKey: string) {
  // Check if order with this key already exists
  const existing = await db.orders.findOne({ idempotencyKey });
  if (existing) {
    return existing; // Return existing order
  }
  
  // Create new order
  const order = await db.orders.create({
    ...orderData,
    idempotencyKey,
  });
  
  return order;
}
```

### Pattern: Idempotent Update

```typescript
async function updateOrder(orderId: string, updates: Partial<Order>) {
  // Get current order
  const order = await db.orders.findById(orderId);
  
  // Check if update would change anything
  const hasChanges = Object.keys(updates).some(
    key => order[key] !== updates[key]
  );
  
  if (!hasChanges) {
    return order; // No changes, return current state (idempotent)
  }
  
  // Apply updates
  return db.orders.update(orderId, updates);
}
```

## Key Principles

1. **Same Input, Same Output**: Idempotent operations always return same result
2. **Safe to Retry**: Operations can be safely retried
3. **Check Before Act**: Verify state before performing operation
4. **Use Idempotency Keys**: For non-idempotent HTTP methods
5. **Version Resources**: Track changes to detect conflicts
6. **State Machines**: Use states to control operation flow
7. **Log Operations**: Track idempotent operations for debugging
