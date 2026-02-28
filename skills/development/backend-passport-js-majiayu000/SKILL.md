---
name: backend-passport-js
description: Authentication middleware for Express.js and Node.js applications. Use when building Express APIs that need JWT authentication, OAuth, or custom auth strategies. Provides 500+ authentication strategies. Choose Passport.js over Auth.js for Express backends, pure API servers, or when you need maximum control over auth flow.
allowed-tools: Read, Edit, Write, Bash (*)
---

# Passport.js (Express Authentication)

## Overview

Passport.js is the de-facto standard authentication middleware for Express.js. It uses a strategy pattern with 500+ available strategies for different auth mechanisms.

**Version**: passport@0.7.x, passport-jwt@4.x  
**Use case**: Express.js applications, custom auth flows, pure APIs

**Key Benefit**: Maximum flexibility, works with any Express app, battle-tested in production.

## When to Use This Skill

✅ **Use Passport.js when:**
- Building Express.js REST APIs
- Need JWT-based authentication for APIs
- Want 500+ auth strategies available
- Require custom authentication logic
- Building microservices or pure backend APIs
- Not using Next.js

❌ **Use Auth.js instead when:**
- Building Next.js applications
- Want minimal configuration
- Need quick OAuth setup
- Want serverless/Edge support

---

## Quick Start

### Installation

```bash
npm install passport passport-jwt jsonwebtoken
npm install -D @types/passport @types/passport-jwt @types/jsonwebtoken
```

### JWT Strategy Setup

```typescript
// src/strategies/jwt.strategy.ts
import passport from 'passport';
import { Strategy as JwtStrategy, ExtractJwt } from 'passport-jwt';
import { prisma } from '../lib/prisma';

const options = {
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET!,
};

passport.use(new JwtStrategy(options, async (payload, done) => {
  try {
    const user = await prisma.user.findUnique({ 
      where: { id: payload.sub } 
    });
    
    if (!user) {
      return done(null, false);
    }
    
    return done(null, user);
  } catch (error) {
    return done(error, false);
  }
}));

export default passport;
```

---

## Express Integration

### Complete Setup

```typescript
// src/app.ts
import express from 'express';
import passport from 'passport';
import jwt from 'jsonwebtoken';
import { hash, verify } from 'argon2';
import './strategies/jwt.strategy';
import { prisma } from './lib/prisma';

const app = express();
app.use(express.json());
app.use(passport.initialize());

// Register
app.post('/auth/register', async (req, res) => {
  const { email, password, name } = req.body;
  
  const hashedPassword = await hash(password);
  const user = await prisma.user.create({
    data: { email, password: hashedPassword, name },
  });
  
  res.status(201).json({ id: user.id, email: user.email });
});

// Login
app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;
  
  const user = await prisma.user.findUnique({ where: { email } });
  if (!user || !user.password) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }
  
  const valid = await verify(user.password, password);
  if (!valid) {
    return res.status(401).json({ message: 'Invalid credentials' });
  }
  
  const accessToken = jwt.sign(
    { sub: user.id, email: user.email, role: user.role },
    process.env.JWT_SECRET!,
    { expiresIn: '15m' }
  );
  
  const refreshToken = jwt.sign(
    { sub: user.id },
    process.env.JWT_REFRESH_SECRET!,
    { expiresIn: '7d' }
  );
  
  res.json({ accessToken, refreshToken });
});

// Protected route
app.get('/api/profile',
  passport.authenticate('jwt', { session: false }),
  (req, res) => {
    res.json(req.user);
  }
);

// Refresh token
app.post('/auth/refresh', async (req, res) => {
  const { refreshToken } = req.body;
  
  try {
    const payload = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET!) as { sub: string };
    const user = await prisma.user.findUnique({ where: { id: payload.sub } });
    
    if (!user) {
      return res.status(401).json({ message: 'Invalid token' });
    }
    
    const accessToken = jwt.sign(
      { sub: user.id, email: user.email, role: user.role },
      process.env.JWT_SECRET!,
      { expiresIn: '15m' }
    );
    
    res.json({ accessToken });
  } catch {
    res.status(401).json({ message: 'Invalid token' });
  }
});

app.listen(3000);
```

---

## Middleware Helpers

### Role-Based Authorization

```typescript
// src/middleware/auth.middleware.ts
import { Request, Response, NextFunction } from 'express';
import passport from 'passport';

// Authenticate and attach user
export const authenticate = passport.authenticate('jwt', { session: false });

// Check specific role
export function requireRole(...roles: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    const user = req.user as { role: string } | undefined;
    
    if (!user || !roles.includes(user.role)) {
      return res.status(403).json({ message: 'Forbidden' });
    }
    
    next();
  };
}

// Usage:
app.get('/admin/users', 
  authenticate, 
  requireRole('admin'), 
  (req, res) => { /* ... */ }
);
```

