---
name: hive-database
description: Database operations in Hive framework
globs:
  - src/**/*.js
alwaysApply: false
---

# Database Operations

## Access Service

```javascript
import db from 'db';
const taskService = db.services.tasks;  // From schema name
```

## Find Operations

```javascript
// Find many
const { results, pagesCount, count } = await service.find(
  { status: 'active' },
  { page: 1, perPage: 20, sort: '-createdOn', fields: ['_id', 'title'] }
);

// Find one
const doc = await service.findOne({ _id: id });

// Check exists
const exists = await service.exists({ _id: id });

// Count
const total = await service.count({ status: 'active' });

// Distinct values
const statuses = await service.distinct('status');

// Aggregate
const { results } = await service.aggregate([
  { $match: { status: 'done' } },
  { $group: { _id: '$project._id', count: { $sum: 1 } } },
]);
```

## Write Operations

```javascript
// Create
const doc = await service.create({ title: 'New', user: ctx.state.user });

// Update one (must use function)
const updated = await service.updateOne(
  { _id: id },
  (doc) => ({ ...doc, title: 'Updated' })
);

// Update many
await service.updateMany(
  { status: 'pending' },
  (doc) => ({ ...doc, status: 'active' })
);

// Remove
await service.remove({ _id: id });

// Generate ID
const newId = service.generateId();
```

## Atomic Operations (No Events)

```javascript
// Silent bulk update
await service.atomic.update(
  { 'project._id': projectId },
  { $set: { 'project.name': newName } },
  { multi: true }
);
```

## Query Patterns

```javascript
import { when } from 'services/utils';

const { results } = await service.find({
  // Conditional fields
  ...when(status, { status }),
  ...when(userId, { 'user._id': userId }),
  
  // Complex conditions
  status: { $in: ['active', 'pending'] },
  createdOn: { $gte: startDate, $lt: endDate },
  $or: [{ 'user._id': id }, { isPublic: true }],
});
```

## Rules

- `updateOne` requires a function, not an object
- Use `atomic.update` for bulk/silent updates
- Use `when()` helper for conditional query building
