---
name: preferences-react-tanstack-ui-development
description: React and TanStack UI development patterns including component design, routing, and state management. Load when working with React components or TanStack libraries.
---

# React / TanStack UI development

## Architectural patterns alignment

See @~/.claude/skills/preferences-architectural-patterns/SKILL.md for overarching principles.

Apply functional programming patterns to React UI development:
- Components are pure functions from props to JSX
- State transformations should be explicit and traceable
- Side effects isolated to hooks and boundaries (useEffect, mutations)
- Composition over inheritance for component design
- Type-safety enforced from routing through rendering

## TanStack ecosystem integration

The TanStack ecosystem provides type-safe, composable primitives for modern React applications.

### Core libraries

- **TanStack Router**: File-based routing with full type inference for params, search, and loaders
- **TanStack Query**: Declarative server state management with automatic caching and revalidation
- **TanStack Form**: Type-safe form handling with validation and error management
- **TanStack Store**: Reactive client state with fine-grained subscriptions
- **TanStack DB**: Reactive collections with live queries for complex client-side data

### Integration pattern

```typescript
// TanStack Router + Query + tRPC creates end-to-end type safety
import { createFileRoute } from '@tanstack/react-router'
import { api } from '@/lib/trpc'

export const Route = createFileRoute('/users/$userId')({
  // Loader provides type-safe data to component
  loader: async ({ params }) => {
    const user = await api.users.getById.query(params.userId)
    return { user }
  },
})

function UserProfile() {
  const { user } = Route.useLoaderData() // Fully typed!
  // ...
}
```

## State management philosophy

Separate concerns between server state and client state.

### Server state (TanStack Query)

Use TanStack Query for any data that originates from the server:
- API responses
- Database queries via tRPC
- Remote resources and assets
- Paginated and infinite lists

**Key patterns**:
- Let TanStack Query handle caching, revalidation, and background updates
- Use optimistic updates for perceived performance
- Leverage query invalidation for cache management
- Structure queries with proper keys for selective invalidation

```typescript
// Server state with TanStack Query + tRPC
import { api } from '@/lib/trpc'

function UserList() {
  const { data, isLoading } = api.users.list.useQuery()
  const updateUser = api.users.update.useMutation({
    onSuccess: () => {
      // Invalidate to refetch
      utils.users.list.invalidate()
    }
  })

  // ...
}
```

### Client state (TanStack Store / Zustand)

Use client state management for UI-only state:
- Form field values before submission
- UI toggles (modals, dropdowns, themes)
- Client-side filters and sorts
- Transient interaction state

**Key patterns**:
- Keep client state minimal and derive from server state when possible
- Use TanStack Store for reactive updates with fine-grained subscriptions
- Consider Zustand for simpler, less reactive client state needs
- Avoid duplicating server state in client state

```typescript
// Client state with Zustand
import { create } from 'zustand'

interface UIState {
  isModalOpen: boolean
  selectedIds: Set<string>
  toggleModal: () => void
  selectId: (id: string) => void
}

const useUIStore = create<UIState>((set) => ({
  isModalOpen: false,
  selectedIds: new Set(),
  toggleModal: () => set((state) => ({ isModalOpen: !state.isModalOpen })),
  selectId: (id) => set((state) => ({
    selectedIds: new Set(state.selectedIds).add(id)
  })),
}))
```

## Pragmatic exceptions for specialized use cases

### Client-side analytics with DuckDB WASM

For applications performing client-side data analysis with DuckDB WASM (e.g., using SQLRooms), **Zustand is acceptable** instead of TanStack Store when:
- The application architecture follows vertical slices isolating analytics concerns
- Type-safe data source configuration uses Zod schemas (ADT alignment maintained)
- Query execution and state updates are confined to dedicated slices
- The imperative state patterns do not leak into general application logic

**Rationale**: DuckDB WASM analytics applications require fine-grained control over query execution, caching, and result streaming.
Zustand's imperative API provides ergonomic access to these patterns without the overhead of TanStack Store's reactive primitives.
The vertical slice architecture ensures analytics state remains isolated from general UI state.

#### SQLRooms vertical slice pattern

SQLRooms implements a composable slice architecture where each feature is self-contained:

```typescript
// Vertical slice contains: state + actions + UI + config schema
import { createRoomStore, createRoomShellSlice } from '@sqlrooms/room-shell'
import { createSqlEditorSlice } from '@sqlrooms/sql-editor'
import { createWasmDuckDbConnector } from '@sqlrooms/duckdb'
import { z } from 'zod'
import { persist } from 'zustand/middleware'

// Type-safe configuration with Zod (ADT pattern)
const RoomConfig = BaseRoomConfig.merge(SqlEditorSliceConfig)
type RoomConfig = z.infer<typeof RoomConfig>

// Compose slices into unified store
const { roomStore, useRoomStore } = createRoomStore<RoomConfig, RoomState>(
  persist(
    (set, get, store) => ({
      // DuckDB connector slice
      ...createRoomShellSlice({
        connector: createWasmDuckDbConnector(),
        config: { layout: {...}, dataSources: [...] },
        room: { panels: {...} }
      })(set, get, store),

      // SQL editor slice
      ...createSqlEditorSlice()(set, get, store),

      // Custom application slice
      customAnalytics: {
        filters: {},
        applyFilter: (filter) => set({ customAnalytics: {...} })
      }
    }),
    {
      name: 'analytics-app-storage',
      // Only persist configuration, not runtime state
      partialize: (state) => ({
        config: RoomConfig.parse(state.config)
      })
    }
  )
)
```

#### Type-safe S3/HTTPFS data sources with Zod

Use discriminated unions for data source configuration (matches ADT patterns from @~/.claude/skills/preferences-algebraic-data-types/SKILL.md):

```typescript
// From @sqlrooms/room-config - Zod schemas for type safety
const DataSourceTypes = z.enum(['file', 'url', 'sql'])

const BaseDataSource = z.object({
  type: DataSourceTypes,
  tableName: z.string()
})

// Discriminated union for different source types
const UrlDataSource = BaseDataSource.extend({
  type: z.literal('url'),
  url: z.string(),
  loadOptions: z.object({
    method: z.enum(['read_csv', 'read_parquet']),
    select: z.array(z.string()).optional(),
    where: z.string().optional()
  }).optional(),
  httpMethod: z.string().optional(),
  headers: z.record(z.string(), z.string()).optional()
})

const DataSource = z.discriminatedUnion('type', [
  FileDataSource,
  UrlDataSource,
  SqlQueryDataSource
])

// S3 data source configuration for public ducklake data
const s3DataSource: UrlDataSource = {
  type: 'url',
  tableName: 'public_ducklake_data',
  // DuckDB WASM with httpfs extension supports direct S3 access
  url: 's3://ducklake-public/data/events.parquet',
  loadOptions: {
    method: 'read_parquet',
    // Push-down filters to parquet reader (performance optimization)
    select: ['user_id', 'event_type', 'created_at', 'payload'],
    where: "created_at >= '2024-01-01' AND event_type = 'UserCreated'"
  }
}

// Configuration validated at runtime and compile-time
const config = {
  dataSources: [s3DataSource]
}
RoomConfig.parse(config) // Runtime validation
```

#### Railway-oriented programming for query error handling

Apply Result types to query execution for explicit error handling.

See @~/.claude/skills/preferences-railway-oriented-programming/SKILL.md for Result patterns and error composition.

