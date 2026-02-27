---
name: Frontend Development
description: Expert skill for Next.js 16 frontend development with React, TypeScript, and TailwindCSS. Use when building UI components, implementing hooks, or debugging frontend issues.
---

# Frontend Development Skill

## Overview

This skill provides expertise in developing the Chimera frontend using Next.js 16, React, TypeScript, and TailwindCSS with a focus on avant-garde, premium UI design.

## When to Use This Skill

- Building or modifying React components
- Implementing custom hooks (useAuth, useAegisTelemetry, etc.)
- Debugging WebSocket connections
- Styling with TailwindCSS and custom CSS
- Fixing TypeScript type errors
- Troubleshooting build or dev server issues

## Technology Stack

### Core Framework

- **Next.js 16**: React framework with App Router
- **React 19**: Latest React with Server Components
- **TypeScript 5.7+**: Strict type checking
- **TailwindCSS 3**: Utility-first CSS framework

### Key Dependencies

```json
{
  "next": "^16.0.0",
  "react": "^19.0.0",
  "react-dom": "^19.0.0",
  "typescript": "^5.7.0",
  "tailwindcss": "^3.4.0",
  "zustand": "^4.5.0",  // State management
  "axios": "^1.6.0",     // HTTP client
  "zod": "^3.22.0"       // Schema validation
}
```

## Project Structure

```
frontend/
├── src/
│   ├── app/                    # Next.js App Router pages
│   │   ├── (auth)/            # Authentication layouts
│   │   │   ├── login/
│   │   │   └── register/
│   │   ├── dashboard/         # Main dashboard
│   │   └── layout.tsx         # Root layout
│   ├── components/            # Reusable components
│   │   ├── aegis/            # Aegis-specific components
│   │   │   ├── AegisCampaignDashboard.tsx
│   │   │   ├── CampaignMetrics.tsx
│   │   │   └── PersonaVisualization.tsx
│   │   ├── ui/               # Radix UI primitives
│   │   └── layout/           # Layout components
│   ├── hooks/                # Custom React hooks
│   │   ├── useAuth.ts
│   │   ├── useAegisTelemetry.ts
│   │   └── useWebSocket.ts
│   ├── contexts/             # React Context providers
│   │   ├── AuthContext.tsx
│   │   └── WebSocketProvider.tsx
│   ├── lib/                  # Utility functions
│   │   ├── api.ts           # API client
│   │   └── utils.ts         # Helper functions
│   ├── styles/              # Global and custom CSS
│   │   └── globals.css
│   └── types/               # TypeScript type definitions
├── public/                  # Static assets
├── tailwind.config.ts       # TailwindCSS configuration
└── next.config.ts           # Next.js configuration
```

## Design Philosophy: "Avant-Garde Minimalism"

### Core Principles

1. **Anti-Generic**: Reject bootstrap templates, create bespoke layouts
2. **Intentional Placement**: Every element must have a calculated purpose
3. **Premium Aesthetics**: Use vibrant colors, dark modes, glassmorphism, dynamic animations
4. **Micro-Interactions**: Smooth hover effects, subtle animations for engagement

### Color Palette Guidelines

```css
/* AVOID: Plain generic colors */
--color-bad-red: #ff0000;
--color-bad-blue: #0000ff;

/* USE: Curated HSL-based colors */
--color-primary: hsl(260, 85%, 60%);      /* Vibrant purple */
--color-accent: hsl(180, 100%, 50%);      /* Cyan glow */
--color-success: hsl(142, 76%, 36%);      /* Rich green */
--color-danger: hsl(0, 84%, 60%);         /* Warm red */
```

### Typography

```css
/* Import Google Fonts in layout.tsx */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

.heading {
  font-family: 'Inter', 'Google Sans Flex', sans-serif;
  font-weight: 600;
  letter-spacing: -0.02em;
}
```

### Glassmorphism Example

```tsx
<div className="
  backdrop-blur-md bg-white/10 
  border border-white/20 
  rounded-2xl shadow-2xl 
  p-6 
  hover:bg-white/15 
  transition-all duration-300
">
  {/* Content */}
</div>
```

## Common Commands

### Development Server