### Optional Authentication

```typescript
export function optionalAuth(req: Request, res: Response, next: NextFunction) {
  passport.authenticate('jwt', { session: false }, (err, user) => {
    if (user) req.user = user;
    next();
  })(req, res, next);
}

// Route works with or without auth
app.get('/api/posts', optionalAuth, (req, res) => {
  const userId = (req.user as { id: string })?.id;
  // Show public posts, or personalized if logged in
});
```

---

## Integration with tRPC (Express)

```typescript
// src/server/context.ts
import { CreateExpressContextOptions } from '@trpc/server/adapters/express';
import { prisma } from '../lib/prisma';

export const createContext = ({ req }: CreateExpressContextOptions) => ({
  user: req.user, // Populated by Passport middleware
  prisma,
});

export type Context = ReturnType<typeof createContext>;
```

```typescript
// src/server/index.ts
import { createExpressMiddleware } from '@trpc/server/adapters/express';
import passport from 'passport';

// Apply Passport to all /trpc routes
app.use('/trpc', 
  passport.authenticate('jwt', { session: false, failWithError: false }),
  createExpressMiddleware({
    router: appRouter,
    createContext,
  })
);
```

---

## JWT Token Structure

```typescript
// Access token payload
interface AccessTokenPayload {
  sub: string;      // User ID
  email: string;
  role: string;
  iat: number;      // Issued at
  exp: number;      // Expiration
}

// Generate tokens
function generateTokens(user: User) {
  const accessToken = jwt.sign(
    { sub: user.id, email: user.email, role: user.role },
    process.env.JWT_SECRET!,
    { expiresIn: '15m' }
  );
  
  const refreshToken = jwt.sign(
    { sub: user.id },
    process.env.JWT_REFRESH_SECRET!,
    { expiresIn: '7d' }
  );
  
  return { accessToken, refreshToken };
}
```

---

## Type Definitions

```typescript
// src/types/express.d.ts
import { User } from '@prisma/client';

declare global {
  namespace Express {
    interface User {
      id: string;
      email: string;
      role: string;
    }
  }
}
```

---

## Environment Variables

```env
JWT_SECRET=your-access-token-secret-min-32-chars
JWT_REFRESH_SECRET=your-refresh-token-secret-min-32-chars
```

---

## Rules

### Do ✅

- Use `session: false` for stateless JWT auth
- Hash passwords with argon2 or bcrypt
- Use short-lived access tokens (15min)
- Use refresh tokens for long sessions
- Store refresh tokens securely (httpOnly cookie)
- Validate JWT on every request

### Avoid ❌

- Storing sensitive data in JWT payload
- Using long-lived access tokens
- Storing tokens in localStorage (XSS risk)
- Using weak secrets
- Skipping password hashing

---

## Common Strategies

| Strategy | Package | Use Case |
|----------|---------|----------|
| JWT | `passport-jwt` | API authentication |
| Local | `passport-local` | Username/password |
| OAuth2 | `passport-oauth2` | Generic OAuth |
| Google | `passport-google-oauth20` | Google login |
| GitHub | `passport-github2` | GitHub login |

---

## Auth.js vs Passport.js Decision Tree

| Scenario | Recommendation |
|----------|---------------|
| Next.js App Router | **Auth.js** |
| Express.js API | **Passport.js** |
| Quick OAuth setup | **Auth.js** |
| Custom auth logic | **Passport.js** |
| Serverless/Edge | **Auth.js** |
| Multiple strategies | **Passport.js** |
| Pure JWT API | **Passport.js** |

---

## Troubleshooting

```yaml
"401 on every request":
  → Check Authorization header format: "Bearer <token>"
  → Verify JWT_SECRET matches between sign and verify
  → Check token expiration

"req.user is undefined":
  → Ensure passport.initialize() is called
  → Check authenticate middleware is applied
  → Verify strategy name matches

"CORS issues with auth":
  → Set credentials: true in CORS config
  → Include origin whitelist
  → Allow Authorization header

"Token not refreshing":
  → Check refresh token secret is different
  → Verify refresh token hasn't expired
  → Store refresh token in httpOnly cookie
```

---

## File Structure

```
src/
├── strategies/
│   └── jwt.strategy.ts    # JWT strategy config
├── middleware/
│   └── auth.middleware.ts # Auth helpers
├── routes/
│   └── auth.routes.ts     # Auth endpoints
└── types/
    └── express.d.ts       # Type extensions
```

## References

- https://passportjs.org — Official documentation
- https://www.npmjs.com/package/passport-jwt — JWT strategy
- http://www.passportjs.org/packages/ — All strategies
