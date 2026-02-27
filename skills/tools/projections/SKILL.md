---
name: projections
description: Build or query TES projections (current state views). State is derived from TES events - projections are the queryable current truth.
allowed-tools: Read, Grep, Glob, Edit, Write
---

# Projections Skill

## Core Principle

**State is derived from TES events. Projections are the queryable current truth.**

---

## Projection Types

| Projection | Purpose | Key |
|------------|---------|-----|
| ItemCurrentState | Current state of an item | item canonical_id |
| ListingCurrentState | Current listing status | listing canonical_id |
| ParticipantInventory | What a user owns | participant canonical_id |
| CampaignState | Campaign metrics | campaign canonical_id |

---

## Durable Object Pattern

```javascript
// workers/projections/ItemProjection.js

export class ItemProjection {
  constructor(state, env) {
    this.state = state;
    this.env = env;
  }

  async fetch(request) {
    const url = new URL(request.url);

    if (request.method === "GET") {
      const currentState = await this.state.storage.get("state");
      return Response.json(currentState || { error: "Not found" });
    }

    if (request.method === "POST" && url.pathname === "/apply-event") {
      const event = await request.json();
      return this.applyEvent(event);
    }
  }

  async applyEvent(event) {
    // Idempotency check
    const processedEvents = await this.state.storage.get("processed") || [];
    if (processedEvents.includes(event.id)) {
      return Response.json({ status: "already_processed" });
    }

    // Get current state
    let state = await this.state.storage.get("state") || this.initialState();

    // Apply event
    state = this.reduce(state, event);
    state.last_event_id = event.id;
    state.last_updated = event.timestamp;

    // Save
    await this.state.storage.put("state", state);
    await this.state.storage.put("processed", [...processedEvents.slice(-100), event.id]);

    return Response.json({ status: "applied", state });
  }

  reduce(state, event) {
    switch (event.type) {
      case "item.captured":
        return { ...state, status: "captured", media: event.payload.image_urls };
      case "item.identified":
        return { ...state, status: "identified", product_ref: event.payload.product_ref };
      case "item.listed":
        return { ...state, status: "listed", listing_id: event.payload.listing_id };
      case "item.sold":
        return { ...state, status: "sold", sold_at: event.timestamp };
      default:
        return state;
    }
  }

  initialState() {
    return { status: "unknown", created_at: new Date().toISOString() };
  }
}
```

---

## Querying Projections

```javascript
// Get current item state
async function getItemState(env, itemId) {
  const doId = env.ITEM_PROJECTION.idFromName(itemId);
  const stub = env.ITEM_PROJECTION.get(doId);

  const response = await stub.fetch(new Request("https://do/"));
  return response.json();
}

// In API handler
const itemState = await getItemState(env, "ptc_item_abc123");
```

---

## TES Router (Event Fan-out)

```javascript
// workers/tes-router.js

export default {
  async queue(batch, env) {
    for (const message of batch.messages) {
      const event = message.body;

      // Route to appropriate projections
      const routes = getRoutes(event);

      await Promise.all(routes.map(async (route) => {
        const doId = env[route.binding].idFromName(route.key);
        const stub = env[route.binding].get(doId);

        await stub.fetch(new Request("https://do/apply-event", {
          method: "POST",
          body: JSON.stringify(event),
        }));
      }));

      message.ack();
    }
  },
};

function getRoutes(event) {
  const routes = [];

  if (event.entity_type === "item") {
    routes.push({ binding: "ITEM_PROJECTION", key: event.entity_id });
  }

  if (event.type === "item.sold") {
    routes.push({ binding: "LISTING_PROJECTION", key: event.payload.listing_id });
    routes.push({ binding: "PARTICIPANT_PROJECTION", key: event.payload.seller_id });
    routes.push({ binding: "PARTICIPANT_PROJECTION", key: event.payload.buyer_id });
  }

  return routes;
}
```

---

## Anti-Patterns

- Querying D1 for "current truth" instead of projections
- Mutating projection state without event
- Global projections (hot-key bottleneck)
- Not handling idempotency
