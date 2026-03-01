---
name: blong-layer
description: Organize handlers into named functional groups within a Blong realm. Layers include gateway (API), adapter (external systems), orchestrator (business logic), error (domain errors), and test (automation). Use when organizing code by functional concern, defining handler groups, or controlling deployment activation.
---

# Implementing a Layer

## Overview

Layers are named groups of handlers that organize code by functional concern within a realm. They enable clear separation of responsibilities and support the framework's modular architecture.

**Key Pattern:** Layers are self-contained — each layer file defines its own configuration and validation, co-located with its implementation. No need to maintain configuration in a central `server.ts`.

## Purpose

- **Separation of Concerns:** Group related functionality together
- **Co-located Config:** Each layer owns its configuration and validation
- **Code Organization:** Clear folder structure for handlers
- **Deployment Control:** Layers can be activated/deactivated per environment
- **Team Coordination:** Different teams can work on different layers

## Recommended Layer Names

### Server-Side Layers (auto-detected)

- **`gateway`** - API gateway: routes, validation, documentation (minimal business logic)
- **`adapter`** - External system communication: SQL, HTTP, FTP, mail protocols
- **`orchestrator`** - Business process coordination between adapters
- **`error`** - Domain-specific error definitions
- **`test`** - Test automation (dev/build only)
- **`eft`** - Electronic Funds Transfer (high TPS OLTP requirements)

### Browser-Side Layers (auto-detected)

- **`backend`** - Browser adapter talking to server
- **`component`** - React UI components
- **`browser`** - Server-side code serving browser assets

## Folder Structure

### Typical Realm with Layers

Well-known layer folders are auto-discovered — no `layer.server.ts` needed:

```
realmname/
├── server.ts            # Optional — only for realm-level config/validation
├── error/               # Auto-activated (well-known name)
│   └── error.ts
├── adapter/             # Auto-activated (well-known name)
│   ├── db.ts            # Self-contained database adapter (with config)
│   ├── http.ts          # Self-contained HTTP adapter (with config)
│   └── db/              # Handler group: realmname.db
│       ├── userUserAdd.ts
│       └── userUserFind.ts
├── orchestrator/        # Auto-activated (well-known name)
│   └── dispatch.ts      # Self-contained orchestrator (with config)
└── gateway/             # Auto-activated (well-known name)
    └── api/
        └── user.yaml
```

Custom (non-well-known) layer folders need a `layer.server.ts` or `layer.browser.ts`:

```
realmname/
└── myCustomLayer/
    └── layer.server.ts  # Required: declares activation for non-well-known folder
```

## layer.server.ts / layer.browser.ts

For folders with non-standard names, add a `layer.server.ts` or `layer.browser.ts` to declare activation. This eliminates the need for an explicit `children` array or activation config in the parent `server.ts`.

```typescript
// myCustomLayer/layer.server.ts
import {layer} from '@feasibleone/blong';

export default layer({
    default: true, // active in all environments
    microservice: true, // additionally active in microservice deployment
});
```

```typescript
// myBrowserLayer/layer.browser.ts
import {layer} from '@feasibleone/blong';

export default layer({
    integration: true, // only active during integration testing
});
```

### Well-Known Layer Default Activations

| Folder         | Server default        | Browser default       |
| -------------- | --------------------- | --------------------- |
| `error`        | `{default: true}`     | —                     |
| `sim`          | `{integration: true}` | -                     |
| `adapter`      | `{default: true}`     | -                     |
| `orchestrator` | `{default: true}`     | —                     |
| `gateway`      | `{default: true}`     | —                     |
| `browser`      | `{default: true}`     | —                     |
| `backend`      |                       | `{default: true}`     |
| `component`    |                       | `{default: true}`     |
| `test`         |                       | `{integration: true}` |

## Self-Contained Layer Pattern

### Adapter Layer Definition

