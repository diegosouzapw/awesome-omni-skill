---
name: backend-development
description: Use this skill when the user asks about "async patterns", "Promise.all", "parallel execution", "Firebase functions", "webhook handlers", "cron jobs", "cold start", "timeout", "pubsub", "background processing", "message queue", "fan-out", "event-driven", or any Node.js/Firebase backend patterns. Provides async/await patterns, function configuration, Pub/Sub messaging, and scalable background processing.
---

# Node.js & Firebase Functions Patterns

## Async/Await Patterns

### Parallel Execution

```javascript
// ❌ BAD: Sequential (3000ms)
const customers = await getCustomers();
const settings = await getSettings();
const tiers = await getTiers();

// ✅ GOOD: Parallel (1000ms)
const [customers, settings, tiers] = await Promise.all([
  getCustomers(),
  getSettings(),
  getTiers()
]);
```

### Avoid Await in Loops

```javascript
// ❌ BAD: Sequential loop
for (const customer of customers) {
  await updateCustomer(customer);
}

// ✅ GOOD: Parallel
await Promise.all(customers.map(c => updateCustomer(c)));

// ✅ BETTER: Chunked for rate limits
async function processInChunks(items, fn, chunkSize = 10) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    await Promise.all(chunk.map(fn));
  }
}
```

---

## Firebase Functions Configuration

### Right-Sizing Guide

| Function Type | Memory | Timeout |
|---------------|--------|---------|
| Simple API handler | 256MB | 60s |
| Webhook handler | 256-512MB | 60s |
| Data sync (small) | 512MB | 120s |
| Data sync (large) | 1GB | 540s |
| Bulk operations | 1GB | 540s |
| High-traffic API | 512MB | 60s |

---

## Webhook Handlers (CRITICAL)

**Shopify requires response within 5 seconds or it will retry.**

```javascript
// ❌ BAD: Heavy processing (may timeout)
app.post('/webhooks/orders/create', async (req, res) => {
  await calculatePoints(req.body);
  await updateCustomer(req.body);
  res.status(200).send('OK');
});

// ✅ GOOD: Queue and respond fast
app.post('/webhooks/orders/create', async (req, res) => {
  if (!verifyHmac(req)) {
    return res.status(401).send('Unauthorized');
  }

  await webhookQueueRef.add({
    type: 'orders/create',
    payload: req.body
  });

  res.status(200).send('OK');
});
```

---

## Background Processing

| Method | Use Case | Volume | Latency |
|--------|----------|--------|---------|
| Firestore trigger | Simple queuing | Low-Medium | Real-time |
| Cloud Tasks | Delayed processing, rate limits | Medium | Configurable |
| **Pub/Sub** | High volume, fan-out, scaling | **High** | **Real-time** |

See `cloud-tasks-queue` skill for delayed/scheduled patterns.

---

## Pub/Sub Patterns (Scalable Background Processing)

### When to Use Pub/Sub

| Scenario | Recommended |
|----------|-------------|
| High-volume webhooks (100+ per minute) | ✅ Pub/Sub |
| Fan-out to multiple consumers | ✅ Pub/Sub |
| Decoupled microservices | ✅ Pub/Sub |
| At-least-once delivery needed | ✅ Pub/Sub |
| Delayed execution (specific time) | ❌ Use Cloud Tasks |
| Rate-limited API calls | ❌ Use Cloud Tasks |

### Publishing Messages

```javascript
import {PubSub} from '@google-cloud/pubsub';

const pubsub = new PubSub();

// Publish single message
async function publishMessage(topicName, data) {
  const topic = pubsub.topic(topicName);
  const messageBuffer = Buffer.from(JSON.stringify(data));

  const messageId = await topic.publishMessage({data: messageBuffer});
  return messageId;
}

// Publish with attributes (for filtering)
async function publishWithAttributes(topicName, data, attributes) {
  const topic = pubsub.topic(topicName);
  const messageBuffer = Buffer.from(JSON.stringify(data));

  const messageId = await topic.publishMessage({
    data: messageBuffer,
    attributes: {
      shopId: data.shopId,
      eventType: attributes.eventType
    }
  });
  return messageId;
}

// Batch publish for high volume
async function publishBatch(topicName, items) {
  const topic = pubsub.topic(topicName, {
    batching: {
      maxMessages: 100,
      maxMilliseconds: 100
    }
  });

  const promises = items.map(item =>
    topic.publishMessage({data: Buffer.from(JSON.stringify(item))})
  );

  return Promise.all(promises);
}
```

### Subscriber Functions (Firebase)

```javascript
import * as functions from 'firebase-functions';

// Basic subscriber
exports.processOrderEvents = functions.pubsub
  .topic('order-events')
  .onPublish(async (message, context) => {
    const data = JSON.parse(Buffer.from(message.data, 'base64').toString());

    try {
      await processOrder(data);
    } catch (error) {
      console.error('Processing failed:', error);
      throw error; // Pub/Sub will retry
    }
  });

// With message attributes filtering (server-side)
exports.processVipOrders = functions.pubsub
  .topic('order-events')
  .onPublish(async (message, context) => {
    const attributes = message.attributes;

    // Skip non-VIP orders early
    if (attributes.tierLevel !== 'vip') {
      return;
    }

    const data = JSON.parse(Buffer.from(message.data, 'base64').toString());
    await processVipOrder(data);
  });
```

### Fan-Out Pattern (One Event → Multiple Actions)

