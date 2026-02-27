---
name: hubble-cloud-api
description: This skill should be used when the user needs to integrate with Hubble Network Platform API for IoT device management, packet data retrieval, webhook configuration, metrics tracking, user management, or billing. Triggers include 'Hubble API', 'Hubble devices', 'Hubble packets', 'register IoT devices', 'configure webhooks for Hubble', 'Hubble Network integration', 'stream packet data'.
---

# Hubble Cloud API Integration

## Overview

Hubble Network connects off-the-shelf Bluetooth chips to a global network of 90M+ gateways. This skill helps you integrate with the Hubble Cloud API to manage devices, retrieve packet data, configure webhooks, monitor metrics, and manage your organization.

**API Base URL**: `https://api.hubble.com`

**API Version**: Rolling release with continuous deployment and backward compatibility

## Domain Context

### Key Concepts

**Device**: A Bluetooth-capable hardware endpoint registered on Hubble Network. Each device has:
- Unique device identifier (dev_eui)
- Device encryption keys for secure transmission
- Custom tags for organization and routing
- Platform-assigned tags (e.g., `hubnet.platform: LoRaWAN`)

**Packet**: Base64-encoded Bluetooth data transmitted by devices and received via gateways. Packets contain:
- Encrypted payload data
- Sequence numbers for ordering
- Authentication tags for verification
- Metadata (RSSI, SNR, gateway info, timestamps)

**Webhook**: HTTP endpoints you control that receive real-time packet data in batches. Key features:
- Push-based delivery (no polling required)
- Configurable batch sizes (10-1000 packets)
- Automatic retries on failure
- Delivery metrics for monitoring

**Organization**: Top-level entity containing devices, API keys, users, and webhooks. Organizations have:
- Unique organization ID (UUID)
- Multiple users with different roles
- API keys with granular scopes
- Billing and usage tracking

**API Keys**: JWT bearer tokens with 16 distinct authorization scopes controlling access to:
- Device management (read/write)
- Packet access (read)
- Webhook configuration (read/write)
- User management (read/write)
- Metrics and billing data (read)

### Data Flow

1. **Device broadcasts** Bluetooth packet
2. **Gateway receives** and forwards to Hubble Network
3. **Platform processes** and decrypts packet
4. **Data available via**:
   - Real-time webhooks (push notification)
   - API streaming (pagination with continuation tokens)
   - Metrics dashboard

### Typical Use Cases

- **Device onboarding**: Register new IoT devices before deployment
- **Packet retrieval**: Stream historical or real-time packet data
- **Webhook setup**: Configure push notifications for packet delivery
- **Fleet management**: Tag, organize, and monitor large device deployments
- **Metrics monitoring**: Track API usage, packet volumes, webhook health
- **Access control**: Manage users and API key scopes

## Authentication

### Bearer Token Authentication

All API requests require a JWT bearer token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

### Obtaining API Keys

1. Log in to Hubble Dashboard
2. Navigate to **Developer → API Tokens**
3. Click **Generate New Token**
4. Select required scopes (principle of least privilege)
5. Copy and securely store the token

**Important**: API keys cannot be retrieved after creation. Store them securely.

### Authorization Scopes

The API supports 16 distinct scopes for granular access control:

**Device Management**:
- `devices:read` - List and retrieve device information
- `devices:write` - Register, update, and delete devices

**Packet Access**:
- `packets:read` - Query and stream packet data

**Webhook Management**:
- `webhooks:read` - List webhook configurations
- `webhooks:write` - Create, update, and delete webhooks

**Organization Management**:
- `organization:read` - View organization details
- `organization:write` - Update organization settings

**User Management**:
- `users:read` - List organization users
- `users:write` - Invite, update, and remove users

**Invitations**:
- `invitations:read` - List pending invitations
- `invitations:write` - Send and revoke invitations

**API Keys**:
- `api_keys:read` - List API keys
- `api_keys:write` - Create and delete API keys

**Metrics**:
- `metrics:read` - Access API, packet, webhook, and device metrics

**Billing**:
- `billing:read` - View invoices and usage data

**Best Practice**: Request only the scopes your application needs. For read-only integrations, use `*:read` scopes only.

## Rate Limits

### Rate Limit Policy

- **Per-endpoint limit**: 3 requests/second
- **Organization-wide limit**: 15 requests/second
- **HTTP 429**: Rate limit exceeded

### Rate Limit Headers

Response headers indicate your current rate limit status:

```
X-RateLimit-Limit: 3
X-RateLimit-Remaining: 2
X-RateLimit-Reset: 1640000000
```

