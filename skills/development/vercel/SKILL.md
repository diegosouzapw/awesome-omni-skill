---
name: vercel
description: Deploys applications to Vercel including serverless functions, edge functions, environment variables, and CI/CD. Use when deploying Next.js applications, frontend projects, or serverless APIs.
---

# Vercel

The frontend cloud platform for deploying web applications.

## Quick Start

**Install CLI:**
```bash
npm i -g vercel
```

**Deploy:**
```bash
vercel
```

**Deploy to production:**
```bash
vercel --prod
```

## Project Setup

### Connect Git Repository

1. Go to vercel.com/new
2. Import repository from GitHub/GitLab/Bitbucket
3. Configure build settings (auto-detected for most frameworks)
4. Deploy

### vercel.json Configuration

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "installCommand": "npm install",
  "framework": "nextjs",
  "regions": ["iad1"],
  "functions": {
    "api/**/*.ts": {
      "memory": 1024,
      "maxDuration": 10
    }
  },
  "rewrites": [
    { "source": "/api/:path*", "destination": "/api/:path*" }
  ],
  "redirects": [
    { "source": "/old", "destination": "/new", "permanent": true }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" }
      ]
    }
  ]
}
```

## Environment Variables

### Setting Variables

**Via Dashboard:**
1. Project Settings > Environment Variables
2. Add key-value pairs
3. Select environments (Production, Preview, Development)

**Via CLI:**
```bash
vercel env add MY_VAR
vercel env ls
vercel env pull .env.local
```

### Environment Types

```typescript
// Production only
NEXT_PUBLIC_API_URL=https://api.example.com

// Preview (PR deployments)
NEXT_PUBLIC_API_URL=https://staging-api.example.com

// Development
NEXT_PUBLIC_API_URL=http://localhost:3001
```

### Using Variables

```typescript
// Next.js - server side
const apiKey = process.env.API_KEY;

// Next.js - client side (must be prefixed)
const publicUrl = process.env.NEXT_PUBLIC_API_URL;
```

## Serverless Functions

### API Routes (Next.js App Router)

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const users = await getUsers();
  return NextResponse.json(users);
}

export async function POST(request: NextRequest) {
  const body = await request.json();
  const user = await createUser(body);
  return NextResponse.json(user, { status: 201 });
}
```

### API Routes (Pages Router)

```typescript
// pages/api/users.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method === 'GET') {
    const users = await getUsers();
    return res.json(users);
  }

  if (req.method === 'POST') {
    const user = await createUser(req.body);
    return res.status(201).json(user);
  }

  res.setHeader('Allow', ['GET', 'POST']);
  res.status(405).end(`Method ${req.method} Not Allowed`);
}
```

### Standalone Functions

```typescript
// api/hello.ts
import type { VercelRequest, VercelResponse } from '@vercel/node';

export default function handler(req: VercelRequest, res: VercelResponse) {
  return res.json({ message: 'Hello from Vercel!' });
}
```

### Function Configuration

```typescript
// app/api/heavy/route.ts
export const runtime = 'nodejs';
export const maxDuration = 60; // seconds
export const dynamic = 'force-dynamic';

export async function GET() {
  // Long-running operation
}
```

## Edge Functions

### Edge Runtime

```typescript
// app/api/geo/route.ts
import { NextRequest, NextResponse } from 'next/server';

export const runtime = 'edge';

export async function GET(request: NextRequest) {
  const country = request.geo?.country ?? 'Unknown';
  const city = request.geo?.city ?? 'Unknown';

  return NextResponse.json({
    country,
    city,
    message: `Hello from ${city}, ${country}!`,
  });
}
```

### Middleware

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Check auth
  const token = request.cookies.get('token');

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // Add headers
  const response = NextResponse.next();
  response.headers.set('x-custom-header', 'value');

  return response;
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

## Caching & ISR

### Static Generation with Revalidation

```typescript
// app/posts/page.tsx
export const revalidate = 3600; // Revalidate every hour

async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 3600 },
  });
  return res.json();
}

export default async function PostsPage() {
  const posts = await getPosts();
  return <PostList posts={posts} />;
}
```

### On-Demand Revalidation

```typescript
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache';
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { path, tag, secret } = await request.json();

  if (secret !== process.env.REVALIDATE_SECRET) {
    return NextResponse.json({ error: 'Invalid secret' }, { status: 401 });
  }

  if (path) {
    revalidatePath(path);
  }

  if (tag) {
    revalidateTag(tag);
  }

  return NextResponse.json({ revalidated: true });
}
```

## Redirects & Rewrites

### vercel.json

```json
{
  "redirects": [
    { "source": "/blog/:slug", "destination": "/posts/:slug", "permanent": true },
    { "source": "/old-page", "destination": "/new-page", "statusCode": 301 }
  ],
  "rewrites": [
    { "source": "/api/:path*", "destination": "https://api.example.com/:path*" },
    { "source": "/:path*", "destination": "/index.html" }
  ]
}
```

### Next.js Config

```javascript
// next.config.js
module.exports = {
  async redirects() {
    return [
      {
        source: '/old-blog/:slug',
        destination: '/blog/:slug',
        permanent: true,
      },
    ];
  },
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'https://api.backend.com/:path*',
      },
    ];
  },
};
```

## Deployment

### Preview Deployments

```bash
# Every push to non-production branch creates preview
git push origin feature-branch
# Creates: feature-branch-abc123.vercel.app
```

### Production Deployment

```bash
# Via CLI
vercel --prod

# Via Git (push to main/master)
git push origin main
```

### Rollback

```bash
# Via CLI
vercel rollback [deployment-url]

# Via Dashboard
# Deployments > ... > Promote to Production
```

## Monitoring

### Analytics

```typescript
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

### Speed Insights

```typescript
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <SpeedInsights />
      </body>
    </html>
  );
}
```

## Cron Jobs

```json
// vercel.json
{
  "crons": [
    {
      "path": "/api/cron/daily",
      "schedule": "0 0 * * *"
    },
    {
      "path": "/api/cron/hourly",
      "schedule": "0 * * * *"
    }
  ]
}
```

```typescript
// app/api/cron/daily/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const authHeader = request.headers.get('authorization');

  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // Run daily task
  await runDailyTask();

  return NextResponse.json({ success: true });
}
```

## Best Practices

1. **Use environment variables** - Never commit secrets
2. **Configure regions** - Deploy close to users/data
3. **Enable preview deployments** - Test before production
4. **Use ISR for dynamic content** - Balance freshness and speed
5. **Add monitoring** - Analytics and Speed Insights

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Exposing secrets | Use env vars, not code |
| Large function bundles | Split into smaller functions |
| Missing NEXT_PUBLIC_ prefix | Prefix client-side vars |
| No error handling | Add try/catch in functions |
| Cold start issues | Use edge runtime when possible |

## Reference Files

- [references/functions.md](references/functions.md) - Function patterns
- [references/caching.md](references/caching.md) - Caching strategies
- [references/domains.md](references/domains.md) - Custom domains