```typescript
// adapter/db.ts - self-contained with config and validation
import {adapter} from '@feasibleone/blong';

export default adapter(blong => ({
    extends: 'adapter.knex',

    // Layer's own validation schema
    validation: blong.type.Object({
        namespace: blong.type.Union([blong.type.String(), blong.type.Array(blong.type.String())]),
        imports: blong.type.Union([blong.type.String(), blong.type.Array(blong.type.String())]),
        logLevel: blong.type.Optional(blong.type.String()),
    }),

    // Layer's own configuration per environment
    activation: {
        default: {
            namespace: 'db/$subject',
            imports: '$subject.db',
        },
        dev: {
            logLevel: 'trace',
        },
        prod: {
            logLevel: 'warn',
        },
    },
}));
```

### Orchestrator Layer Definition

```typescript
// orchestrator/dispatch.ts - self-contained with config and validation
import {orchestrator} from '@feasibleone/blong';

export default orchestrator(blong => ({
    extends: 'orchestrator.dispatch',

    validation: blong.type.Object({
        namespace: blong.type.Union([blong.type.String(), blong.type.Array(blong.type.String())]),
        imports: blong.type.Union([
            blong.type.String(),
            blong.type.Array(blong.type.String()),
            blong.type.Array(blong.type.RegExp()),
        ]),
        validations: blong.type.Optional(
            blong.type.Array(blong.type.Union([blong.type.String(), blong.type.RegExp()])),
        ),
        destination: blong.type.Optional(blong.type.String()),
    }),

    activation: {
        default: {
            destination: 'db',
            namespace: ['$subject'],
            imports: [/^$subject\./],
            validations: [/^$subject\.\w+\.validation$/],
        },
    },
}));
```

### Realm Entry Point (Only When Needed)

`server.ts` is optional. It is only needed when there is realm-level config or validation shared across layers.

```typescript
// server.ts — only when realm-level config is needed
import {realm} from '@feasibleone/blong';

export default realm(blong => ({
    url: import.meta.url,
    validation: blong.type.Object({
        myService: blong.type.Object({url: blong.type.String()}),
    }),
    activation: {
        default: {
            myService: {url: 'http://localhost:8080'},
        },
    },
}));
```

## Handler Group Pattern

### Group Naming Convention

Groups are named in format: `realmname.foldername`

Example:

- Realm: `user`
- Folder: `orchestrator/user/`
- Group name: `user.user`

### Referencing Groups

Groups are referenced in the `imports` property of the adapter/orchestrator:

```typescript
// adapter/db.ts - imports declared inline
export default adapter(blong => ({
    extends: 'adapter.knex',
    activation: {
        default: {
            namespace: ['user', 'role'],
            imports: ['user.user', 'user.role'], // handler groups loaded
        },
    },
}));
```

## Implementation Patterns

### Error Layer

```typescript
// realmname/error/error.ts
export default {
    userNotFound: 'User not found',
    userExists: 'User already exists',
    invalidEmail: 'Invalid email format',
    permissionDenied: 'Permission denied',
};
```

### Adapter Layer Structure

```
adapter/
├── db.ts              # Self-contained database adapter
├── http.ts            # Self-contained HTTP adapter
├── db/                # Handler group for db operations
│   ├── userAdd.ts
│   └── userFind.ts
└── http/              # Handler group for HTTP operations
    ├── send.ts
    └── receive.ts
```

### Orchestrator Layer Structure

```
orchestrator/
├── dispatch.ts        # Self-contained orchestrator definition
├── entity1/           # Handler group: realmname.entity1
│   ├── ~.schema.ts   # Auto-generated validation
│   ├── helper.ts     # Library function
│   └── realmEntity1Action.ts
└── entity2/           # Handler group: realmname.entity2
    ├── ~.schema.ts
    └── realmEntity2Action.ts
```

### Gateway Layer Structure

```
gateway/
└── api/
    ├── entity1.yaml   # OpenAPI spec for entity1
    └── entity2.yaml   # OpenAPI spec for entity2
```

Or with custom validation:

```
gateway/
└── entity/
    ├── entityAction1.ts  # Manual validation
    └── entityAction2.ts
```

### Test Layer Structure