### Handling Rate Limits

When you receive a 429 response:

1. **Read `Retry-After` header** (seconds to wait)
2. **Implement exponential backoff** (1s, 2s, 4s, 8s, ...)
3. **Use batch operations** when available (e.g., batch device updates)
4. **Cache responses** to reduce repeated requests
5. **Spread requests** across time windows

**Example exponential backoff**:
```python
import time

def make_request_with_backoff(func, max_retries=5):
    for i in range(max_retries):
        response = func()
        if response.status_code != 429:
            return response
        wait_time = 2 ** i  # Exponential: 1, 2, 4, 8, 16 seconds
        time.sleep(wait_time)
    raise Exception("Max retries exceeded")
```

## API Endpoints Overview

The Hubble Cloud API provides 40+ endpoints organized into 9 categories. For complete endpoint documentation, see [API_REFERENCE.md](./API_REFERENCE.md).

### Device Management

- `POST /api/v2/org/{org_id}/devices` - Batch register devices (up to 1,000 per request)
- `GET /api/org/{org_id}/devices` - List devices with filtering and sorting
- `GET /api/org/{org_id}/devices/{device_id}` - Retrieve specific device details
- `PATCH /api/org/{org_id}/devices/{device_id}` - Update device metadata and tags
- `PATCH /api/org/{org_id}/devices` - Batch update up to 1,000 devices
- `DELETE /api/org/{org_id}/devices` - Batch delete up to 1,000 devices

**Key features**:
- **Batch operations**: Create, update, and delete up to 1,000 devices per API call
- **Auto-generated credentials**: API generates device IDs and encryption keys
- **Encryption options**: AES-256-CTR, AES-128-CTR, or NONE
- **Device management**: Names, custom tags, platform tags, timestamp filtering

### Packet Retrieval

- `GET /api/org/{org_id}/packets` - Stream packets with pagination

**Parameters**:
- `start_time` / `end_time` - Time range (default 7 days)
- `device_id` - Filter by specific device
- `platform_tag` - Filter by tag (e.g., `hubnet.platform=LoRaWAN`)

**Pagination**: Uses `Continuation-Token` header for streaming large datasets

#### Packet Structure

The API returns packets with a **nested structure**. Understanding this structure is critical for extracting device information and metadata:

```json
{
  "location": {
    "timestamp": 1765598212.181298,    // Unix timestamp (seconds)
    "latitude": 47.61421,
    "longitude": -122.31929,
    "altitude": 829,
    "horizontal_accuracy": 29,
    "vertical_accuracy": 29
  },
  "device": {
    "id": "bc17a947-7a4f-4cff-9127-340cc4005272",  // Device UUID
    "name": "Device Display Name",
    "tags": ["tag1", "tag2"],
    "payload": "SGVsbG8gV29ybGQ=",    // Base64-encoded
    "timestamp": "2025-01-15T10:30:45Z",
    "rssi": -85,                        // Signal strength (dBm)
    "sequence_number": 42,
    "counter": 123
  },
  "network_type": "bluetooth"
}
```

**Important Field Locations**:
- **Device ID**: `packet.device.id` (UUID format)
- **Device Name**: `packet.device.name`
- **RSSI**: `packet.device.rssi` (not at top level)
- **Sequence Number**: `packet.device.sequence_number`
- **GPS Location**: `packet.location.latitude` and `packet.location.longitude`
- **Timestamp**: `packet.location.timestamp` (Unix epoch in seconds)

**Common Mistakes**:
- ❌ `packet.device_id` - Does not exist
- ❌ `packet.dev_eui` - Does not exist at top level
- ❌ `packet.rssi` - Does not exist at top level
- ✅ `packet.device.id` - Correct way to get device ID
- ✅ `packet.device.rssi` - Correct way to get signal strength

**Helper Function Example**:
```javascript
function extractPacketInfo(packet) {
  return {
    deviceId: packet.device.id,
    deviceName: packet.device.name,
    rssi: packet.device.rssi,
    sequence: packet.device.sequence_number,
    location: {
      lat: packet.location?.latitude,
      lng: packet.location?.longitude,
      timestamp: packet.location?.timestamp ?
        new Date(packet.location.timestamp * 1000) : null
    },
    payload: packet.device.payload
  };
}
```

### Webhook Management

- `POST /api/org/{org_id}/webhooks` - Register webhook endpoint
- `GET /api/org/{org_id}/webhooks` - List all webhooks
- `PATCH /api/org/{org_id}/webhooks/{webhook_id}` - Update webhook configuration
- `DELETE /api/org/{org_id}/webhooks/{webhook_id}` - Remove webhook