```typescript
import { z } from 'zod'

// Result type for query operations
type Result<T, E> =
  | { ok: true; value: T }
  | { ok: false; error: E }

const success = <T, E>(value: T): Result<T, E> =>
  ({ ok: true, value })

const failure = <T, E>(error: E): Result<T, E> =>
  ({ ok: false, error })

// Error types as discriminated union
type QueryError =
  | { type: 'connection'; message: string }
  | { type: 'syntax'; message: string; line?: number }
  | { type: 'data_source'; message: string; source: string }
  | { type: 'network'; message: string; status?: number }

// Query execution with Result type
const executeQuery = async (
  sql: string,
  db: DuckDB
): Promise<Result<QueryResult, QueryError>> => {
  try {
    // Validate SQL syntax (applicative validation)
    const syntaxResult = validateSqlSyntax(sql)
    if (!syntaxResult.ok) return syntaxResult

    // Execute query
    const result = await db.query(sql)
    return success(result)
  } catch (error) {
    // Map exceptions to typed errors
    if (error.message.includes('Catalog Error')) {
      return failure({
        type: 'data_source',
        message: 'Table not found or data source not loaded',
        source: extractTableName(error.message)
      })
    }
    if (error.message.includes('Syntax error')) {
      return failure({
        type: 'syntax',
        message: error.message,
        line: extractLineNumber(error.message)
      })
    }
    return failure({
      type: 'connection',
      message: error.message
    })
  }
}

// Zustand slice with railway-oriented query actions
const createQuerySlice = () => (set, get, store) => ({
  query: {
    sql: '',
    result: null as QueryResult | null,
    error: null as QueryError | null,
    isExecuting: false,

    // Action with explicit error handling
    executeQuery: async (sql: string) => {
      set({ query: { ...get().query, isExecuting: true, error: null } })

      const result = await executeQuery(sql, get().db.connection)

      // Railway tracks: success or failure
      if (result.ok) {
        set({
          query: {
            ...get().query,
            result: result.value,
            isExecuting: false
          }
        })
      } else {
        set({
          query: {
            ...get().query,
            error: result.error,
            isExecuting: false
          }
        })
      }
    }
  }
})

// UI component with pattern matching on error type
function QueryErrorDisplay({ error }: { error: QueryError }) {
  switch (error.type) {
    case 'syntax':
      return (
        <div className="text-destructive">
          <p>Syntax error{error.line ? ` at line ${error.line}` : ''}</p>
          <p className="text-sm">{error.message}</p>
        </div>
      )
    case 'data_source':
      return (
        <div className="text-destructive">
          <p>Data source error: {error.source}</p>
          <p className="text-sm">{error.message}</p>
          <button onClick={loadDataSource}>Load data source</button>
        </div>
      )
    case 'network':
      return (
        <div className="text-destructive">
          <p>Network error{error.status ? ` (${error.status})` : ''}</p>
          <p className="text-sm">{error.message}</p>
          <button onClick={retry}>Retry</button>
        </div>
      )
    case 'connection':
      return (
        <div className="text-destructive">
          <p>Connection error</p>
          <p className="text-sm">{error.message}</p>
        </div>
      )
  }
}
```

#### S3/HTTPFS configuration requirements

For browser-based S3 queries with DuckDB WASM httpfs extension:

**CORS configuration**: S3 bucket must allow browser origins

```json
{
  "CORSRules": [
    {
      "AllowedOrigins": ["https://your-app.com"],
      "AllowedMethods": ["GET", "HEAD"],
      "AllowedHeaders": ["*"],
      "ExposeHeaders": ["ETag", "Content-Length"]
    }
  ]
}
```

**DuckDB httpfs extension**: Automatically loaded by @sqlrooms/duckdb

```typescript
// Extension loads transparently when using s3:// URLs
const connector = createWasmDuckDbConnector({
  // Optional: Initialize additional extensions
  initializationQuery: `
    INSTALL httpfs;
    LOAD httpfs;
    SET s3_region='us-east-1';
  `
})
```

**Direct parquet queries**: No table creation needed for one-off queries

```typescript
// Query parquet files directly from S3
const adhocQuery = `
  SELECT
    event_type,
    COUNT(*) as count
  FROM read_parquet('s3://ducklake-public/data/events/*.parquet')
  WHERE created_at >= '2024-01-01'
  GROUP BY event_type
`

const result = await executeQuery(adhocQuery, db)
```

#### When to use this pattern

Use SQLRooms with Zustand for:
- **Client-side analytics applications**: User-facing data exploration tools
- **Privacy-preserving analytics**: Data never leaves the user's device
- **Offline-first data tools**: Full query capabilities without server
- **Local-first dashboards**: Real-time visualization of local/remote data

Return to TanStack Store for:
- **General application state**: Non-analytics UI state
- **Server state management**: Use TanStack Query + tRPC instead
- **Real-time collaboration**: Use TanStack DB for live queries

### TanStack DB for reactive collections

Use TanStack DB when you need multiple live views of the same data:
- Dashboards with filtered sections
- Real-time collaborative interfaces
- Complex client-side aggregations
- Data tables with dynamic filtering

