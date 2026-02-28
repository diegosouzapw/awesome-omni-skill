---
name: cachibot-frontend-view
description: Create new frontend views and components for CachiBot's React+TypeScript UI with Zustand state management. Use this skill when building new pages, views, panels, or major UI components — e.g., "add a dashboard view", "create a logs panel", "build a new settings tab".
metadata:
  author: cachibot
  version: "1.0"
---

# CachiBot Frontend View Creation

Create new views and components for CachiBot's React frontend following its established patterns.

## Architecture Overview

- **Framework**: React 19 + TypeScript (strict mode)
- **State**: Zustand stores with localStorage persistence
- **Styling**: Tailwind CSS (dark-first, zinc color palette)
- **Icons**: lucide-react
- **Routing**: React Router v7
- **Path alias**: `@/*` maps to `src/*`

## Step-by-Step Process

### 1. Create the View Component

Create `frontend/src/components/views/YourView.tsx`:

```tsx
import { useState, useEffect } from 'react'
import { /* icons you need */ Loader2, Plus } from 'lucide-react'
import { useBotStore } from '../../stores/bots'
import { useYourStore } from '../../stores/your-store'  // if needed
import { cn } from '../../lib/utils'

export function YourView() {
  const { getActiveBot } = useBotStore()
  const activeBot = getActiveBot()
  const [loading, setLoading] = useState(false)

  if (!activeBot) {
    return (
      <div className="flex h-full items-center justify-center">
        <p className="text-zinc-500">Select a bot to continue</p>
      </div>
    )
  }

  return (
    <div className="flex h-full flex-col bg-zinc-100 dark:bg-zinc-950">
      {/* Header */}
      <div className="flex items-center justify-between border-b border-zinc-200 bg-white px-6 py-4 dark:border-zinc-800 dark:bg-zinc-900/50">
        <div>
          <h2 className="text-lg font-semibold text-zinc-900 dark:text-zinc-100">
            Your View Title
          </h2>
          <p className="text-sm text-zinc-500">Description text</p>
        </div>
        <button
          className="flex items-center gap-2 rounded-lg bg-cachi-600 px-4 py-2 text-sm font-medium text-white transition-colors hover:bg-cachi-500"
        >
          <Plus className="h-4 w-4" />
          Action
        </button>
      </div>

      {/* Content */}
      <div className="flex-1 overflow-y-auto p-6">
        <div className="mx-auto max-w-4xl">
          {loading ? (
            <div className="flex items-center justify-center py-12">
              <Loader2 className="h-6 w-6 animate-spin text-zinc-400" />
            </div>
          ) : (
            <div className="space-y-4">
              {/* Your content here */}
            </div>
          )}
        </div>
      </div>
    </div>
  )
}
```

### 2. Export from Views Index

Add to `frontend/src/components/views/index.ts`:

```typescript
export { YourView } from './YourView'
```

### 3. Add to View Type

Edit `frontend/src/types/index.ts` — add to `BotView`:

```typescript
export type BotView = 'chats' | 'tasks' | 'tools' | 'settings' | 'voice' | 'work' | 'schedules' | 'your-view'
```

### 4. Add Navigation in BotSidebar

Edit `frontend/src/components/layout/BotSidebar.tsx` — add a nav button:

```tsx
// Add import for your icon
import { YourIcon } from 'lucide-react'

// In the navigation section, add:
<button
  onClick={() => setActiveView('your-view')}
  className={cn(
    'flex items-center gap-3 rounded-lg px-3 py-2 text-sm transition-colors',
    activeView === 'your-view'
      ? 'bg-cachi-600/20 text-cachi-400'
      : 'text-zinc-400 hover:bg-zinc-800 hover:text-zinc-200'
  )}
>
  <YourIcon className="h-4 w-4" />
  Your View
</button>
```

### 5. Add View Routing in MainLayout

Edit `frontend/src/components/layout/MainLayout.tsx` — add the view case:

```tsx
// Add import
import { YourView } from '../views/YourView'

// In the view switch/conditional rendering:
{activeView === 'your-view' && <YourView />}
```

## Styling Conventions

### Color Palette (Dark-First)
```
Background:     bg-zinc-950 (page), bg-zinc-900/50 (cards/headers)
Borders:        border-zinc-800
Text primary:   text-zinc-100
Text secondary: text-zinc-400, text-zinc-500
Accent:         bg-cachi-600, text-cachi-400, hover:bg-cachi-500
Success:        text-green-400
Error:          text-red-400
Warning:        text-amber-400
```

### Common Component Patterns

**Card:**
```tsx
<div className="rounded-xl border border-zinc-200 bg-white p-4 dark:border-zinc-800 dark:bg-zinc-900/50">
  {/* content */}
</div>
```

**Empty state:**
```tsx
<div className="flex flex-col items-center justify-center py-12 text-center">
  <SomeIcon className="mb-4 h-12 w-12 text-zinc-600" />
  <h3 className="mb-2 text-lg font-medium text-zinc-300">No items yet</h3>
  <p className="mb-4 text-sm text-zinc-500">Description of what to do</p>
  <button className="rounded-lg bg-cachi-600 px-4 py-2 text-sm text-white hover:bg-cachi-500">
    Create First Item
  </button>
</div>
```

**Action button row:**
```tsx
<div className="flex items-center gap-2 opacity-0 transition-opacity group-hover:opacity-100">
  <button className="flex h-6 items-center gap-1 rounded px-2 text-xs text-zinc-500 hover:bg-zinc-800 hover:text-zinc-300">
    <EditIcon className="h-3 w-3" />
    Edit
  </button>
</div>
```

### Zustand Store Pattern

If your view needs its own store, create `frontend/src/stores/your-store.ts`:

```typescript
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

interface YourState {
  items: YourItem[]
  loading: boolean

  setItems: (items: YourItem[]) => void
  addItem: (item: YourItem) => void
  removeItem: (id: string) => void
}

export const useYourStore = create<YourState>()(
  persist(
    (set) => ({
      items: [],
      loading: false,

      setItems: (items) => set({ items }),
      addItem: (item) => set((s) => ({ items: [...s.items, item] })),
      removeItem: (id) => set((s) => ({
        items: s.items.filter((i) => i.id !== id),
      })),
    }),
    { name: 'cachibot-your-store' }  // localStorage key
  )
)
```

### API Client Function

If the view fetches data from the backend, add to `frontend/src/api/client.ts`:

```typescript
export async function getYourItems(botId: string): Promise<YourItem[]> {
  const res = await fetchWithAuth(`/api/bots/${botId}/your-domain`)
  if (!res.ok) throw new Error('Failed to fetch items')
  return res.json()
}
```

## Checklist

- [ ] View component created in `frontend/src/components/views/`
- [ ] View exported from `frontend/src/components/views/index.ts`
- [ ] `BotView` type updated in `frontend/src/types/index.ts`
- [ ] Navigation button added in `BotSidebar.tsx`
- [ ] View routing added in `MainLayout.tsx`
- [ ] TypeScript types defined in `frontend/src/types/index.ts`
- [ ] Zustand store created (if needed) in `frontend/src/stores/`
- [ ] API client functions added (if needed) in `frontend/src/api/`
- [ ] Dark-mode-first styling with zinc/cachi color palette
- [ ] `npm run build` passes with no type errors