```javascript
// Webhook receives order → Publish once → Multiple subscribers process
app.post('/webhooks/orders/create', async (req, res) => {
  if (!verifyHmac(req)) {
    return res.status(401).send('Unauthorized');
  }

  // Single publish, multiple consumers handle different aspects
  await pubsub.topic('order-created').publishMessage({
    data: Buffer.from(JSON.stringify(req.body)),
    attributes: {
      shopId: req.body.shopId,
      orderValue: String(req.body.total_price)
    }
  });

  res.status(200).send('OK');
});

// Consumer 1: Calculate points
exports.calculatePoints = functions.pubsub
  .topic('order-created')
  .onPublish(async (message) => {
    const order = JSON.parse(Buffer.from(message.data, 'base64').toString());
    await pointsService.calculateAndAward(order);
  });

// Consumer 2: Update VIP tier
exports.updateTier = functions.pubsub
  .topic('order-created')
  .onPublish(async (message) => {
    const order = JSON.parse(Buffer.from(message.data, 'base64').toString());
    await tierService.recalculate(order.customerId);
  });

// Consumer 3: Send notification
exports.sendNotification = functions.pubsub
  .topic('order-created')
  .onPublish(async (message) => {
    const order = JSON.parse(Buffer.from(message.data, 'base64').toString());
    await notificationService.sendPointsEarned(order);
  });
```

### Error Handling & Dead Letter Queue

```javascript
// Configure subscription with DLQ in GCP Console or Terraform
// subscription: order-events-sub
// dead_letter_topic: order-events-dlq
// max_delivery_attempts: 5

// Process dead letter messages
exports.handleDeadLetters = functions.pubsub
  .topic('order-events-dlq')
  .onPublish(async (message) => {
    const data = JSON.parse(Buffer.from(message.data, 'base64').toString());

    // Log for investigation
    console.error('Dead letter received:', {
      data,
      attributes: message.attributes,
      deliveryAttempt: message.attributes?.deliveryAttempt
    });

    // Store for manual review
    await firestore.collection('failedEvents').add({
      data,
      attributes: message.attributes,
      timestamp: new Date()
    });
  });
```

### Ordering Messages (When Order Matters)

```javascript
// Use ordering key for messages that must be processed in sequence
async function publishOrderedMessage(topicName, data, orderingKey) {
  const topic = pubsub.topic(topicName, {
    messageOrdering: true
  });

  await topic.publishMessage({
    data: Buffer.from(JSON.stringify(data)),
    orderingKey // e.g., customerId - ensures all messages for same customer processed in order
  });
}
```

### Decision: Pub/Sub vs Cloud Tasks

| Need | Solution |
|------|----------|
| Process NOW, scale automatically | Pub/Sub |
| Process LATER (delay) | Cloud Tasks |
| Multiple consumers for same event | Pub/Sub (fan-out) |
| Rate-limited external API | Cloud Tasks |
| At-least-once delivery | Both work |
| Exactly-once processing | Implement idempotency |

---

## Cron Jobs (Scheduled Functions)

```javascript
import * as functions from 'firebase-functions';

// Daily cleanup at midnight UTC
exports.dailyCleanup = functions.pubsub
  .schedule('0 0 * * *')
  .timeZone('UTC')
  .onRun(async (context) => {
    await cleanupExpiredRewards();
    await archiveOldActivities();
  });

// Every 5 minutes - sync pending updates
exports.syncPendingUpdates = functions.pubsub
  .schedule('every 5 minutes')
  .onRun(async (context) => {
    const pending = await getPendingUpdates();
    await processBatch(pending);
  });

// Weekly tier recalculation (Sunday 2am)
exports.weeklyTierRecalc = functions.pubsub
  .schedule('0 2 * * 0')
  .timeZone('America/New_York')
  .onRun(async (context) => {
    await tierService.recalculateAllTiers();
  });
```

---

## Queue System (Firestore-based)

For simple queuing without Cloud Tasks/Pub/Sub overhead:

```javascript
// Add to queue
async function enqueue(type, payload, shopId) {
  await firestore.collection('queues').add({
    type,
    payload,
    shopId,
    status: 'pending',
    createdAt: new Date(),
    attempts: 0
  });
}

// Process queue with Firestore trigger
exports.processQueue = functions.firestore
  .document('queues/{docId}')
  .onCreate(async (snap, context) => {
    const job = snap.data();
    const docRef = snap.ref;

    try {
      await docRef.update({status: 'processing'});

      switch (job.type) {
        case 'sync_customer':
          await syncCustomer(job.payload);
          break;
        case 'send_email':
          await sendEmail(job.payload);
          break;
      }

      await docRef.update({status: 'completed', completedAt: new Date()});
    } catch (error) {
      const attempts = job.attempts + 1;
      if (attempts >= 3) {
        await docRef.update({status: 'failed', error: error.message});
      } else {
        await docRef.update({status: 'pending', attempts});
      }
    }
  });
```

### Queue vs Cloud Tasks vs Pub/Sub

| Feature | Firestore Queue | Cloud Tasks | Pub/Sub |
|---------|-----------------|-------------|---------|
| Setup complexity | Low | Medium | Medium |
| Delayed execution | ❌ Manual | ✅ Built-in | ❌ No |
| Rate limiting | ❌ Manual | ✅ Built-in | ❌ No |
| Fan-out | ❌ No | ❌ No | ✅ Yes |
| High volume | ⚠️ Limited | ✅ Good | ✅ Best |
| Cost | Firestore reads | Task fees | Message fees |
| Retries | Manual | Automatic | Automatic |

---

## Checklist

```
□ Independent async operations use Promise.all
□ No await inside loops
□ Functions right-sized (memory, timeout)
□ Webhooks respond < 5 seconds
□ Heavy processing in background queue
□ Proper error handling with try/catch
□ High-volume events use Pub/Sub fan-out
□ Delayed tasks use Cloud Tasks
□ Simple queuing uses Firestore triggers
□ Scheduled jobs use cron functions
```