```typescript
// Reactive collections with TanStack DB
import { createCollection } from '@tanstack/react-db'
import { useLiveQuery } from '@tanstack/react-db'

const todoCollection = createCollection(
  queryCollectionOptions<Todo>({
    queryKey: ['todos'],
    queryFn: async () => api.todos.list.query(),
    queryClient,
    getKey: (item) => item.id,
  })
)

// Multiple views of same data, all automatically reactive
function TodoDashboard() {
  const { data: all } = useLiveQuery(q => q.from({ todo: todoCollection }))
  const { data: pending } = useLiveQuery(q =>
    q.from({ todo: todoCollection }).where(({ todo }) => !todo.completed)
  )
  const { data: completed } = useLiveQuery(q =>
    q.from({ todo: todoCollection }).where(({ todo }) => todo.completed)
  )

  // All views update automatically when data changes
}
```

## Component architecture

### Composition patterns

Favor composition over prop drilling or complex context hierarchies.

```typescript
// Component composition with compound components
function DataTable({ children }: { children: React.ReactNode }) {
  const [sorting, setSorting] = useState<SortingState>([])

  return (
    <DataTableContext.Provider value={{ sorting, setSorting }}>
      {children}
    </DataTableContext.Provider>
  )
}

DataTable.Header = function Header() {
  const { sorting, setSorting } = useDataTableContext()
  // ...
}

DataTable.Body = function Body({ data }: { data: unknown[] }) {
  const { sorting } = useDataTableContext()
  // ...
}

// Usage
<DataTable>
  <DataTable.Header />
  <DataTable.Body data={users} />
</DataTable>
```

### Component libraries

**Preferred**: shadcn/ui with Radix UI primitives
- Copy-paste components for full control
- Accessible by default (Radix UI)
- Customizable with Tailwind CSS
- Type-safe composition

**Pattern**: Extend shadcn/ui components with domain logic

```typescript
// Extend shadcn Button with app-specific variants
import { Button } from '@/components/ui/button'
import { cva, type VariantProps } from 'class-variance-authority'

const appButtonVariants = cva('', {
  variants: {
    intent: {
      primary: 'bg-primary text-primary-foreground',
      danger: 'bg-destructive text-destructive-foreground',
      success: 'bg-green-600 text-white',
    },
  },
})

interface AppButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof appButtonVariants> {
  isLoading?: boolean
}

export function AppButton({ intent, isLoading, children, ...props }: AppButtonProps) {
  return (
    <Button className={appButtonVariants({ intent })} disabled={isLoading} {...props}>
      {isLoading ? <Spinner /> : children}
    </Button>
  )
}
```

### Avoid prop drilling

Use composition and context judiciously:
- **Composition first**: Pass components as children or props
- **Context for cross-cutting concerns**: Theme, auth, i18n
- **Not for data flow**: Use TanStack Router loaders instead

## Type-safe routing

TanStack Router provides compile-time type safety for routing.

### File-based routing conventions

```
routes/
  __root.tsx                    # Root layout
  index.tsx                     # / route
  users/
    index.tsx                   # /users
    $userId.tsx                 # /users/:userId
    $userId/
      edit.tsx                  # /users/:userId/edit
  (auth)/                       # Route group (no path segment)
    login.tsx                   # /login
    register.tsx                # /register
```

### Route definitions with loaders

```typescript
// routes/users/$userId.tsx
import { createFileRoute } from '@tanstack/react-router'
import { z } from 'zod'

// Validate search params
const userSearchSchema = z.object({
  tab: z.enum(['profile', 'settings', 'activity']).default('profile'),
})

export const Route = createFileRoute('/users/$userId')({
  // Validate params
  validateSearch: userSearchSchema,

  // Type-safe loader
  loader: async ({ params, context }) => {
    const user = await context.api.users.getById.query(params.userId)
    if (!user) throw new Error('User not found')
    return { user }
  },

  // Error boundary
  errorComponent: ({ error }) => <ErrorDisplay error={error} />,
})

function UserPage() {
  const { user } = Route.useLoaderData() // Typed as User!
  const search = Route.useSearch() // Typed from schema!
  const navigate = Route.useNavigate()

  // Type-safe navigation
  navigate({
    to: '/users/$userId',
    params: { userId: user.id },
    search: { tab: 'settings' }, // Type-checked!
  })
}
```

### Protected routes

```typescript
// routes/(protected)/_layout.tsx
export const Route = createFileRoute('/(protected)/_layout')({
  beforeLoad: async ({ context }) => {
    if (!context.auth.isAuthenticated) {
      throw redirect({ to: '/login' })
    }
  },
})
```

## Form handling

Integrate TanStack Form with Zod for type-safe validation and railway-oriented submission.

### Form patterns with railway-oriented programming

