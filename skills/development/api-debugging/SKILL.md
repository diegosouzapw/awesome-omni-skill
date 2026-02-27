---
name: api-debugging
description: Helps debug API requests, responses, status codes, and network issues. Auto-activates when users ask about API errors, HTTP status codes, request failures, or API testing. Provides diagnostic insights and resolution steps.
allowed-tools: Read, Grep, Bash
---

# API Debugging Skill

## Purpose & Activation

**What it does:** Provides diagnostic insights for API-related issues, HTTP errors, and network debugging.

**When it activates:**
- "API request failed"
- "Why am I getting 404?"
- "Debug this API call"
- "HTTP status code meaning"
- "API authentication error"

**How it helps:**
- Interprets HTTP status codes
- Suggests fixes for common API errors
- Validates API request/response structure
- Checks authentication issues
- Tests API endpoints

**JIT Help Available:**
For quick reference on common API errors and debugging commands, see: **[Cheat Sheet](./cheatsheet.md)**

---

## Key Operations

### Operation 1: HTTP Status Code Interpretation

**User Query:**
> "What does status code 403 mean?"
> "Why am I getting 401?"
> "Explain this error code"

**Confidence:** 🟢 High (0.95)
- HTTP status codes are well-defined standards
- Exact keyword match: status code number
- Deterministic interpretation

**Skill Response:**
```
📡 HTTP Status Code: 403 Forbidden

**Meaning:**
The server understood your request but refuses to authorize it.

**Common Causes:**
1. **Authentication missing** - No credentials provided
2. **Authorization failed** - Valid credentials but insufficient permissions
3. **IP blocked** - Your IP address is denied access
4. **API rate limit** - Too many requests (check X-RateLimit headers)
5. **Invalid API key** - Key expired or revoked

**Quick Fixes:**
1. Check authorization header: `Authorization: Bearer <token>`
2. Verify API key hasn't expired
3. Check user permissions for this endpoint
4. Review rate limit status
5. Confirm IP is whitelisted (if applicable)

**Debug Command:**
```bash
# Test with verbose output
curl -v -H "Authorization: Bearer YOUR_TOKEN" https://api.example.com/endpoint

# Check response headers
curl -I https://api.example.com/endpoint
```
```

**Why This Matters:**
- 403 vs 401 distinction is critical (authorization vs authentication)
- Misdiagnosing means wasting time on wrong fix
- Status codes reveal server-side vs client-side issues
- **Quick win:** Check authorization header format right now

**Next Step:** Test API with curl command to see full error details

### Operation 2: API Request Validation

**User Query:**
> "My API request isn't working"
> "Debug this fetch call"
> "API returns unexpected response"

**Confidence:** 🟡 Medium (0.70)
- Requires seeing actual request code
- Multiple possible failure points
- Context-dependent debugging

**Skill Response:**
```
🔍 API Request Debugging Checklist

**1. Request Structure:**
- ✓ Correct HTTP method? (GET/POST/PUT/DELETE)
- ✓ Valid URL format? (https://api.example.com/v1/resource)
- ✓ Headers included? (Content-Type, Authorization)
- ✓ Body format correct? (JSON, form-data, etc.)

**2. Authentication:**
- ✓ API key/token present in headers?
- ✓ Token hasn't expired?
- ✓ Correct authentication scheme? (Bearer, Basic, API-Key)

**3. Request Body:**
- ✓ JSON syntax valid? (use json.tool to validate)
- ✓ Required fields included?
- ✓ Data types correct? (string vs number vs boolean)

**4. Network:**
- ✓ Internet connection working?
- ✓ Firewall/proxy not blocking?
- ✓ CORS configured (if browser request)?

**Debug Steps:**
```bash
# 1. Test with curl first (eliminates code issues)
curl -X POST https://api.example.com/endpoint \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"key":"value"}'

# 2. Validate JSON body
echo '{"your":"json"}' | python -m json.tool

# 3. Check DNS resolution
nslookup api.example.com

# 4. Test connectivity
ping api.example.com
```

**Common Issues:**
- **Missing Content-Type** → Server rejects request
- **Wrong HTTP method** → 405 Method Not Allowed
- **CORS error** → Add CORS headers or use proxy
- **SSL error** → Certificate validation issue
```

**Why This Matters:**
- Systematic checklist prevents missing obvious issues
- curl eliminates language-specific bugs (tests API directly)
- 80% of API errors are client-side (your request, not their server)
- **Quick win:** Run curl command to isolate issue to code vs API

**Next Step:** Share curl output or request code for specific diagnosis

### Operation 3: API Response Parsing Issues

**User Query:**
> "API response is undefined"
> "Can't parse JSON response"
> "Response data is null"

**Confidence:** 🟢 High (0.85)
- Common parsing error patterns
- Deterministic validation steps
- Clear fix recommendations

**Skill Response:**
```
🔧 API Response Parsing Debug

**Step 1: Check Response Status**
```javascript
fetch('https://api.example.com/data')
  .then(response => {
    console.log('Status:', response.status); // Check this first!
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return response.json();
  })
```

**Step 2: Validate Response Body**
```javascript
// Log raw response before parsing
fetch('https://api.example.com/data')
  .then(response => response.text()) // Don't parse yet
  .then(text => {
    console.log('Raw response:', text); // Inspect actual content
    return JSON.parse(text); // Parse manually to catch errors
  })
