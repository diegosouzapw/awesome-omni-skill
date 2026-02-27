---
name: frontend-dev
description: Frontend development skill for React, Next.js, Tailwind CSS, and TypeScript. Use when implementing web frontend features.
---

# Frontend Development Skill

Platform-specific knowledge for web frontend development.

## Tech Stack

| Component | Technology |
|-----------|------------|
| Framework | Next.js 14+ (App Router) |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS |
| State | React hooks, Zustand (if needed) |
| Testing | Vitest, Playwright |
| Deployment | Cloudflare Pages |

## Coding Standards

### Naming
- Components: `PascalCase`
- Files: `kebab-case.tsx` or `PascalCase.tsx` (follow project)
- Functions/Variables: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE`
- Language: English

### File Organization
```typescript
// 1. External imports
import { useState } from 'react'
import { clsx } from 'clsx'

// 2. Internal imports
import { Button } from '@/components/ui/button'
import { useUser } from '@/hooks/use-user'

// 3. Types
interface Props {
  title: string
  onSubmit: () => void
}

// 4. Component
export function FeatureCard({ title, onSubmit }: Props) {
  const [isOpen, setIsOpen] = useState(false)

  return (
    // ...
  )
}
```

### Component Patterns
```typescript
// Prefer composition over props drilling
export function Card({ children }: { children: React.ReactNode }) {
  return <div className="rounded-lg border p-4">{children}</div>
}

Card.Header = function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="font-bold">{children}</div>
}

Card.Body = function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="mt-2">{children}</div>
}
```

### Tailwind Patterns
```typescript
// Use clsx for conditional classes
import { clsx } from 'clsx'

<button
  className={clsx(
    'rounded px-4 py-2',
    isActive ? 'bg-blue-500 text-white' : 'bg-gray-200'
  )}
/>

// Extract common patterns
const buttonVariants = {
  primary: 'bg-blue-500 text-white hover:bg-blue-600',
  secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
}
```

## Build Commands

```bash
# Install dependencies
pnpm install  # or npm install

# Development
pnpm dev

# Type check
pnpm typecheck  # or tsc --noEmit

# Lint
pnpm lint

# Build
pnpm build

# Test
pnpm test
```

## Next.js App Router

### Route Structure
```
app/
├── layout.tsx        # Root layout
├── page.tsx          # Home page
├── (marketing)/      # Route group (no URL segment)
│   ├── about/
│   │   └── page.tsx
│   └── pricing/
│       └── page.tsx
├── dashboard/
│   ├── layout.tsx    # Dashboard layout
│   └── page.tsx
└── api/
    └── route.ts      # API route
```

### Server vs Client Components
```typescript
// Server Component (default) - no 'use client'
async function ServerComponent() {
  const data = await fetch('...')  // Can fetch directly
  return <div>{data}</div>
}

// Client Component - needs interactivity
'use client'

import { useState } from 'react'

function ClientComponent() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```

### Data Fetching
```typescript
// Server Component
async function Page() {
  const data = await fetch('https://api.example.com/data', {
    next: { revalidate: 60 }  // ISR: revalidate every 60s
  })
  return <div>{data}</div>
}

// Client Component with SWR
'use client'
import useSWR from 'swr'

function ClientPage() {
  const { data, error } = useSWR('/api/data', fetcher)
  if (error) return <div>Error</div>
  if (!data) return <div>Loading...</div>
  return <div>{data}</div>
}
```

## Testing

### Vitest Unit Test
```typescript
import { describe, it, expect } from 'vitest'
import { render, screen } from '@testing-library/react'
import { Button } from './button'

describe('Button', () => {
  it('renders children', () => {
    render(<Button>Click me</Button>)
    expect(screen.getByText('Click me')).toBeInTheDocument()
  })
})
```

### Playwright E2E
```typescript
import { test, expect } from '@playwright/test'

test('home page', async ({ page }) => {
  await page.goto('/')
  await expect(page.getByRole('heading', { name: 'Welcome' })).toBeVisible()
})
```

## Common Issues

### Hydration Mismatch
- Use `suppressHydrationWarning` for dynamic content
- Check for client-only code in Server Components

### "Module not found"
- Check tsconfig paths
- Verify import aliases in next.config.js

### Tailwind not applying
- Check `content` paths in tailwind.config.js
- Restart dev server after config changes
