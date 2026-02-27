---
name: dcyfr-security
description: CodeQL suppressions, security vulnerability troubleshooting, and security best practices
compatibility: opencode
metadata:
  audience: developers
  workflow: security
  category: security
---

## What I do

I guide security practices and CodeQL false positive suppression:

- **CodeQL suppressions** with LGTM syntax
- **Security vulnerability troubleshooting** (SSRF, CWE-918)
- **False positive patterns** and verification
- **Security testing** patterns

## When to use me

✅ **Use this skill when:**
- Suppressing CodeQL false positives
- Investigating security alerts
- Implementing security fixes
- Reviewing security-sensitive code

❌ **Don't use this skill for:**
- General code review (use dcyfr-validation)
- Performance issues (separate concern)
- Non-security bugs

## CodeQL Suppression Syntax

### Basic Suppression

```typescript
// lgtm[js/path-injection]
const filePath = path.join(baseDir, userInput);
```

### With Justification

```typescript
// lgtm[js/sql-injection]: Input validated by schema
const query = `SELECT * FROM users WHERE id = ${userId}`;
```

### Multiple Alerts

```typescript
// lgtm[js/path-injection,js/sql-injection]
const result = await processInput(userInput);
```

## Common False Positives

### 1. Path Injection (Safe)

```typescript
// lgtm[js/path-injection]: userInput validated against allowlist
const filePath = path.join(PUBLIC_DIR, userInput);
```

### 2. SQL Injection (Parameterized)

```typescript
// lgtm[js/sql-injection]: Using parameterized query
await db.query('SELECT * FROM users WHERE id = $1', [userId]);
```

### 3. XSS (React Auto-Escapes)

```typescript
// lgtm[js/xss]: React auto-escapes JSX content
<div>{userContent}</div>
```

## Security Testing Patterns

### API Endpoint Security Test

```typescript
describe('/api/secure-endpoint', () => {
  it('rejects unauthorized requests', async () => {
    const response = await fetch('/api/secure-endpoint');
    expect(response.status).toBe(401);
  });

  it('validates input', async () => {
    const response = await fetch('/api/secure-endpoint', {
      method: 'POST',
      body: JSON.stringify({ malicious: '<script>alert(1)</script>' }),
    });
    expect(response.status).toBe(400);
  });
});
```

## SSRF Prevention

```typescript
// ✅ CORRECT: Validate URLs before fetching
const allowedDomains = ['api.trusted.com'];
const url = new URL(userProvidedUrl);

if (!allowedDomains.includes(url.hostname)) {
  throw new Error('Invalid domain');
}

const response = await fetch(url.toString());
```

## Related Documentation

- **CodeQL suppressions**: `.github/agents/patterns/CODEQL_SUPPRESSIONS.md`
- **Security troubleshooting**: `.github/agents/patterns/SECURITY_VULNERABILITY_TROUBLESHOOTING.md`

## Approval Gates

Security compliance is **STRICT** (hard block):

- ❌ Cannot suppress without justification
- ❌ Cannot merge with HIGH/CRITICAL alerts
- ✅ Must verify suppressions are valid
