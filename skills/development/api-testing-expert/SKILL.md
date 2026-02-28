---
name: API Testing Expert
description: API testing - Postman, REST clients, contract testing, mock servers
allowed-tools: Read, Edit, Bash, WebFetch
model: sonnet
---

# API Testing Expert

Comprehensive API testing patterns for REST, GraphQL, and more.

## Tools Comparison

| Tool | Best For | Free Tier | Pricing |
|------|----------|-----------|---------|
| Postman | Teams | 3 users | $14/user/mo |
| Bruno | Local-first | Unlimited | Free |
| Hoppscotch | Open source | Unlimited | Free |
| REST Client (VS Code) | In-editor | Unlimited | Free |

## Bruno (Recommended for Vibe Coders)

Git-friendly, no account needed.

### Install
```bash
brew install bruno
```

### Collection Structure
```
api-tests/
├── bruno.json
├── environments/
│   ├── local.bru
│   └── production.bru
└── requests/
    ├── auth/
    │   └── login.bru
    └── users/
        └── get-user.bru
```

### Request File
```bru
meta {
  name: Get User
  type: http
  seq: 1
}

get {
  url: {{baseUrl}}/users/{{userId}}
  body: none
  auth: bearer {{token}}
}

tests {
  test("status is 200", function() {
    expect(res.status).to.equal(200);
  });

  test("has user data", function() {
    expect(res.body.id).to.exist;
  });
}
```

## REST Client (VS Code)

### HTTP File
```http
### Login
POST {{baseUrl}}/auth/login
Content-Type: application/json

{
  "email": "test@example.com",
  "password": "secret"
}

### Get User (use token from login)
GET {{baseUrl}}/users/me
Authorization: Bearer {{$dotenv TOKEN}}
```

## Postman CLI (Newman)

### Run in CI
```bash
npx newman run collection.json \
  -e environment.json \
  --reporters cli,json \
  --reporter-json-export results.json
```

### GitHub Actions
```yaml
- name: Run API Tests
  run: |
    npx newman run ./postman/collection.json \
      -e ./postman/ci-environment.json \
      --bail
```

## Contract Testing (Pact)

### Consumer Test
```javascript
const { Pact } = require('@pact-foundation/pact');

const provider = new Pact({
  consumer: 'Frontend',
  provider: 'UserService',
});

describe('User API', () => {
  it('returns user by id', async () => {
    await provider.addInteraction({
      state: 'user 1 exists',
      uponReceiving: 'a request for user 1',
      withRequest: {
        method: 'GET',
        path: '/users/1',
      },
      willRespondWith: {
        status: 200,
        body: {
          id: 1,
          name: like('John'),
        },
      },
    });
  });
});
```

## Mock Servers

### MSW (Mock Service Worker)
```typescript
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

const handlers = [
  http.get('/api/users/:id', ({ params }) => {
    return HttpResponse.json({
      id: params.id,
      name: 'Test User',
    });
  }),
];

export const server = setupServer(...handlers);
```

### Prism (OpenAPI Mock)
```bash
# Mock from OpenAPI spec
npx @stoplight/prism-cli mock openapi.yaml
```

## Quick API Test Pattern

```bash
# Simple curl test script
curl -sf "$API_URL/health" || exit 1
curl -sf -H "Authorization: Bearer $TOKEN" "$API_URL/users/me" | jq .
```

Use when: API testing, contract testing, mock servers, CI integration
