---
name: frontend-patterns
description: Modern frontend architecture patterns for React, Next.js, and TypeScript including component composition, state management, performance optimization, accessibility, and responsive design. Use when building UI components, implementing frontend features, optimizing performance, or working with React/Next.js applications.
---

# Frontend Development Patterns

This skill provides comprehensive guidance for modern frontend development using React, Next.js, TypeScript, and related technologies.

## Component Architecture

### Component Composition Patterns

**Container/Presentational Pattern**:
```typescript
// Presentational component (pure, reusable)
interface UserCardProps {
  name: string;
  email: string;
  avatar: string;
  onEdit: () => void;
}

function UserCard({ name, email, avatar, onEdit }: UserCardProps) {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
      <button onClick={onEdit}>Edit</button>
    </div>
  );
}

// Container component (handles logic, state, data fetching)
function UserCardContainer({ userId }: { userId: string }) {
  const { data: user, isLoading } = useUser(userId);
  const { mutate: updateUser } = useUpdateUser();

  if (isLoading) return <Skeleton />;
  if (!user) return <NotFound />;

  return <UserCard {...user} onEdit={() => updateUser(user.id)} />;
}
```

**Compound Components Pattern**:
```typescript
// Flexible, composable API
<Tabs defaultValue="profile">
  <TabsList>
    <TabsTrigger value="profile">Profile</TabsTrigger>
    <TabsTrigger value="settings">Settings</TabsTrigger>
  </TabsList>
  <TabsContent value="profile">
    <ProfileForm />
  </TabsContent>
  <TabsContent value="settings">
    <SettingsForm />
  </TabsContent>
</Tabs>
```

### Component Organization

```
components/
├── ui/                    # Primitive components (buttons, inputs)
│   ├── button.tsx
│   ├── input.tsx
│   └── card.tsx
├── forms/                 # Form components
│   ├── login-form.tsx
│   └── register-form.tsx
├── features/              # Feature-specific components
│   ├── user-profile/
│   │   ├── profile-header.tsx
│   │   ├── profile-stats.tsx
│   │   └── index.ts
│   └── dashboard/
│       ├── dashboard-grid.tsx
│       └── dashboard-card.tsx
└── layouts/               # Layout components
    ├── main-layout.tsx
    └── auth-layout.tsx
```

## State Management

### Local State (useState)

Use for:
- Component-specific UI state
- Form inputs
- Toggles, modals

```typescript
function SearchBar() {
  const [query, setQuery] = useState('');
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      {isOpen && <SearchResults query={query} />}
    </div>
  );
}
```

### Global State (Zustand)

Use for:
- User authentication state
- Theme preferences
- Shopping cart
- Cross-component shared state

```typescript
import create from 'zustand';

interface UserStore {
  user: User | null;
  setUser: (user: User) => void;
  logout: () => void;
}

export const useUserStore = create<UserStore>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  logout: () => set({ user: null }),
}));

// Usage
function Header() {
  const user = useUserStore((state) => state.user);
  const logout = useUserStore((state) => state.logout);

  return <div>{user ? user.name : 'Guest'}</div>;
}
```

### Server State (React Query / TanStack Query)

Use for:
- API data fetching
- Caching API responses
- Optimistic updates
- Background refetching

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Fetch data
function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });

  if (isLoading) return <Skeleton />;
  if (error) return <Error error={error} />;

  return <div>{data.name}</div>;
}

// Mutations with optimistic updates
function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (user: User) => api.updateUser(user),
    onMutate: async (newUser) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['user', newUser.id] });

      // Snapshot previous value
      const previous = queryClient.getQueryData(['user', newUser.id]);

      // Optimistically update
      queryClient.setQueryData(['user', newUser.id], newUser);

      return { previous };
    },
    onError: (err, newUser, context) => {
      // Rollback on error
      queryClient.setQueryData(['user', newUser.id], context?.previous);
    },
    onSettled: (newUser) => {
      // Refetch after mutation
      queryClient.invalidateQueries({ queryKey: ['user', newUser.id] });
    },
  });
}
```

## Performance Optimization

### 1. Memoization

**useMemo** (expensive calculations):
```typescript
function ProductList({ products }: { products: Product[] }) {
  const sortedProducts = useMemo(
    () => products.sort((a, b) => b.price - a.price),
    [products]
  );

  return <div>{sortedProducts.map(...)}</div>;
}
```

**useCallback** (prevent re-renders):
```typescript
function Parent() {
  const [count, setCount] = useState(0);

  // ✅ Memoized - Child won't re-render unless count changes
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []);

  return <Child onClick={handleClick} />;
}