**Configuration**: URL, name, custom max batch size (10-1000 packets)

### Metrics

- `GET /api/org/{org_id}/api_metrics` - API request metrics with hourly breakdowns
- `GET /api/org/{org_id}/packet_metrics` - Packet volume metrics
- `GET /api/org/{org_id}/webhook_metrics` - Webhook delivery success/failure rates
- `GET /api/org/{org_id}/device_metrics` - Active and registered device counts

**Time ranges**: Configurable days back (1-365) and intervals (hour/day/month)

### Organization Management

- `GET /api/org/{org_id}` - Retrieve organization details
- `PATCH /api/org/{org_id}` - Update name, address, contact info, timezone

### User Management

- `GET /api/org/{org_id}/users` - List users (paginated)
- `POST /api/org/{org_id}/users` - Add user directly
- `PATCH /api/org/{org_id}/users/{user_id}` - Update role or name
- `DELETE /api/org/{org_id}/users/{user_id}` - Remove user

**Roles**: Admin, Member

### Invitation Management

- `GET /api/org/{org_id}/invitations` - List pending invitations
- `POST /api/org/{org_id}/invitations` - Invite new user
- `DELETE /api/org/{org_id}/invitations` - Revoke invitation

### API Key Management

- `GET /api/org/{org_id}/check` - Validate current API key
- `POST /api/org/{org_id}/key` - Provision new API key with scopes
- `GET /api/org/{org_id}/key` - List all API keys
- `PATCH /api/org/{org_id}/key/{key_id}` - Update key metadata
- `DELETE /api/org/{org_id}/key/{key_id}` - Revoke API key
- `GET /api/org/{org_id}/key_scopes` - List available authorization scopes

### Billing

- `GET /api/org/{org_id}/billing/invoices` - List recent invoices
- `GET /api/org/{org_id}/billing/invoices/{invoice_id}/pdf` - Download PDF
- `GET /api/org/{org_id}/billing/usage` - Daily/monthly active device usage

## Common Workflows

For detailed step-by-step guides, see [WORKFLOWS.md](./WORKFLOWS.md).

### Quick Reference

**Device Onboarding**:
1. Generate device credentials (device ID + encryption key)
2. Base64-encode the encryption key
3. POST to `/api/v2/org/{org_id}/devices`
4. Verify registration with GET request
5. Flash firmware to device with credentials

**Packet Streaming**:
1. GET `/api/org/{org_id}/packets` with time range
2. Process returned packets
3. Check for `Continuation-Token` header
4. If present, repeat request with token
5. Continue until no more tokens

**Webhook Setup**:
1. POST to `/api/org/{org_id}/webhooks` with endpoint URL
2. Configure batch size (default 100, max 1000)
3. Implement endpoint to receive POST requests
4. Validate `HTTP-X-HUBBLE-TOKEN` header
5. Return 200 status for successful receipt
6. Monitor delivery with webhook metrics

**API Key Rotation**:
1. POST new key with required scopes
2. Update applications with new key
3. Verify new key works with GET `/api/org/{org_id}/check`
4. DELETE old key

**Device Batch Update**:
1. GET devices to identify targets
2. Prepare updates (max 1000 devices)
3. PATCH `/api/org/{org_id}/devices` with array
4. Verify changes with GET requests

## Code Examples

For complete, runnable examples, see [EXAMPLES.md](./EXAMPLES.md).

### Quick Python Example: Batch Register Devices

```python
import requests

def batch_register_devices(api_token, org_id, n_devices=100, encryption="AES-128-CTR"):
    """
    Register multiple devices in a single API call.

    Args:
        api_token: Hubble API bearer token
        org_id: Organization UUID
        n_devices: Number of devices to create (1-1000)
        encryption: Encryption type (AES-256-CTR, AES-128-CTR, or NONE)

    Returns:
        List of device objects with generated credentials
    """
    url = f"https://api.hubble.com/api/v2/org/{org_id}/devices"
    headers = {
        "Authorization": f"Bearer {api_token}",
        "Content-Type": "application/json"
    }
    payload = {
        "n_devices": n_devices,
        "encryption": encryption
    }

    response = requests.post(url, json=payload, headers=headers)

    if response.status_code == 200:
        data = response.json()
        devices = data.get("devices", [])
        print(f"Created {len(devices)} devices")
        return devices
    else:
        print(f"Error: {response.status_code} - {response.text}")
        return []

# Example usage
devices = batch_register_devices("your-api-token", "your-org-id", n_devices=10)
for device in devices:
    print(f"Device ID: {device['device_id']}, Key: {device['key']}")
```

