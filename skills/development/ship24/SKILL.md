---
name: ship24
description: Track parcels worldwide via Ship24 API. Create trackers, get tracking results, monitor delivery status for packages from DHL, FedEx, USPS, DPD, and 1500+ carriers. Use when tracking packages, monitoring deliveries, or checking shipment status.
---

# Ship24 Tracking Skill

Track parcels worldwide using the Ship24 API. Supports 1500+ carriers including DHL, FedEx, USPS, UPS, DPD, China Post, and more.

## Setup

**Get API Key:** https://dashboard.ship24.com/integrations/api-keys

**Store credentials** in `~/.config/ship24/credentials.json`:
```json
{
  "api_key": "apik_xxx"
}
```

Or use environment variable: `SHIP24_API_KEY`

## API Base

```
https://api.ship24.com/public/v1
```

**Auth Header:** `Authorization: Bearer YOUR_API_KEY`

---

## Quick Reference

### Create a Tracker

```bash
curl -X POST "https://api.ship24.com/public/v1/trackers" \
  -H "Authorization: Bearer $SHIP24_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "trackingNumber": "1234567890",
    "courierCode": ["dpd-de"],
    "destinationCountryCode": "DE"
  }'
```

**Response:**
```json
{
  "data": {
    "tracker": {
      "trackerId": "abc-123-def",
      "trackingNumber": "1234567890",
      "isSubscribed": true,
      "isTracked": true,
      "createdAt": "2026-01-15T10:00:00.000Z"
    }
  }
}
```

### Get Tracking Results

```bash
curl "https://api.ship24.com/public/v1/trackers/$TRACKER_ID/results" \
  -H "Authorization: Bearer $SHIP24_API_KEY"
```

**Response:**
```json
{
  "data": {
    "trackings": [{
      "tracker": { "trackerId": "abc-123" },
      "shipment": {
        "statusCode": "in_transit",
        "statusMilestone": "in_transit",
        "originCountryCode": "CN",
        "destinationCountryCode": "DE"
      },
      "events": [{
        "status": "Parcel is in transit",
        "occurrenceDatetime": "2026-01-16T14:30:00",
        "location": "Frankfurt, DE",
        "statusCode": "in_transit"
      }]
    }]
  }
}
```

### Create Tracker and Get Results (One Call)

```bash
curl -X POST "https://api.ship24.com/public/v1/trackers/track" \
  -H "Authorization: Bearer $SHIP24_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "trackingNumber": "1234567890"
  }'
```

### List All Trackers

```bash
curl "https://api.ship24.com/public/v1/trackers?page=1&limit=25" \
  -H "Authorization: Bearer $SHIP24_API_KEY"
```

### Search by Tracking Number (No Tracker)

```bash
curl -X POST "https://api.ship24.com/public/v1/tracking/search" \
  -H "Authorization: Bearer $SHIP24_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"trackingNumber": "1234567890"}'
```

---

## Status Codes

| Milestone | Description |
|-----------|-------------|
| `pending` | Tracking created, no events yet |
| `info_received` | Carrier received shipment info |
| `in_transit` | Package in transit |
| `out_for_delivery` | Out for delivery today |
| `delivered` | Successfully delivered |
| `failed_attempt` | Delivery attempt failed |
| `available_for_pickup` | Ready for pickup |
| `exception` | Problem with delivery |
| `expired` | Tracking expired |

**Status Categories:** `pending`, `info_received`, `in_transit`, `out_for_delivery`, `delivery`, `exception`

---

## Common Courier Codes

| Carrier | Code |
|---------|------|
| DHL Express | `dhl` |
| DHL Germany | `dhl-de` |
| DPD Germany | `dpd-de` |
| DPD UK | `dpd-uk` |
| FedEx | `fedex` |
| UPS | `ups` |
| USPS | `us-post` |
| China Post | `china-post` |
| Deutsche Post | `deutsche-post` |
| Hermes Germany | `hermes-de` |
| GLS | `gls` |

Full list: `GET /public/v1/couriers`

---

## Heartbeat Integration

Track deliveries in your heartbeat routine. Store state in your memory file:

```json
{
  "deliveries": [
    {
      "tracking": "1234567890",
      "carrier": "DPD",
      "description": "New laptop",
      "ship24TrackerId": "abc-123-def",
      "lastStatus": "in_transit",
      "lastCheck": "2026-01-16T10:00:00Z"
    }
  ],
  "ship24ApiKey": "apik_xxx"
}
```

**Check routine:**
1. Loop through active deliveries
2. Fetch results for each trackerId
3. Compare status with lastStatus
4. Notify human on significant changes:
   - `out_for_delivery` → "Your package is out for delivery today!"
   - `delivered` → "Package delivered!" + remove from list
   - `exception` → "Delivery problem with [description]"
5. Update lastStatus and lastCheck

---

## Adding a Delivery

When human says "track this package: ABC123":

1. Create tracker:
```bash
curl -X POST "https://api.ship24.com/public/v1/trackers" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"trackingNumber": "ABC123"}'
```

2. Save to memory:
```json
{
  "tracking": "ABC123",
  "ship24TrackerId": "<trackerId from response>",
  "description": "Package from human",
  "lastStatus": "pending",
  "added": "2026-01-16"
}
```

3. Confirm: "Added tracking for ABC123. I'll let you know when there are updates."

---

## Tips

- **Auto-detect carrier:** Ship24 usually detects the carrier automatically. Only specify `courierCode` if you know it.
- **Destination country:** Helps with detection. Use ISO 2-letter codes (DE, US, CN).
- **Idempotent:** Creating a tracker with the same tracking number returns the existing one.
- **Rate limits:** Check your plan limits at dashboard.ship24.com
- **Webhooks:** For real-time updates, configure webhooks in the dashboard instead of polling.

---

## Error Handling

| Code | Meaning |
|------|---------|
| 401 | Invalid API key |
| 404 | Tracker not found |
| 429 | Rate limit exceeded |
| 422 | Invalid tracking number format |

---

## Links

- **Dashboard:** https://dashboard.ship24.com
- **API Docs:** https://docs.ship24.com
- **OpenAPI Spec:** https://docs.ship24.com/assets/openapi/ship24-tracking-api.yaml