const Child = memo(function Child({ onClick }: { onClick: () => void }) {
  console.log('Child rendered');
  return <button onClick={onClick}>Click</button>;
});
```

**React.memo** (prevent component re-renders):
```typescript
const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
  // Only re-renders if data changes
  return <div>{/* expensive rendering */}</div>;
});
```

### 2. Code Splitting

**Route-based splitting** (Next.js automatic):
```typescript
// app/dashboard/page.tsx - automatically code split
export default function DashboardPage() {
  return <Dashboard />;
}
```

**Component-level splitting**:
```typescript
import dynamic from 'next/dynamic';

const HeavyChart = dynamic(() => import('@/components/heavy-chart'), {
  loading: () => <Skeleton />,
  ssr: false, // Don't render on server
});

function Analytics() {
  return <HeavyChart data={chartData} />;
}
```

### 3. Image Optimization

```typescript
import Image from 'next/image';

// ✅ Optimized - Next.js Image component
<Image
  src="/hero.jpg"
  alt="Hero image"
  width={800}
  height={600}
  priority // Load immediately for LCP
  placeholder="blur"
  blurDataURL="data:image/..."
/>

// ❌ Not optimized
<img src="/hero.jpg" alt="Hero" />
```

### 4. Lazy Loading

```typescript
import { lazy, Suspense } from 'react';

const Comments = lazy(() => import('./comments'));

function Post() {
  return (
    <div>
      <PostContent />
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments postId={postId} />
      </Suspense>
    </div>
  );
}
```

### 5. Virtual Scrolling

```typescript
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px` }}>
        {virtualizer.getVirtualItems().map((virtualRow) => (
          <div
            key={virtualRow.index}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Accessibility (a11y)

### Semantic HTML

```typescript
// ✅ Semantic
<nav>
  <ul>
    <li><a href="/home">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>

// ❌ Non-semantic
<div>
  <div>
    <div onClick={goHome}>Home</div>
    <div onClick={goAbout}>About</div>
  </div>
</div>
```

### ARIA Attributes

```typescript
<button
  aria-label="Close dialog"
  aria-expanded={isOpen}
  aria-controls="dialog-content"
  onClick={toggle}
>
  <X aria-hidden="true" />
</button>

<div
  id="dialog-content"
  role="dialog"
  aria-modal="true"
  aria-labelledby="dialog-title"
>
  <h2 id="dialog-title">Dialog Title</h2>
  {content}
</div>
```

### Keyboard Navigation

```typescript
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const [focusedIndex, setFocusedIndex] = useState(0);

  const handleKeyDown = (e: KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex((i) => Math.min(i + 1, items.length - 1));
        break;
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex((i) => Math.max(i - 1, 0));
        break;
      case 'Enter':
        selectItem(items[focusedIndex]);
        break;
      case 'Escape':
        setIsOpen(false);
        break;
    }
  };

  return (
    <div onKeyDown={handleKeyDown} role="combobox">
      {/* dropdown content */}
    </div>
  );
}
```

### Focus Management

```typescript
import { useRef, useEffect } from 'react';