See @~/.claude/skills/preferences-railway-oriented-programming/SKILL.md for Result types and error composition.

```typescript
import { useForm } from '@tanstack/react-form'
import { zodValidator } from '@tanstack/zod-form-adapter'
import { z } from 'zod'

// Schema defines validation rules
const userSchema = z.object({
  email: z.string().email('Invalid email'),
  name: z.string().min(2, 'Name too short'),
  age: z.number().min(18, 'Must be 18+'),
})

type UserFormData = z.infer<typeof userSchema>

function UserForm() {
  const createUser = api.users.create.useMutation()

  const form = useForm({
    defaultValues: {
      email: '',
      name: '',
      age: 0,
    },
    onSubmit: async ({ value }) => {
      // Railway-oriented pipeline:
      // 1. Form validates (handled by TanStack Form + Zod)
      // 2. Submit to API (tRPC ensures type safety)
      // 3. Handle result
      const result = await createUser.mutateAsync(value)

      // Success track
      if (result.ok) {
        toast.success('User created')
        navigate({ to: '/users/$userId', params: { userId: result.data.id } })
      }
      // Failure track
      else {
        toast.error(result.error.message)
      }
    },
    validatorAdapter: zodValidator(),
  })

  return (
    <form onSubmit={(e) => {
      e.preventDefault()
      form.handleSubmit()
    }}>
      <form.Field
        name="email"
        validators={{
          onChange: userSchema.shape.email,
        }}
        children={(field) => (
          <div>
            <Input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
            {field.state.meta.errors && (
              <span className="text-destructive">{field.state.meta.errors[0]}</span>
            )}
          </div>
        )}
      />
      {/* Other fields... */}
      <Button type="submit" disabled={form.state.isSubmitting}>
        Submit
      </Button>
    </form>
  )
}
```

### Collecting all validation errors

Use applicative validation for better UX - show all errors at once.

See @~/.claude/skills/preferences-railway-oriented-programming/SKILL.md for applicative patterns.

```typescript
// TanStack Form validates all fields before submit
// Errors collected and displayed per-field
const form = useForm({
  validators: {
    // Run all validations, collect all errors
    onSubmit: userSchema,
  },
})
```

## Reactive patterns

### Avoiding unnecessary re-renders

```typescript
// ⊘ Bad: Subscribes to entire store
function BadComponent() {
  const store = useUIStore() // Re-renders on ANY state change
  return <div>{store.someValue}</div>
}

// ● Good: Subscribe to specific values
function GoodComponent() {
  const someValue = useUIStore((state) => state.someValue) // Only re-renders when someValue changes
  return <div>{someValue}</div>
}
```

### React Server Components (TanStack Start)

When using TanStack Start with SSR:
- Use server functions for data fetching where appropriate
- Minimize client-side JavaScript with Server Components
- Stream data for progressive enhancement

```typescript
// Server function
import { createServerFn } from '@tanstack/start'

export const getUser = createServerFn('GET', async (userId: string) => {
  // Runs only on server
  const user = await db.users.findById(userId)
  return user
})

// Client component can call server function
function UserProfile({ userId }: { userId: string }) {
  const user = await getUser(userId) // Type-safe server call
  return <div>{user.name}</div>
}
```

## Build tooling

### Vite with Rolldown

Prefer Rolldown over traditional Vite for faster builds:

```json
{
  "devDependencies": {
    "vite": "npm:rolldown-vite@latest"
  },
  "overrides": {
    "vite": "npm:rolldown-vite@latest"
  }
}
```

**Benefits**:
- Faster build times (Rust-based)
- Better tree-shaking
- Smaller bundle sizes

### Biome for linting and formatting

Use Biome instead of ESLint + Prettier for unified tooling.

**Configuration** (`biome.json`):

```json
{
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 120
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "style": {
        "useAsConstAssertion": "error",
        "noParameterAssign": "error"
      }
    }
  }
}
```

**Scripts**:

```json
{
  "scripts": {
    "format": "biome format --write",
    "lint": "biome lint",
    "check": "biome check",
    "check:fix": "biome check --write"
  }
}
```

## Package managers

### Bun vs pnpm

**Prefer Bun** for:
- Monorepo projects with local development
- Projects using Bun runtime features
- Faster installation and execution

**Use pnpm** for:
- Better compatibility with legacy packages
- Stricter dependency isolation
- Projects with complex peer dependencies

