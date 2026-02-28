---
name: hive-overview
description: Hive framework structure and conventions. Apply when working with this codebase.
globs:
  - src/**/*.js
alwaysApply: false
---

# Hive Framework Overview

Hive is a Koa-based Node.js framework with MongoDB, Zod validation, and auto-sync for embedded documents.

## Directory Structure

| Directory | Purpose | Edit? |
|-----------|---------|-------|
| `/.hive/` | Framework core | No - read only |
| `/src/` | Application code | Yes |

## Resources

A resource is a feature module in `/src/resources/{name}/`:

```
src/resources/{name}/
├── {name}.schema.js      # Required: DB schema
├── endpoints/            # API routes
├── handlers/             # Event listeners
└── methods/              # Reusable functions
```

## Built-in (in .hive/)

- `tokens` - Auth token storage
- `users` - Base user schema  
- `db` - Database layer with services

## Key Imports

```javascript
import db from 'db';                    // Database
import config from 'app-config';        // Config
import { z } from 'zod';                // Validation
```

## Services Access

```javascript
const taskService = db.services.tasks;  // Auto-created from schema
```
