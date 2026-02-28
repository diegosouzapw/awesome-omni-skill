---
name: Medusa Ecommerce
description: Build ecommerce applications with Medusa v2 - commerce modules, customization, workflows, and deployment
version: 2.0
---

# Medusa Ecommerce Skill

Build modern ecommerce applications with Medusa v2's modular architecture.

## Context7 Integration

Always use context7 MCP for up-to-date API documentation:

```bash
# Primary sources (use in order of preference)
mcp-cli call context7/resolve-library-id '{"libraryName": "medusajs"}'

# Recommended library IDs:
# - /websites/medusajs_learn      - Tutorials, getting started
# - /websites/medusajs_resources  - API reference, detailed docs
# - /llmstxt/medusajs_llms-full_txt - Comprehensive reference
```

**Query pattern:**
```bash
mcp-cli call context7/query-docs '{"libraryId": "/websites/medusajs_learn", "topic": "your topic"}'
```

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         Medusa Application                       │
├─────────────────────────────────────────────────────────────────┤
│  API Layer        │  Admin Routes (/admin)                       │
│                   │  Store Routes (/store)                       │
│                   │  Custom Routes (/custom)                     │
├─────────────────────────────────────────────────────────────────┤
│  Workflows        │  Multi-step operations with compensation     │
├─────────────────────────────────────────────────────────────────┤
│  Commerce Modules │  Product │ Cart │ Order │ Payment │ etc.    │
├─────────────────────────────────────────────────────────────────┤
│  Infrastructure   │  Cache │ Events │ File │ Notification       │
├─────────────────────────────────────────────────────────────────┤
│  Database         │  PostgreSQL with MikroORM                    │
└─────────────────────────────────────────────────────────────────┘
```

## Project Structure

```
my-medusa-store/
├── src/
│   ├── modules/           # Custom modules
│   │   └── my-module/
│   │       ├── index.ts
│   │       ├── service.ts
│   │       └── models/
│   ├── workflows/         # Custom workflows
│   ├── subscribers/       # Event subscribers
│   ├── api/              # API routes
│   │   ├── admin/
│   │   ├── store/
│   │   └── middlewares.ts
│   ├── admin/            # Admin UI extensions
│   │   ├── routes/
│   │   └── widgets/
│   └── links/            # Module links
├── medusa-config.ts
└── package.json
```

## Quick Reference

### Container Resolution

```typescript
// In API routes, subscribers, workflows
const productService = container.resolve("product")
const query = container.resolve("query")

// Common services
// "product", "cart", "order", "customer", "pricing"
// "payment", "fulfillment", "inventory", "region"
// "query" - for Query/Graph operations
// "logger" - for logging
```

### Query (Graph) Operations

```typescript
import { Query } from "@medusajs/framework"

// Retrieve with relations
const { data: products } = await query.graph({
  entity: "product",
  fields: ["id", "title", "variants.*", "variants.prices.*"],
  filters: { id: productId }
})

// Pagination
const { data, metadata } = await query.graph({
  entity: "order",
  fields: ["*", "items.*"],
  pagination: { skip: 0, take: 20 }
})
```

### Remote Query (Cross-Module)

```typescript
import { useQueryGraphStep } from "@medusajs/medusa/core-flows"

// In workflows - query across linked modules
const { data } = await useQueryGraphStep({
  entity: "product",
  fields: ["*", "inventory_items.*"] // Linked data
})
```

### Creating API Routes

```typescript
// src/api/store/custom/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework"

export const GET = async (req: MedusaRequest, res: MedusaResponse) => {
  const query = req.scope.resolve("query")
  // ... handle request
  res.json({ data })
}

export const POST = async (req: MedusaRequest, res: MedusaResponse) => {
  const body = req.body
  // ... handle request
  res.json({ success: true })
}
```

### Protected Admin Routes

```typescript
// src/api/admin/custom/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework"
import { authenticate } from "@medusajs/medusa"

export const GET = async (req: MedusaRequest, res: MedusaResponse) => {
  // req.auth_context.actor_id contains admin user ID
  res.json({ data })
}

