---
name: hive-auth
description: How authentication works in Hive framework
globs:
  - src/resources/auth/**/*.js
alwaysApply: false
---

# Authentication

Hive provides auth infrastructure but **no built-in auth endpoints**. You implement login/signup yourself.

## Built-in (in .hive/)

- `tokens` collection - stores access tokens
- `attachUser` middleware - loads user from token
- `isAuthorized` middleware - requires auth
- `allowNoAuth` middleware - skips auth

## How It Works

1. Token extracted from cookie `access_token` or `Authorization: Bearer` header
2. Token looked up in `tokens` collection
3. User loaded from `users` collection
4. User attached to `ctx.state.user`

## Token Schema (built-in)

```javascript
{
  _id: string,
  user: { _id: string },
  token: string,
  metadata: object (optional),
}
```

## Implement Auth Endpoints

**1. Create token helper:**
```javascript
// src/resources/auth/methods/createToken.js
import db from 'db';
import crypto from 'crypto';

const tokenService = db.services.tokens;

export default async (ctx, { userId, metadata }) => {
  const token = crypto.randomBytes(32).toString('hex');
  
  await tokenService.create({
    token,
    user: { _id: userId },
    ...(metadata && { metadata }),
  });
  
  ctx.cookies.set('access_token', token, {
    httpOnly: false,
    expires: new Date(Date.now() + 10 * 365 * 24 * 60 * 60 * 1000),
  });
  
  return { token };
};
```

**2. Login endpoint:**
```javascript
// src/resources/auth/endpoints/login.js
import { z } from 'zod';
import db from 'db';
import bcrypt from 'bcrypt';
import createToken from '../methods/createToken';

export const handler = async (ctx) => {
  const { email, password } = ctx.validatedData;
  
  const user = await db.services.users.findOne(
    { email },
    { isIncludeSecureFields: true }
  );
  ctx.assert(user, 401, 'Invalid credentials');
  
  const valid = await bcrypt.compare(password, user.password);
  ctx.assert(valid, 401, 'Invalid credentials');
  
  const { token } = await createToken(ctx, { userId: user._id });
  return { user, token };
};

export const middlewares = ['allowNoAuth'];
export const endpoint = { url: '/login', method: 'post' };
export const requestSchema = z.object({
  email: z.string().email(),
  password: z.string(),
});
```

**3. Logout endpoint:**
```javascript
// src/resources/auth/endpoints/logout.js
import { z } from 'zod';
import db from 'db';

export const handler = async (ctx) => {
  await db.services.tokens.remove({ token: ctx.state.accessToken });
  ctx.cookies.set('access_token', null);
  return { success: true };
};

export const endpoint = { url: '/logout', method: 'post' };
export const requestSchema = z.object({});
```

## Users Schema (add password)

```javascript
// src/resources/users/users.schema.js
password: z.coerce.string().nullable().optional(),

// Hide from responses
export const secureFields = ['password'];
```

## Client Usage

**Browser (cookies):**
```javascript
await fetch('/auth/login', { method: 'POST', credentials: 'include', body });
```

**API (header):**
```javascript
await fetch('/tasks', { headers: { Authorization: `Bearer ${token}` } });
```
