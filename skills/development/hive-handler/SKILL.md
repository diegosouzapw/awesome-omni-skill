---
name: hive-handler
description: How to create event handlers in Hive framework
globs:
  - src/resources/**/handlers/*.js
alwaysApply: false
---

# Creating Event Handlers

Handlers react to database events automatically.

Location: `/src/resources/{name}/handlers/{handlerName}.js`

## Template

```javascript
import db from 'db';

const myService = db.services.myResource;

myService.on('created', async ({ doc }) => {
  // Handle creation
});

myService.on('updated', async ({ doc, prevDoc }) => {
  // Handle update
});

myService.on('removed', async ({ doc }) => {
  // Handle removal
});
```

## Events

| Event | Payload | Trigger |
|-------|---------|---------|
| `created` | `{ doc }` | After `service.create()` |
| `updated` | `{ doc, prevDoc }` | After `service.updateOne/Many()` |
| `removed` | `{ doc }` | After `service.remove()` |

## Common Patterns

**Create related record:**
```javascript
taskService.on('created', async ({ doc }) => {
  await eventService.create({
    type: 'task.created',
    data: { task: doc },
    project: doc.project,
  });
});
```

**Update parent on child change:**
```javascript
docService.on('created', async ({ doc }) => {
  if (doc.role === 'case-study') {
    await projectService.updateOne(
      { _id: doc.project._id },
      (p) => ({ ...p, caseStudy: doc })
    );
  }
});
```

**React to specific field change:**
```javascript
import ifUpdated from 'helpers/db/ifUpdated';

taskService.on('updated', ifUpdated(['status'], async ({ doc, prevDoc }) => {
  // Only runs when status changed
}));
```

**Cleanup on delete:**
```javascript
docService.on('removed', async ({ doc }) => {
  await eventService.remove({ 'data.doc._id': doc._id });
});
```

## Rules

- Keep handlers fast (don't block response)
- Use `atomic.update` for silent bulk updates (no events)
- Use `ifUpdated` helper to filter field changes
