---
name: api-test
description: Test API endpoints for MCP Finance app including authentication, data validation, and error handling. Use when testing APIs, debugging endpoints, or when user mentions API testing, endpoint testing, or API issues.
allowed-tools: Read, Bash(curl *), Bash(node *)
---

# API Testing Skill for nextjs-mcp-finance

Comprehensive guide for testing API endpoints in the MCP Finance Next.js application.

## Quick Start

### 1. Start Dev Server

```bash
cd nextjs-mcp-finance
npm run dev
```

Wait for the server to start (usually at http://localhost:3000).

### 2. Test Health Endpoint

```bash
curl http://localhost:3000/api/health
```

Expected Response:

```json
{
  "status": "ok",
  "timestamp": "2024-01-18T..."
}
```

## API Endpoint Categories

### Authentication Endpoints

- `POST /api/auth/sign-up` - User registration
- `POST /api/auth/sign-in` - User login
- `POST /api/auth/sign-out` - User logout
- `GET /api/auth/me` - Current user info

### Stock Endpoints

- `GET /api/stocks` - List all stocks
- `GET /api/stocks/[symbol]` - Get specific stock
- `POST /api/stocks/favorite` - Add favorite stock
- `DELETE /api/stocks/favorite/[id]` - Remove favorite

### Transaction Endpoints

- `GET /api/transactions` - List transactions
- `POST /api/transactions` - Create transaction
- `GET /api/transactions/[id]` - Get transaction details

### Webhook Endpoints

- `POST /api/webhooks/clerk` - Clerk webhooks
- `POST /api/webhooks/stripe` - Stripe webhooks

## Testing Protected Endpoints

### Test without authentication (should fail)

```bash
curl -X GET http://localhost:3000/api/transactions \
  -H "Content-Type: application/json"
```

Expected: `Unauthorized (401)`

### Test with authentication

First sign in via the browser, then get your session cookie:

```bash
curl -X GET http://localhost:3000/api/transactions \
  -H "Content-Type: application/json" \
  -H "Cookie: __session=YOUR_SESSION_COOKIE"
```

## Testing POST Endpoints

### Create Transaction Example

```bash
curl -X POST http://localhost:3000/api/transactions \
  -H "Content-Type: application/json" \
  -H "Cookie: __session=YOUR_SESSION_COOKIE" \
  -d '{
    "symbol": "AAPL",
    "quantity": 10,
    "price": 150.00,
    "type": "buy"
  }'
```

Expected Response:

```json
{
  "success": true,
  "data": {
    "id": "...",
    "symbol": "AAPL",
    "quantity": 10,
    "price": "150.00",
    "type": "buy",
    "createdAt": "..."
  }
}
```

## Error Handling Tests

### Invalid Data

```bash
curl -X POST http://localhost:3000/api/transactions \
  -H "Content-Type: application/json" \
  -H "Cookie: __session=YOUR_SESSION_COOKIE" \
  -d '{
    "symbol": "INVALID",
    "quantity": -10
  }'
```

Expected: `{ "success": false, "error": "Invalid transaction data" }`

### Missing Required Fields

```bash
curl -X POST http://localhost:3000/api/transactions \
  -H "Content-Type: application/json" \
  -H "Cookie: __session=YOUR_SESSION_COOKIE" \
  -d '{}'
```

Expected: `400` Bad Request with validation errors

## Rate Limiting Tests

### Send Multiple Rapid Requests

```bash
for i in {1..20}; do
  curl -X GET http://localhost:3000/api/stocks
done
```

Look for `429 (Too Many Requests)` responses if rate limiting is implemented.

## Automated Testing Script

Create `test-api.js` in the nextjs-mcp-finance directory:

```javascript
const BASE_URL = "http://localhost:3000";

async function testEndpoint(method, path, data = null, headers = {}) {
  const options = {
    method,
    headers: {
      "Content-Type": "application/json",
      ...headers,
    },
  };

  if (data) {
    options.body = JSON.stringify(data);
  }

  try {
    const response = await fetch(`${BASE_URL}${path}`, options);
    const json = await response.json();

    console.log(`✓ ${method} ${path}: ${response.status}`);
    console.log("  Response:", JSON.stringify(json, null, 2));

    return { status: response.status, data: json };
  } catch (error) {
    console.error(`✗ Error testing ${path}:`, error.message);
    return { status: 0, error: error.message };
  }
}

async function runTests() {
  console.log("🧪 Starting API Tests\n");

  // Test health endpoint
  await testEndpoint("GET", "/api/health");

  // Test unauthenticated request (should fail)
  console.log("\n📋 Testing authentication...");
  await testEndpoint("GET", "/api/transactions");

  // Add more tests here based on your endpoints
  console.log("\n✅ Tests complete");
}

runTests();
```

**Run it:**

```bash
node test-api.js
```

## Response Format Validation Checklist

For each endpoint, verify:

- [ ] **Correct Status Codes**
  - 200 for successful GET
  - 201 for successful POST (created)
  - 400 for bad request
  - 401 for unauthorized
  - 404 for not found
  - 500 for server error

- [ ] **Consistent Response Format**

  ```json
  {
    "success": boolean,
    "data": object | array (if success: true),
    "error": string (if success: false)
  }
  ```

- [ ] **Authentication Enforced**
  - Protected endpoints check auth
  - Returns 401 if not authenticated

- [ ] **Input Validation**
  - Invalid data is rejected
  - Clear error messages provided
  - SQL injection prevention in place

- [ ] **Error Handling**
  - No stack traces exposed to users
  - User-friendly error messages
  - Proper logging in server

- [ ] **Performance**
  - Simple queries < 500ms
  - Complex queries < 2s

## Troubleshooting Common Issues

### CORS Errors

```
Access-Control-Allow-Origin error
```

**Solution:** Check your API route headers:

```typescript
export async function GET(request: Request) {
  return Response.json(data, {
    headers: {
      "Access-Control-Allow-Origin": "*",
    },
  });
}
```

### 500 Internal Server Error

**Debug Steps:**

1. Check Next.js dev server console for error messages
2. Verify environment variables are set
3. Check database connectivity
4. Review recent code changes

```bash
cd nextjs-mcp-finance
npm run dev
# Watch for errors in console output
```

### Slow Response Times

**Profile with curl:**

```bash
curl -w "@curl-format.txt" -o /dev/null -s http://localhost:3000/api/stocks
```

**curl-format.txt:**

```
time_namelookup:  %{time_namelookup}\n
time_connect:  %{time_connect}\n
time_appconnect:  %{time_appconnect}\n
time_starttransfer:  %{time_starttransfer}\n
time_total:  %{time_total}\n
```

### 401 Unauthorized on Protected Endpoints

1. Ensure Clerk is configured in .env
2. Verify session cookie is being set correctly
3. Check auth middleware in API route
4. Look at browser dev tools Network tab to see cookies

## Integration with Other Tests

- **E2E Tests** (`npm run test:e2e`): Test complete user flows through UI
- **API Tests** (this skill): Test endpoints directly with curl/fetch
- **Unit Tests** (`npm run test`): Test individual functions

All should pass before deployment.

## Quick Command Reference

```bash
# Health check
curl http://localhost:3000/api/health

# List stocks (public endpoint)
curl http://localhost:3000/api/stocks | jq

# Get specific stock
curl http://localhost:3000/api/stocks/AAPL | jq

# Test with verbose output
curl -v http://localhost:3000/api/health

# Save response to file
curl http://localhost:3000/api/stocks > response.json

# Test with custom headers
curl -H "Authorization: Bearer TOKEN" http://localhost:3000/api/stocks

# Test with POST data
curl -X POST http://localhost:3000/api/endpoint \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

## Pro Tips

- Use `jq` to format JSON responses: `curl ... | jq`
- Use Postman or Insomnia for interactive testing with saved requests
- Keep curl commands in a test file for repeatability
- Log API responses when debugging issues
- Test both success and error cases for each endpoint
- Verify error messages are helpful but don't expose sensitive info

## Next Steps After API Testing

1. If all APIs pass, run full test suite: `npm run test`
2. Run E2E tests: `npm run test:e2e`
3. Check code quality: `npm run lint`
4. Build for production: `npm run build`
5. Use deployment-checklist skill before deploying

---

For more information about the project structure and setup, see the codebase-navigator skill.