**Lock file convention**:
- Commit `bun.lockb` or `pnpm-lock.yaml`
- Never mix lock files in same project

## Styling

### Tailwind CSS patterns

Use Tailwind CSS v4 with the Vite plugin:

```typescript
// vite.config.ts
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [tailwindcss()],
})
```

**Import in entry**:

```typescript
// src/main.tsx or src/app.css
import 'tailwindcss'
```

### Component variants with CVA

Use `class-variance-authority` for type-safe variant management:

```typescript
import { cva, type VariantProps } from 'class-variance-authority'
import { cn } from '@/lib/utils'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-accent',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 px-3',
        lg: 'h-11 px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  }
)

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {}

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size }), className)}
      {...props}
    />
  )
}
```

### Design tokens

Extract design tokens to CSS variables for consistency:

```css
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
  }
}
```

## Performance optimization

### Code splitting patterns

```typescript
// Route-based code splitting (automatic with TanStack Router)
// Each route component is lazy-loaded

// Component-based splitting
import { lazy, Suspense } from 'react'

const HeavyChart = lazy(() => import('./HeavyChart'))

function Dashboard() {
  return (
    <Suspense fallback={<ChartSkeleton />}>
      <HeavyChart data={data} />
    </Suspense>
  )
}
```

### Bundle optimization with Rolldown

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom'],
          'vendor-tanstack': [
            '@tanstack/react-router',
            '@tanstack/react-query',
          ],
          'vendor-ui': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
        },
      },
    },
  },
})
```

### Image optimization

```typescript
// Use modern formats with fallbacks
<picture>
  <source srcSet="/image.avif" type="image/avif" />
  <source srcSet="/image.webp" type="image/webp" />
  <img src="/image.jpg" alt="Description" loading="lazy" />
</picture>
```

## Deployment

See @~/.claude/skills/preferences-web-application-deployment/SKILL.md for comprehensive deployment guidance including:
- Cloudflare Workers/Pages deployment (preferred)
- Database configuration (D1, PostgreSQL)
- Wrangler configuration and platform bindings
- Multi-environment strategies
- Service bindings for microservices architecture

### React-specific deployment patterns

#### Vite configuration with Cloudflare plugin

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import viteReact from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";
import { cloudflare } from "@cloudflare/vite-plugin";

export default defineConfig({
  plugins: [
    tailwindcss(),
    tanstackStart({
      srcDirectory: "src",
      start: { entry: "./start.tsx" },
      server: { entry: "./server.ts" }, // Custom Cloudflare Workers entry
    }),
    viteReact(),
    cloudflare({
      viteEnvironment: {
        name: "ssr", // Enable SSR environment for Workers
      },
    }),
  ],
});
```

#### SSR hydration considerations

**Environment variables**:
- Avoid `VITE_` prefix for secrets - they bundle into client code
- Access runtime environment via Cloudflare bindings in server functions

```typescript
import { env } from "cloudflare:workers";

// Server function with runtime access
export const serverFunction = createServerFn()
  .handler(async () => {
    // Access Cloudflare bindings
    const data = await env.DB.prepare("SELECT * FROM users").all();
    return data;
  });
```

**Hydration**:
- Ensure server and client render identical markup
- Use `suppressHydrationWarning` only for intentional mismatches (timestamps, randomness)

```typescript
// Prevent hydration mismatch for client-only content
function ClientOnly({ children }: { children: React.ReactNode }) {
  const [mounted, setMounted] = useState(false)

  useEffect(() => {
    setMounted(true)
  }, [])

  if (!mounted) return null
  return <>{children}</>
}

// Usage: Wrap client-only components
<ClientOnly>
  <BrowserOnlyFeature />
</ClientOnly>
```

**Common hydration issues**:
- Different timestamps between server and client renders
- Browser-specific APIs called during SSR (window, localStorage)
- Random values generated server-side not matching client-side

```typescript
// ⊘ Causes hydration mismatch
function BadComponent() {
  return <div>{Math.random()}</div>
}

// ● Use suppressHydrationWarning for intentional mismatch
function GoodComponent() {
  return <div suppressHydrationWarning>{Math.random()}</div>
}

// ● Or use ClientOnly wrapper
function BestComponent() {
  return (
    <ClientOnly>
      <div>{Math.random()}</div>
    </ClientOnly>
  )
}
```

