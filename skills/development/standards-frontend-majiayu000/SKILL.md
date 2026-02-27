---
name: standards-frontend
description: Frontend component and UI development standards for modern React/Next.js applications. Includes React 19 patterns, Tailwind CSS v4, Server Components, accessibility, and the Anthropic frontend-design aesthetic philosophy.
---

# Frontend Standards

Standards for React/Next.js components, state management, styling, and UI patterns.

## When to Use

- Building UI components
- Implementing state management
- Adding styles or layouts
- Handling user interactions
- Creating Server Components or Server Actions

## Resources

| Resource | Use When |
|----------|----------|
| [component-patterns.md](resources/component-patterns.md) | React component architecture |
| [state-management.md](resources/state-management.md) | TanStack Query, Zustand, URL state |
| [styling.md](resources/styling.md) | Tailwind v4, CSS patterns |

## Quick Reference

### Server vs Client Components

```tsx
// ✅ Server Component (default) - No directive needed
export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);
  return <ProductDetails product={product} />;
}

// ✅ Client Component - Required for interactivity
'use client';

import { useState } from 'react';

export function AddToCartButton({ productId }: { productId: string }) {
  const [isAdding, setIsAdding] = useState(false);
  return (
    <button onClick={() => addToCart(productId)} disabled={isAdding}>
      {isAdding ? 'Adding...' : 'Add to Cart'}
    </button>
  );
}
```

### RSC Boundary Cheatsheet

| Feature | Server Component | Client Component |
|---------|-----------------|------------------|
| `async/await` | ✅ Yes | ❌ No |
| `useState`, `useEffect` | ❌ No | ✅ Yes |
| Browser APIs | ❌ No | ✅ Yes |
| Event handlers | ❌ No | ✅ Yes |
| Environment secrets | ✅ Yes | ❌ Never |

### Server Actions (React 19)

```tsx
// actions/cart.ts
'use server';

import { revalidatePath } from 'next/cache';
import { z } from 'zod';

const Schema = z.object({
  productId: z.string().uuid(),
  quantity: z.number().min(1).max(99),
});

export async function addToCart(formData: FormData) {
  const validated = Schema.parse({
    productId: formData.get('productId'),
    quantity: Number(formData.get('quantity')),
  });
  
  await db.cart.add(validated);
  revalidatePath('/cart');
  
  return { success: true };
}
```

### useActionState (React 19)

```tsx
'use client';

import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';
import { createUser } from '@/actions/user';

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create User'}
    </button>
  );
}

export function CreateUserForm() {
  const [state, formAction] = useActionState(createUser, { error: null });
  
  return (
    <form action={formAction}>
      <input name="email" type="email" required />
      <SubmitButton />
      {state.error && <p className="text-red-500">{state.error}</p>}
    </form>
  );
}
```

### Component Structure with CVA

```tsx
import { cva, type VariantProps } from 'class-variance-authority';
import { cn } from '@/lib/utils';

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground',
        outline: 'border border-input bg-background hover:bg-accent',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 px-3',
        lg: 'h-11 px-8',
      },
    },
    defaultVariants: { variant: 'default', size: 'default' },
  }
);

interface ButtonProps
  extends React.ComponentProps<'button'>,
    VariantProps<typeof buttonVariants> {
  isLoading?: boolean;
}

export function Button({ className, variant, size, isLoading, children, ...props }: ButtonProps) {
  return (
    <button className={cn(buttonVariants({ variant, size }), className)} {...props}>
      {isLoading ? <Spinner /> : children}
    </button>
  );
}
```

### TanStack Query

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000,
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: updateUser,
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: ['user', data.id] });
    },
  });
}
```

### Tailwind v4

```css
/* app/globals.css */
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.7 0.15 250);
  --font-sans: "Geist", system-ui, sans-serif;
}
```

## Design Philosophy

1. **Bold Aesthetic Direction**: Commit to a clear visual identity
2. **Typography First**: Use distinctive, memorable fonts
3. **Intentional Color**: Dominant colors with sharp accents
4. **Purposeful Animation**: High-impact moments over micro-interactions

```tsx
// ❌ AVOID: Generic AI aesthetics
// - Inter, Roboto, system fonts
// - Purple gradients on white

// ✅ PREFERRED: Distinctive choices
// - Unique fonts (GT Walsheim, Fraunces)
// - Bold color palettes
```

## Amp Tools to Use

- `finder` - Find existing component patterns
- `Read` - Check component library conventions
- `mcp__exa__get_code_context_exa` - Research latest React/Next.js patterns

## Related Skills

- `standards-global` - TypeScript conventions
- `standards-backend` - API integration patterns
- `standards-testing` - Component testing
