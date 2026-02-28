---
name: hive-scheduler
description: How to create scheduled jobs in Hive framework
globs:
  - src/scheduler/handlers/*.js
alwaysApply: false
---

# Scheduler Jobs

Background jobs on cron schedules.

Location: `/src/scheduler/handlers/{jobName}.js`

## Template

```javascript
import db from 'db';

export const handler = async () => {
  // Job logic
};

export const cron = '0 * * * *';  // Every hour
```

## Cron Patterns

```
┌─ minute (0-59)
│ ┌─ hour (0-23)
│ │ ┌─ day of month (1-31)
│ │ │ ┌─ month (1-12)
│ │ │ │ ┌─ day of week (0-6)
* * * * *
```

| Pattern | Schedule |
|---------|----------|
| `* * * * *` | Every minute |
| `*/5 * * * *` | Every 5 minutes |
| `0 * * * *` | Every hour |
| `0 */12 * * *` | Every 12 hours |
| `0 0 * * *` | Daily at midnight |
| `0 9 * * 1` | Mondays at 9am |

## Examples

**Sync external data:**
```javascript
import db from 'db';
import moment from 'moment';
import externalApi from 'services/externalApi';

export const handler = async () => {
  const items = await externalApi.list({
    updatedSince: moment().subtract(5, 'minutes').toDate(),
  });
  
  for (const item of items) {
    await db.services.items.updateOne(
      { externalId: item.id },
      (doc) => ({ ...doc, ...item })
    );
  }
};

export const cron = '*/5 * * * *';
```

**Mark overdue:**
```javascript
import db from 'db';
import moment from 'moment';

export const handler = async () => {
  await db.services.invoices.updateMany(
    {
      isPaid: { $ne: true },
      isDue: { $ne: true },
      dueOn: { $lt: new Date() },
    },
    (doc) => ({ ...doc, isDue: true })
  );
};

export const cron = '0 */12 * * *';
```

## Rules

- Export both `handler` and `cron`
- Keep jobs idempotent (safe to re-run)
- Log progress for debugging