```
test/
└── test/              # Handler group: test.test
    ├── testEntity1.ts
    ├── testEntity2.ts
    └── testWorkflow.ts
```

## Layer Activation

### In Realm Configuration (server.ts)

```typescript
export default realm(blong => ({
    config: {
        // Activate for automated testing
        test: {
            error: true,
            adapter: true,
            orchestrator: true,
            gateway: true,
            test: true,
        },

        // Activate for microservice deployment
        microservice: {
            error: true,
            adapter: true,
            orchestrator: true,
            gateway: true,
        },

        // Activate for single realm dev focus
        realm: {
            adapter: true,
            orchestrator: true,
        },

        // Development with full stack
        dev: {
            error: true,
            adapter: true,
            orchestrator: true,
            gateway: true,
        },
    },
}));
```

## One Handler Per File Pattern

### Benefits

1. **Fast Discovery:** Use `ctrl+p` in VS Code to find handlers quickly
    - Example: `ctrl+p uua` finds `userUserAdd.ts`
2. **Easier Code Review:** Smaller files, less nesting
3. **Better Isolation:** Clear boundaries between handlers
4. **Git-Friendly:** Smaller diffs, fewer conflicts

### Naming Convention

File name = handler name:

- Handler: `userUserAdd` → File: `userUserAdd.ts`
- Handler: `mathNumberSum` → File: `mathNumberSum.ts`
- Library: `validateEmail` → File: `validateEmail.ts`

## Adding a New Adapter

### Before (Old Pattern) - Touch 2 files

```typescript
// 1. Create adapter/newadapter.ts
export default adapter(() => ({
    extends: 'adapter.http',
}));

// 2. Update server.ts validation + config + children
// (3 edits in server.ts)
```

### After (New Pattern) - Touch 1 file

```typescript
// 1. Create adapter/newadapter.ts - everything co-located
export default adapter(blong => ({
    extends: 'adapter.http',

    validation: blong.type.Object({
        url: blong.type.String(),
        timeout: blong.type.Number(),
    }),

    activation: {
        default: {
            url: 'http://api.example.com',
            timeout: 5000,
        },
    },
}));

// 2. Done! Framework uses the layer's own config.
```

## Multi-Layer Example

### Complete Realm Structure

```
payment/
├── server.ts           # Minimal - activation only
├── error/
│   └── error.ts
├── adapter/
│   ├── db.ts           # Self-contained: config + validation inside
│   ├── fspiop.ts       # Self-contained: config + validation inside
│   └── db/
│       ├── paymentCreate.ts
│       └── paymentFind.ts
├── orchestrator/
│   ├── dispatch.ts     # Self-contained: config + validation inside
│   ├── transfer/
│   │   ├── ~.schema.ts
│   │   ├── calculateFees.ts
│   │   ├── paymentTransferPrepare.ts
│   │   └── paymentTransferCommit.ts
│   └── quote/
│       ├── ~.schema.ts
│       └── paymentQuoteCreate.ts
├── gateway/
│   └── api/
│       ├── transfer.yaml
│       └── quote.yaml
└── test/
    └── test/
        ├── testTransfer.ts
        └── testQuote.ts
```

## Best Practices

1. **Co-locate Config:** Put layer configuration in the layer file, not in server.ts
2. **Clear Separation:** Keep business logic in orchestrator, not gateway
3. **Consistent Naming:** Use lowercase single words for layer names
4. **Group by Entity:** Organize handlers by business entity within layers
5. **Library Functions:** Extract reusable code into library functions
6. **Error Definitions:** Always define errors in error layer first
7. **Test Coverage:** Create test layer with comprehensive test handlers
8. **One File Per Handler:** Follow the one handler per file pattern
9. **Validation:** Use `~.schema.ts` for automatic validation generation
10. **Import Order:** Load layers in order: error → adapter → orchestrator → gateway → test

## Examples from Codebase

- **Complete realm:** `core/test/demo/`
- **Payment realm:** `ml/payment/`
- **Agreement realm:** `ml/agreement/`