```bash
# Start frontend only (port 3000)
npm run dev:frontend

# Or from frontend directory
cd frontend
npm run dev

# Frontend runs on http://localhost:3001
```

### Build and Production

```bash
# Create production build
cd frontend
npm run build

# Start production server
npm start

# Run type checking
npm run type-check
```

### Linting and Formatting

```bash
# Run ESLint
npm run lint

# Fix ESLint issues
npm run lint:fix

# Format with Prettier (if configured)
npm run format
```

## Common Issues and Solutions

### 1. Login Page Not Redirecting After Success

**Symptom**: Login succeeds but page stays on login screen

**Root Cause**: `isAuthStateReady` flag not properly managed or redirect logic missing

**Fix**:

```tsx
// In src/app/(auth)/login/page.tsx
"use client";

import { useAuth } from "@/hooks/useAuth";
import { useRouter } from "next/navigation";
import { useEffect } from "react";

export default function LoginPage() {
  const { user, isAuthStateReady } = useAuth();
  const router = useRouter();

  useEffect(() => {
    if (isAuthStateReady && user) {
      router.push("/dashboard");  // Redirect on successful auth
    }
  }, [isAuthStateReady, user, router]);

  // ... login form
}
```

### 2. WebSocket Connection Failures

**Symptom**: `useAegisTelemetry` hook fails to connect or disconnects immediately

**Root Cause**: Incorrect WebSocket URL or missing authentication

**Fix**:

```tsx
// In src/hooks/useAegisTelemetry.ts
import { useEffect, useState } from "react";

export function useAegisTelemetry(campaignId: string) {
  const [socket, setSocket] = useState<WebSocket | null>(null);
  const [data, setData] = useState(null);

  useEffect(() => {
    const ws = new WebSocket(
      `ws://localhost:8001/api/v1/ws/aegis/telemetry/${campaignId}`
    );

    ws.onopen = () => console.log("WebSocket connected");
    ws.onmessage = (event) => setData(JSON.parse(event.data));
    ws.onerror = (error) => console.error("WebSocket error:", error);
    ws.onclose = () => console.log("WebSocket closed");

    setSocket(ws);

    return () => {
      ws.close();
    };
  }, [campaignId]);

  return { socket, data };
}
```

### 3. TypeScript Type Errors

**Symptom**: `Type 'X' is not assignable to type 'Y'` errors

**Common Fixes**:

```tsx
// Issue: useState type inference
// BAD
const [data, setData] = useState(null);  // Type: null

// GOOD
const [data, setData] = useState<CampaignData | null>(null);

// Issue: Event handler types
// BAD
const handleSubmit = (e) => { ... }

// GOOD
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => { ... }

// Issue: Optional chaining
// BAD
const name = user.profile.name;  // Error if user is null

// GOOD
const name = user?.profile?.name ?? "Guest";
```

### 4. TailwindCSS Styles Not Applying

**Symptom**: Classes not generating styles

**Checklist**:

1. Verify `tailwind.config.ts` has correct content paths:

```ts
export default {
  content: [
    "./src/app/**/*.{js,ts,jsx,tsx}",
    "./src/components/**/*.{js,ts,jsx,tsx}",
  ],
  // ...
}
```

1. Ensure `globals.css` imports Tailwind directives:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

1. Restart dev server after config changes

### 5. API Proxy 404 Errors

**Symptom**: Frontend requests to `/api/*` return 404

**Root Cause**: Next.js not configured to proxy to backend

**Fix in `next.config.ts`**:

```ts
const nextConfig = {
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'http://localhost:8001/api/:path*',
      },
    ];
  },
};
```

## Component Development Best Practices

### 1. Use Radix UI Primitives (If Available)

```tsx
// Check if Radix UI is installed (Shadcn UI components)
// DO NOT build custom modals/dropdowns from scratch if library provides them

import { Button } from "@/components/ui/button";
import { Dialog } from "@/components/ui/dialog";

// Wrap/style library components for custom look
<Button className="bg-gradient-to-r from-purple-600 to-pink-600">
  Launch Campaign
</Button>
```

### 2. Server vs Client Components

```tsx
// Server Component (default in App Router)
// Use for static content, data fetching
export default function CampaignList({ campaigns }) {
  return <div>{campaigns.map(c => <CampaignCard key={c.id} {...c} />)}</div>;
}

// Client Component (for interactivity)
// Add "use client" directive
"use client";

export default function CampaignForm() {
  const [name, setName] = useState("");
  const handleSubmit = () => { /* ... */ };
  return <form onSubmit={handleSubmit}>...</form>;
}
```

### 3. Semantic HTML and Accessibility

```tsx
// GOOD: Semantic, accessible
<nav aria-label="Main navigation">
  <ul>
    <li><a href="/dashboard">Dashboard</a></li>
    <li><a href="/campaigns">Campaigns</a></li>
  </ul>