function Modal({ isOpen, onClose }: ModalProps) {
  const closeButtonRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) {
      // Focus close button when modal opens
      closeButtonRef.current?.focus();

      // Trap focus within modal
      const handleTab = (e: KeyboardEvent) => {
        // Implement focus trap logic
      };

      document.addEventListener('keydown', handleTab);
      return () => document.removeEventListener('keydown', handleTab);
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div role="dialog" aria-modal="true">
      <button ref={closeButtonRef} onClick={onClose}>
        Close
      </button>
      {content}
    </div>
  );
}
```

## Form Patterns

### Controlled Forms with Validation

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import * as z from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  age: z.number().min(18, 'Must be 18 or older'),
});

type FormData = z.infer<typeof schema>;

function RegistrationForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data: FormData) => {
    await api.register(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('email')} type="email" />
        {errors.email && <span>{errors.email.message}</span>}
      </div>

      <div>
        <input {...register('password')} type="password" />
        {errors.password && <span>{errors.password.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

### Form State Management

```typescript
// Optimistic updates
const { mutate } = useMutation({
  mutationFn: updateUser,
  onMutate: async (newData) => {
    // Cancel outgoing queries
    await queryClient.cancelQueries({ queryKey: ['user', userId] });

    // Snapshot previous
    const previous = queryClient.getQueryData(['user', userId]);

    // Optimistically update UI
    queryClient.setQueryData(['user', userId], newData);

    return { previous };
  },
  onError: (err, newData, context) => {
    // Rollback on error
    queryClient.setQueryData(['user', userId], context?.previous);
    toast.error('Update failed');
  },
  onSuccess: () => {
    toast.success('Updated successfully');
  },
});
```

## Error Handling

### Error Boundaries

```typescript
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: any) {
    console.error('Error boundary caught:', error, errorInfo);
    // Log to error tracking service
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div>
          <h2>Something went wrong</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={<ErrorFallback />}>
  <App />
</ErrorBoundary>
```

### Async Error Handling

```typescript
function DataComponent() {
  const { data, error, isError, isLoading } = useQuery({
    queryKey: ['data'],
    queryFn: fetchData,
    retry: 3,
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
  });

  if (isLoading) return <Skeleton />;
  if (isError) return <ErrorDisplay error={error} />;

  return <DisplayData data={data} />;
}
```

## Responsive Design

### Mobile-First Approach

```typescript
// Tailwind CSS (mobile-first)
<div className="
  w-full              /* Full width on mobile */
  md:w-1/2           /* Half width on tablets */
  lg:w-1/3           /* Third width on desktop */
  p-4                /* Padding 16px */
  md:p-6             /* Padding 24px on tablets+ */
">
  Content
</div>
```

### Responsive Hooks

```typescript
import { useMediaQuery } from '@/hooks/use-media-query';

function ResponsiveLayout() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(min-width: 769px) and (max-width: 1024px)');
  const isDesktop = useMediaQuery('(min-width: 1025px)');

  if (isMobile) return <MobileLayout />;
  if (isTablet) return <TabletLayout />;
  return <DesktopLayout />;
}
```

## Data Fetching Strategies

### Server Components (Next.js 14+)

```typescript
// app/users/page.tsx - Server Component
async function UsersPage() {
  // Fetched on server
  const users = await db.user.findMany();

  return <UserList users={users} />;
}
```

### Client Components with React Query

```typescript
'use client';

function UserList() {
  const { data: users, isLoading } = useQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });

  if (isLoading) return <UsersLoading />;

  return <div>{users.map(user => <UserCard key={user.id} {...user} />)}</div>;
}
```

### Parallel Data Fetching

```typescript
function Dashboard() {
  const { data: user } = useQuery({ queryKey: ['user'], queryFn: fetchUser });
  const { data: stats } = useQuery({ queryKey: ['stats'], queryFn: fetchStats });
  const { data: posts } = useQuery({ queryKey: ['posts'], queryFn: fetchPosts });

  // All three queries run in parallel
  return <div>...</div>;
}
```

### Dependent Queries

```typescript
function UserPosts({ userId }: { userId: string }) {
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  const { data: posts } = useQuery({
    queryKey: ['posts', user?.id],
    queryFn: () => fetchUserPosts(user!.id),
    enabled: !!user, // Only fetch after user is loaded
  });

  return <div>...</div>;
}
```

## TypeScript Patterns

### Prop Types

```typescript
// Basic props
interface ButtonProps {
  children: ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

// Props with generic
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => ReactNode;
  keyExtractor: (item: T) => string;
}