## Integration with backend patterns

### tRPC bridges frontend and backend

tRPC provides end-to-end type safety from database to UI.

**Server procedure**:

```typescript
// server/routes/users.ts
import { publicProcedure, router } from '../trpc'
import { z } from 'zod'

export const usersRouter = router({
  getById: publicProcedure
    .input(z.string().uuid())
    .query(async ({ input, ctx }) => {
      const user = await ctx.db.users.findById(input)
      if (!user) {
        throw new TRPCError({ code: 'NOT_FOUND' })
      }
      return user
    }),

  create: publicProcedure
    .input(z.object({
      email: z.string().email(),
      name: z.string().min(2),
    }))
    .mutation(async ({ input, ctx }) => {
      const user = await ctx.db.users.create(input)
      return user
    }),
})
```

**Client usage**:

```typescript
// Fully typed - no manual type definitions needed!
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading } = api.users.getById.useQuery(userId)
  const createUser = api.users.create.useMutation()

  // TypeScript knows:
  // - user type from server return type
  // - createUser input type from server input schema
  // - Error types from TRPCError
}
```

### Where to apply Effect-TS in UI layer

Reserve Effect-TS for complex client-side effects that benefit from explicit effect management:

```typescript
import { Effect } from 'effect'

// Example: Complex form submission with retries and fallbacks
const submitFormWithRetry = (data: FormData) =>
  Effect.gen(function* (_) {
    // Attempt primary API
    const result = yield* _(
      Effect.tryPromise({
        try: () => api.users.create.mutate(data),
        catch: (error) => new SubmitError({ cause: error }),
      })
    )

    return result
  }).pipe(
    // Retry with exponential backoff
    Effect.retry({
      times: 3,
      schedule: Schedule.exponential('100 millis'),
    }),
    // Fallback to queue for offline
    Effect.catchAll(() =>
      Effect.succeed(queueForLater(data))
    )
  )
```

Most UI code should use simpler patterns (TanStack Query, tRPC) unless:
- Complex retry/fallback logic required
- Multiple dependent effects need composition
- Advanced concurrency control needed

See @~/.claude/skills/preferences-typescript-nodejs-development/SKILL.md for Effect-TS patterns in backend code.

## Testing UI components

### Testing Library patterns

```typescript
import { render, screen, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

describe('UserForm', () => {
  it('validates all fields and shows errors', async () => {
    const user = userEvent.setup()
    render(<UserForm />)

    // Submit without filling fields
    await user.click(screen.getByRole('button', { name: /submit/i }))

    // All validation errors should appear
    await waitFor(() => {
      expect(screen.getByText(/invalid email/i)).toBeInTheDocument()
      expect(screen.getByText(/name too short/i)).toBeInTheDocument()
      expect(screen.getByText(/must be 18\+/i)).toBeInTheDocument()
    })
  })
})
```

### Testing with TanStack Query

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

function createWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  })

  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}

describe('UserList', () => {
  it('fetches and displays users', async () => {
    render(<UserList />, { wrapper: createWrapper() })

    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument()
    })
  })
})
```

## Integration with algebraic data types

See @~/.claude/skills/preferences-algebraic-data-types/SKILL.md for ADT patterns in TypeScript.

**Use discriminated unions for component state machines**:

```typescript
// Form state as sum type
type FormState =
  | { status: 'idle' }
  | { status: 'validating' }
  | { status: 'submitting' }
  | { status: 'success'; data: User }
  | { status: 'error'; error: Error }

function FormComponent() {
  const [state, setState] = useState<FormState>({ status: 'idle' })

  // Pattern match on state
  switch (state.status) {
    case 'idle':
      return <FormFields onSubmit={handleSubmit} />
    case 'validating':
      return <FormFields disabled />
    case 'submitting':
      return <FormFields disabled><Spinner /></FormFields>
    case 'success':
      return <SuccessMessage user={state.data} />
    case 'error':
      return <ErrorMessage error={state.error} />
  }
}
```

**Use branded types for type-safe IDs**:

```typescript
import { Brand } from '@/lib/utils'

type UserId = Brand<string, 'UserId'>
type PostId = Brand<string, 'PostId'>

// Type error if you pass PostId to function expecting UserId
function getUser(id: UserId): Promise<User> { /* ... */ }
function getPost(id: PostId): Promise<Post> { /* ... */ }
```