</nav>

// BAD: Divs for everything
<div className="nav">
  <div className="link">Dashboard</div>
</div>
```

### 4. Loading States and Error Boundaries

```tsx
import { Suspense } from "react";
import Loading from "./loading";
import ErrorBoundary from "./error";

export default function CampaignPage() {
  return (
    <ErrorBoundary fallback={<Error />}>
      <Suspense fallback={<Loading />}>
        <CampaignDashboard />
      </Suspense>
    </ErrorBoundary>
  );
}
```

## Custom Hooks Patterns

### useAuth Hook

```tsx
// src/hooks/useAuth.ts
import { useContext } from "react";
import { AuthContext } from "@/contexts/AuthContext";

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth must be used within AuthProvider");
  }
  return context;
}
```

### useWebSocket Hook

```tsx
// src/hooks/useWebSocket.ts
export function useWebSocket(url: string) {
  const [status, setStatus] = useState<"connecting" | "open" | "closed">("connecting");
  
  useEffect(() => {
    const ws = new WebSocket(url);
    ws.onopen = () => setStatus("open");
    ws.onclose = () => setStatus("closed");
    // Reconnection logic here
    return () => ws.close();
  }, [url]);

  return { status };
}
```

## Performance Optimization

### 1. Code Splitting

```tsx
// Dynamic import for heavy components
import dynamic from "next/dynamic";

const HeavyChart = dynamic(() => import("@/components/HeavyChart"), {
  ssr: false,  // Disable server-side rendering
  loading: () => <Skeleton />,
});
```

### 2. Image Optimization

```tsx
import Image from "next/image";

<Image 
  src="/campaign-bg.jpg"
  alt="Campaign background"
  width={1200}
  height={630}
  priority  // For above-the-fold images
/>
```

### 3. Memoization

```tsx
import { useMemo, useCallback } from "react";

const expensiveValue = useMemo(() => {
  return computeExpensiveValue(data);
}, [data]);

const handleClick = useCallback(() => {
  // Handler logic
}, [dependency]);
```

## Animation Examples

### Micro-Interactions

```tsx
<button className="
  px-6 py-3 
  bg-gradient-to-r from-purple-600 to-pink-600
  rounded-lg
  transform transition-all duration-200
  hover:scale-105 hover:shadow-2xl
  active:scale-95
  focus:outline-none focus:ring-2 focus:ring-purple-500
">
  Start Campaign
</button>
```

### Loading Spinner

```tsx
<div className="
  w-12 h-12 
  border-4 border-purple-200 
  border-t-purple-600 
  rounded-full 
  animate-spin
" />
```

## Testing

### Unit Tests (Vitest)

```tsx
import { render, screen } from "@testing-library/react";
import { describe, it, expect } from "vitest";

describe("Button", () => {
  it("renders with text", () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText("Click me")).toBeInTheDocument();
  });
});
```

### E2E Tests (Playwright)

```ts
import { test, expect } from "@playwright/test";

test("login flow", async ({ page }) => {
  await page.goto("http://localhost:3001/login");
  await page.fill('input[name="email"]', "test@example.com");
  await page.fill('input[name="password"]', "password");
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL(/.*dashboard/);
});
```

## References

- [frontend/README.md](../../frontend/README.md): Frontend-specific documentation
- [Next.js 16 Documentation](https://nextjs.org/docs)
- [TailwindCSS Documentation](https://tailwindcss.com/docs)
- [Radix UI Documentation](https://www.radix-ui.com/) (if using Shadcn UI)