// Props extending HTML attributes
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}
```

### Type-Safe API Responses

```typescript
// API response types
interface ApiResponse<T> {
  data: T;
  error?: never;
}

interface ApiError {
  data?: never;
  error: {
    code: string;
    message: string;
  };
}

type ApiResult<T> = ApiResponse<T> | ApiError;

// Usage
async function fetchUser(id: string): Promise<ApiResult<User>> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}
```

## Testing Patterns

### Component Testing

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { LoginForm } from './login-form';

test('submits form with email and password', async () => {
  const onSubmit = jest.fn();

  render(<LoginForm onSubmit={onSubmit} />);

  fireEvent.change(screen.getByLabelText('Email'), {
    target: { value: 'test@example.com' },
  });

  fireEvent.change(screen.getByLabelText('Password'), {
    target: { value: 'password123' },
  });

  fireEvent.click(screen.getByRole('button', { name: 'Login' }));

  await waitFor(() => {
    expect(onSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123',
    });
  });
});
```

### Mock API Calls

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.get('/api/users/:id', (req, res, ctx) => {
    return res(ctx.json({
      id: req.params.id,
      name: 'Test User',
      email: 'test@example.com',
    }));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

test('displays user data', async () => {
  const queryClient = new QueryClient();

  render(
    <QueryClientProvider client={queryClient}>
      <UserProfile userId="123" />
    </QueryClientProvider>
  );

  expect(await screen.findByText('Test User')).toBeInTheDocument();
});
```

## Build Optimization

### Bundle Analysis

```bash
# Next.js bundle analyzer
npm install @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // config
});

# Run analysis
ANALYZE=true npm run build
```

### Tree Shaking

```typescript
// ✅ Named imports (tree-shakeable)
import { Button } from '@/components/ui/button';

// ❌ Namespace import (includes everything)
import * as UI from '@/components/ui';
```

### Dynamic Imports

```typescript
// Import only when needed
async function handleExport() {
  const { exportToPDF } = await import('@/lib/pdf-export');
  await exportToPDF(data);
}
```

## Common Frontend Mistakes to Avoid

1. **Prop drilling**: Use Context or state management library instead
2. **Unnecessary re-renders**: Use memo, useMemo, useCallback appropriately
3. **Missing loading states**: Always show loading indicators
4. **No error boundaries**: Catch errors before they break the app
5. **Inline functions in JSX**: Causes re-renders, use useCallback
6. **Large bundle sizes**: Code split and lazy load
7. **Missing alt text**: All images need descriptive alt text
8. **Inaccessible forms**: Use proper labels and ARIA
9. **Console.log in production**: Remove or use proper logging
10. **Mixing server and client code**: Know Next.js boundaries

## Performance Metrics (Core Web Vitals)

### LCP (Largest Contentful Paint)
**Target**: < 2.5 seconds

**Optimize**:
- Preload critical images
- Use Next.js Image component
- Minimize render-blocking resources
- Use CDN for assets

### FID (First Input Delay)
**Target**: < 100 milliseconds

**Optimize**:
- Minimize JavaScript execution
- Code split large bundles
- Use web workers for heavy computation
- Defer non-critical JavaScript

### CLS (Cumulative Layout Shift)
**Target**: < 0.1

**Optimize**:
- Set explicit width/height on images
- Reserve space for ads/embeds
- Avoid inserting content above existing content
- Use CSS transforms instead of layout properties

## When to Use This Skill

Use this skill when:
- Building React or Next.js components
- Implementing frontend features
- Optimizing frontend performance
- Debugging rendering issues
- Setting up state management
- Implementing forms
- Ensuring accessibility
- Working with responsive design
- Fetching and caching data
- Testing frontend code

---

**Remember**: Modern frontend development is about creating fast, accessible, and delightful user experiences. Follow these patterns to build UIs that users love.