### Quick Python Example: Batch Delete Devices

```python
import requests

def batch_delete_devices(api_token, org_id, device_ids):
    """
    Delete multiple devices in a single API call.

    Args:
        api_token: Hubble API bearer token
        org_id: Organization UUID
        device_ids: List of device UUIDs to delete (up to 1000)

    Returns:
        Dictionary with deletion results
    """
    url = f"https://api.hubble.com/api/org/{org_id}/devices"
    headers = {
        "Authorization": f"Bearer {api_token}",
        "Content-Type": "application/json"
    }
    payload = {"device_ids": device_ids}

    response = requests.delete(url, json=payload, headers=headers)

    if response.status_code in [200, 204]:
        result = response.json() if response.text else {"deleted": len(device_ids)}
        print(f"Deleted {result.get('deleted', len(device_ids))} devices")
        return result
    else:
        print(f"Error: {response.status_code} - {response.text}")
        return {"deleted": 0, "failed": len(device_ids)}
```

### Quick curl Example: List Devices

```bash
curl -X GET "https://api.hubble.com/api/org/YOUR_ORG_ID/devices" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

## Pagination

### Continuation Token Pattern

For endpoints returning large datasets, the API uses continuation tokens instead of offset-based pagination:

1. Make initial request to endpoint
2. Process returned results (max 1000 items)
3. Check response headers for `Continuation-Token`
4. If present, make new request with `Continuation-Token` header set
5. Repeat until `Continuation-Token` is absent

**Example**:
```python
import requests

def stream_all_packets(api_token, org_id):
    url = f"https://api.hubble.com/api/org/{org_id}/packets"
    headers = {"Authorization": f"Bearer {api_token}"}

    all_packets = []
    continuation_token = None

    while True:
        if continuation_token:
            headers["Continuation-Token"] = continuation_token

        response = requests.get(url, headers=headers)
        data = response.json()
        all_packets.extend(data["packets"])

        continuation_token = response.headers.get("Continuation-Token")
        if not continuation_token:
            break

    return all_packets
```

## Data Encoding

### Base64 Encoding Requirements

**Device Keys**: Encryption keys are binary data and must be Base64-encoded for JSON transport:

```python
import base64

# Binary key (32 bytes)
binary_key = b'\x1a\x2b\x3c...'

# Encode for API
encoded_key = base64.b64encode(binary_key).decode('utf-8')

# Decode from API response
decoded_key = base64.b64decode(encoded_key)
```

**Packet Payloads**: Similarly, packet payloads returned by the API are Base64-encoded:

```python
# Decode packet payload
packet_payload_b64 = "SGVsbG8gV29ybGQ="
decoded_payload = base64.b64decode(packet_payload_b64)
```

**Common Error**: Forgetting to Base64-encode device keys results in 400 errors with message "Invalid deviceKey format".

## Error Handling

For comprehensive troubleshooting, see [TROUBLESHOOTING.md](./TROUBLESHOOTING.md).

### HTTP Status Codes

- **200 OK**: Successful request
- **201 Created**: Resource successfully created
- **400 Bad Request**: Invalid request format or parameters
- **401 Unauthorized**: Missing or invalid API token
- **403 Forbidden**: Insufficient scopes for operation
- **404 Not Found**: Resource does not exist
- **429 Too Many Requests**: Rate limit exceeded
- **500 Internal Server Error**: Server-side error (retry with backoff)

### Common Error Patterns

**401 Unauthorized**:
- Missing `Authorization` header
- Expired or revoked API token
- Malformed token format

**403 Forbidden**:
- API key lacks required scope
- Attempting to access resources from different organization

**400 Bad Request (Device Registration)**:
- Device key not Base64-encoded
- Missing required fields (name, dev_eui, device_key)
- Invalid device EUI format

**429 Rate Limit Exceeded**:
- Too many requests in time window
- Implement exponential backoff
- Check `Retry-After` header

**404 Not Found**:
- Invalid organization ID
- Device ID doesn't exist
- Webhook ID not found

### Request Tracing

All responses include `X-Request-ID` header for debugging:

```
X-Request-ID: 550e8400-e29b-41d4-a716-446655440000
```

Include this ID when contacting support for faster issue resolution.

## Best Practices

### Security

1. **Never commit API keys** to version control
2. **Store tokens securely** (environment variables, secret managers)
3. **Use minimal scopes** - Only request permissions you need
4. **Rotate keys regularly** - Implement key rotation workflow
5. **Validate webhook tokens** - Check `HTTP-X-HUBBLE-TOKEN` header

### Performance

1. **Use batch operations** - Update up to 1000 devices in single request
2. **Implement caching** - Cache device lists, reduce repeated requests
3. **Stream with continuation tokens** - Don't try to fetch all data at once
4. **Handle rate limits gracefully** - Exponential backoff, not retry loops
5. **Use webhooks for real-time** - Avoid polling for new packets

### Data Handling

1. **Base64-encode binary data** - Device keys and some payloads
2. **Validate data before API calls** - Check formats locally first
3. **Handle pagination correctly** - Use continuation tokens, not offsets
4. **Set appropriate time ranges** - Default 7 days, adjust as needed
5. **Store device keys securely** - Encryption keys are sensitive

### Monitoring

1. **Track API metrics** - Use `/api/org/{org_id}/api_metrics`
2. **Monitor webhook health** - Check delivery success rates
3. **Set up alerting** - For failed webhooks or API errors
4. **Log request IDs** - Include `X-Request-ID` in application logs
5. **Track rate limit usage** - Monitor `X-RateLimit-*` headers

### Error Recovery

1. **Implement retry logic** - With exponential backoff
2. **Handle partial failures** - In batch operations
3. **Log all errors** - With context and request IDs
4. **Graceful degradation** - Fall back to cached data
5. **Alert on repeated failures** - Don't silently fail

## Webhook Validation

### Webhook Request Format

Hubble sends POST requests to your webhook endpoint:

```http
POST /your/webhook/endpoint HTTP/1.1
Host: your-domain.com
Content-Type: application/json
HTTP-X-HUBBLE-TOKEN: your-webhook-secret