// Middleware applies authentication automatically for /admin routes
```

### Creating Workflows

```typescript
import { createWorkflow, createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

const myStep = createStep(
  "my-step",
  async (input: { data: string }, { container }) => {
    const service = container.resolve("myService")
    const result = await service.doSomething(input.data)
    return new StepResponse(result, result.id) // data, compensateInput
  },
  async (id, { container }) => {
    // Compensation (rollback) logic
    const service = container.resolve("myService")
    await service.undoSomething(id)
  }
)

export const myWorkflow = createWorkflow("my-workflow", (input) => {
  const result = myStep(input)
  return result
})
```

### Event Subscribers

```typescript
// src/subscribers/order-placed.ts
import { SubscriberArgs, SubscriberConfig } from "@medusajs/framework"

export default async function orderPlacedHandler({
  event,
  container
}: SubscriberArgs<{ id: string }>) {
  const orderId = event.data.id
  const logger = container.resolve("logger")
  logger.info(`Order placed: ${orderId}`)
}

export const config: SubscriberConfig = {
  event: "order.placed"
}
```

### Data Models

```typescript
import { model } from "@medusajs/framework/utils"

const MyModel = model.define("my_model", {
  id: model.id().primaryKey(),
  name: model.text(),
  description: model.text().nullable(),
  metadata: model.json().nullable(),
  is_active: model.boolean().default(true),
  created_at: model.dateTime(),
})

export default MyModel
```

### Module Links

```typescript
// src/links/product-custom.ts
import { defineLink } from "@medusajs/framework/utils"
import ProductModule from "@medusajs/medusa/product"
import MyModule from "../modules/my-module"

export default defineLink(
  ProductModule.linkable.product,
  MyModule.linkable.myModel
)
```

## Common Operations

### Create Product with Variants

```typescript
const product = await productService.createProducts({
  title: "T-Shirt",
  options: [{ title: "Size", values: ["S", "M", "L"] }],
  variants: [
    { title: "Small", options: { Size: "S" }, prices: [{ amount: 1000, currency_code: "usd" }] },
    { title: "Medium", options: { Size: "M" }, prices: [{ amount: 1000, currency_code: "usd" }] },
  ]
})
```

### Cart to Order Flow

```typescript
// 1. Create cart
const cart = await cartService.createCarts({ region_id, currency_code: "usd" })

// 2. Add items
await cartService.addLineItems(cart.id, [{ variant_id, quantity: 1 }])

// 3. Add shipping/payment
await cartService.addShippingMethods(cart.id, [{ shipping_option_id }])
await paymentService.createPaymentCollections({ cart_id: cart.id })

// 4. Complete checkout (via workflow)
import { completeCartWorkflow } from "@medusajs/medusa/core-flows"
await completeCartWorkflow(container).run({ input: { id: cart.id } })
```

### Payment Integration

```typescript
// Initialize payment session
const paymentCollection = await paymentService.createPaymentCollections({
  cart_id: cartId,
  amount: cart.total,
  currency_code: cart.currency_code,
})

await paymentService.createPaymentSession(paymentCollection.id, {
  provider_id: "stripe", // or your provider
  data: { /* provider-specific */ }
})
```

## CLI Commands

```bash
# Development
npx medusa develop          # Start dev server with admin

# Database
npx medusa db:migrate       # Run migrations
npx medusa db:generate      # Generate migration from models
npx medusa db:sync-links    # Sync module links

# Build
npx medusa build            # Production build
npx medusa start            # Start production server
```

## Reference Files

Detailed documentation organized by topic:

| File | Topics |
|------|--------|
| [commerce-modules.md](references/commerce-modules.md) | Product, Cart, Order, Pricing, Inventory, Customer |
| [checkout-payments.md](references/checkout-payments.md) | Payment, Fulfillment, Tax, Region, Sales Channel |
| [customization.md](references/customization.md) | Custom modules, services, data models, links, API routes |
| [workflows-events.md](references/workflows-events.md) | Workflows, steps, compensation, subscribers, events |
| [admin-storefront.md](references/admin-storefront.md) | Admin UI extensions, JS SDK, storefront integration |
| [infrastructure-production.md](references/infrastructure-production.md) | Redis, S3, SendGrid, deployment, medusa-config.ts |

## Best Practices

1. **Use workflows for multi-step operations** - Built-in compensation handles failures
2. **Query with `query.graph()`** - Efficient data fetching with relations
3. **Extend, don't modify** - Create custom modules instead of modifying core
4. **Use module links** - Connect custom data to core entities
5. **Validate at API layer** - Use Zod schemas for request validation
6. **Subscribe to events** - React to changes asynchronously

## Troubleshooting

**Module not found**: Ensure module is registered in `medusa-config.ts`

**Link not working**: Run `npx medusa db:sync-links` after adding links

**Migration issues**: Check model definitions match database schema

**Type errors**: Regenerate types with `npx medusa generate:types`