```

**Common Issues:**

**Issue 1: Empty Response Body**
```
Problem: response.json() fails because body is empty
Fix: Check if response.status is 204 (No Content) or 304 (Not Modified)
```

**Issue 2: HTML Error Page (Not JSON)**
```
Problem: API returns HTML error page instead of JSON
Symptom: "Unexpected token < in JSON"
Fix: Log response.text() to see actual HTML error
```

**Issue 3: Wrong Content-Type**
```
Problem: Server sends JSON but headers say text/plain
Fix: Use response.text() then JSON.parse() manually
```

**Issue 4: Malformed JSON**
```
Problem: Server returns invalid JSON
Fix: Validate with: echo 'RESPONSE' | python -m json.tool
```

**Debug Command:**
```bash
# Test response in terminal
curl -v https://api.example.com/endpoint 2>&1 | less

# Save response to file for inspection
curl https://api.example.com/endpoint > response.json
cat response.json | python -m json.tool
```
```

**Why This Matters:**
- "undefined" usually means successful request but wrong parsing approach
- Checking status before parsing prevents cryptic errors
- Raw response inspection reveals server-side issues (HTML errors, wrong format)
- **Quick win:** Add console.log('Status:', response.status) before parsing

**Next Step:** Log raw response to identify if it's format issue or network issue

---

## Common API Error Patterns

### Pattern 1: Authentication Errors (401/403)
```
401 Unauthorized → Missing or invalid credentials
403 Forbidden → Valid credentials but no permission

Fix order:
1. Verify credentials exist and aren't expired
2. Check header format: Authorization: Bearer <token>
3. Confirm user has required permissions
4. Test with API provider's example credentials
```

### Pattern 2: Rate Limiting (429)
```
429 Too Many Requests → Exceeded API rate limit

Headers to check:
- X-RateLimit-Limit: Total requests allowed
- X-RateLimit-Remaining: Requests left
- X-RateLimit-Reset: When limit resets (Unix timestamp)

Fix:
- Implement exponential backoff
- Cache responses to reduce requests
- Upgrade API plan if needed
```

### Pattern 3: Server Errors (500-599)
```
500 Internal Server Error → Server-side bug
502 Bad Gateway → Proxy/load balancer issue
503 Service Unavailable → Server overloaded or maintenance
504 Gateway Timeout → Upstream service too slow

Your action:
- NOT your fault (server-side issue)
- Check API status page
- Implement retry logic with backoff
- Contact API support if persistent
```

---

## HTTP Status Code Quick Reference

**2xx Success:**
- 200 OK - Request succeeded
- 201 Created - Resource created successfully
- 204 No Content - Success but no response body

**3xx Redirection:**
- 301 Moved Permanently - Resource moved, update URL
- 304 Not Modified - Use cached version

**4xx Client Errors (Your Problem):**
- 400 Bad Request - Invalid request syntax
- 401 Unauthorized - Authentication required
- 403 Forbidden - No permission
- 404 Not Found - Resource doesn't exist
- 405 Method Not Allowed - Wrong HTTP method
- 429 Too Many Requests - Rate limited

**5xx Server Errors (Their Problem):**
- 500 Internal Server Error - Server bug
- 502 Bad Gateway - Proxy error
- 503 Service Unavailable - Server down
- 504 Gateway Timeout - Server too slow

---

## Debugging Tools

### curl (Command Line)
```bash
# Basic GET request
curl https://api.example.com/endpoint

# POST with JSON body
curl -X POST https://api.example.com/endpoint \
  -H "Content-Type: application/json" \
  -d '{"key":"value"}'

# With authentication
curl -H "Authorization: Bearer TOKEN" https://api.example.com/endpoint

# Verbose output (see full request/response)
curl -v https://api.example.com/endpoint

# Save response to file
curl -o response.json https://api.example.com/endpoint
```

### Browser DevTools
```
Network tab → Click request → Check:
- Status code
- Request headers (Authorization, Content-Type)
- Response headers
- Response body (Preview or Response tab)
- Timing (to identify slow requests)
```

### Postman/Insomnia
```
Benefits:
- Visual interface for API testing
- Save requests for reuse
- Environment variables for tokens
- Auto-generate code snippets
```

---

## Token Efficiency Analysis

### Without This Skill (900 tokens per debug session)
**Process:**
1. User: "Why 403 error?" (50 tokens)
2. Claude: "Let me check HTTP standards..." (150 tokens)
3. Explain status codes (200 tokens)
4. Ask for request details (100 tokens)
5. User provides details (150 tokens)
6. Diagnose specific issue (250 tokens)

**Total: ~900 tokens**

### With This Skill (350 tokens per debug session)
**Process:**
1. User: "Why 403 error?" (50 tokens)
2. Skill activates with pre-compiled knowledge (100 tokens)
3. Provides interpretation + fixes immediately (200 tokens)

**Total: ~350 tokens**

**Savings:** 550 tokens per debug session (61% reduction)

**Frequency:** Developers debug APIs 5-10 times per week
- Weekly: 2,750-5,500 tokens saved
- Monthly: 11,000-22,000 tokens saved (~$0.33-$0.66/month)

---

## Best Practices

### For Users
1. **Test with curl first** - Eliminates code-specific issues
2. **Log raw responses** - Before parsing, inspect actual content
3. **Check status codes** - Before assuming success
4. **Read API docs** - Authentication, rate limits, error codes
5. **Use API status pages** - Check if service is down

### For Claude (Using This Skill)
1. **Ask for status code** - First question in debugging
2. **Provide curl examples** - Universal testing method
3. **Distinguish 4xx vs 5xx** - Client vs server responsibility
4. **Link to API docs** - When available
5. **Suggest logging** - Before parsing

---

## See Also

- **testing-workflow skill** - For API integration test results
- **HTTP Status Codes:** https://httpstatuses.com
- **MDN HTTP Docs:** https://developer.mozilla.org/en-US/docs/Web/HTTP
- **Postman Learning Center:** https://learning.postman.com

---

**Skill Version:** 3.5.0
**Last Updated:** 2025-12-15
**Target Audience:** Backend/frontend developers, API consumers
**Maintained By:** claude-config-template project