{
  "packets": [
    {
      "device_id": "device123",
      "payload": "SGVsbG8gV29ybGQ=",
      "timestamp": "2024-01-15T10:30:00Z",
      "metadata": { ... }
    }
  ]
}
```

### Validating Webhook Requests

Always validate the `HTTP-X-HUBBLE-TOKEN` header to prevent spoofing:

```python
from flask import Flask, request, abort

app = Flask(__name__)
WEBHOOK_SECRET = "your-webhook-secret"

@app.route('/webhook', methods=['POST'])
def handle_webhook():
    # Validate token
    token = request.headers.get('HTTP-X-HUBBLE-TOKEN')
    if token != WEBHOOK_SECRET:
        abort(403)

    # Process packets
    data = request.json
    packets = data.get('packets', [])

    for packet in packets:
        process_packet(packet)

    return '', 200
```

### Webhook Response

- Return **200 OK** for successful receipt
- Hubble considers 2xx status codes as success
- Non-2xx responses trigger automatic retries
- Retries use exponential backoff

## Getting Help

### Resources

- **API Documentation**: https://docs.hubble.com/docs/api-specification/hubble-cloud-api
- **OpenAPI Spec**: See [resources/hubble-openapi.yaml](./resources/hubble-openapi.yaml)
- **Developer Portal**: https://hubble.com/developers
- **Support Email**: [email protected]

### Debugging Checklist

When encountering issues:

1. ✓ Check API key has required scopes
2. ✓ Verify organization ID is correct
3. ✓ Confirm Bearer token format in header
4. ✓ Validate Base64 encoding for device keys
5. ✓ Check rate limits aren't exceeded
6. ✓ Review error response body for details
7. ✓ Include `X-Request-ID` in support requests

### Quick Diagnostics

**Test API Key**:
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.hubble.com/api/org/YOUR_ORG_ID/check
```

**Check Scopes**:
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://api.hubble.com/api/org/YOUR_ORG_ID/key_scopes
```

## Summary

The Hubble Cloud API provides comprehensive access to device management, packet streaming, webhook configuration, and organization administration. Key points to remember:

- **Authentication**: Bearer tokens with granular scopes
- **Rate Limits**: 3/sec per endpoint, 15/sec org-wide
- **Pagination**: Continuation tokens for streaming
- **Encoding**: Base64 for binary data (keys, payloads)
- **Webhooks**: Push notifications with validation
- **Best Practices**: Batch operations, caching, exponential backoff

For detailed implementations, refer to:
- [API_REFERENCE.md](./API_REFERENCE.md) - Complete endpoint documentation
- [WORKFLOWS.md](./WORKFLOWS.md) - Step-by-step implementation guides
- [EXAMPLES.md](./EXAMPLES.md) - Runnable code examples
- [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) - Error resolution guides
