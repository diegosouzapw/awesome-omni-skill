---
name: fullstack-modern
description: Apply when working with modern fullstack patterns including React/Vue, GraphQL, REST APIs, and headless architectures
globs:
  - "**/*.tsx"
  - "**/*.jsx"
  - "**/*.vue"
  - "**/api/**/*.ts"
  - "**/*.graphql"
  - "**/next.config.*"
---

# Modern Fullstack Integration Patterns

## Data Flow Architecture

### Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        Client                                │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   React/    │───▶│   State     │───▶│   UI        │     │
│  │   Vue       │    │   (Zustand/ │    │   Render    │     │
│  │   Components│    │   Pinia)    │    │             │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│         │                                                    │
│         ▼                                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │   Data Fetching (React Query / SWR / Apollo)        │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ HTTP / GraphQL
┌─────────────────────────────────────────────────────────────┐
│                        Server                                │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   API       │───▶│   Business  │───▶│   Data      │     │
│  │   Routes    │    │   Logic     │    │   Layer     │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

## Next.js API Routes

### Basic API Route

```typescript
// app/api/products/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { productService } from '@/lib/services/product-service';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const page = parseInt(searchParams.get('page') ?? '1');
  const limit = parseInt(searchParams.get('limit') ?? '10');
  const category = searchParams.get('category');

  try {
    const products = await productService.getProducts({
      page,
      limit,
      category: category ?? undefined,
    });

    return NextResponse.json(products);
  } catch (error) {
    console.error('Failed to fetch products:', error);
    return NextResponse.json(
      { error: 'Failed to fetch products' },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    // Validate request body
    const validationResult = validateCreateProduct(body);
    if (!validationResult.success) {
      return NextResponse.json(
        { errors: validationResult.errors },
        { status: 400 }
      );
    }

    const product = await productService.createProduct(body);
    return NextResponse.json(product, { status: 201 });
  } catch (error) {
    console.error('Failed to create product:', error);
    return NextResponse.json(
      { error: 'Failed to create product' },
      { status: 500 }
    );
  }
}
```

### Dynamic API Route

```typescript
// app/api/products/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

interface RouteParams {
  params: { id: string };
}

export async function GET(request: NextRequest, { params }: RouteParams) {
  const product = await productService.getById(params.id);
  
  if (!product) {
    return NextResponse.json(
      { error: 'Product not found' },
      { status: 404 }
    );
  }

  return NextResponse.json(product);
}

export async function PUT(request: NextRequest, { params }: RouteParams) {
  const body = await request.json();
  
  try {
    const product = await productService.update(params.id, body);
    return NextResponse.json(product);
  } catch (error) {
    if (error instanceof NotFoundError) {
      return NextResponse.json(
        { error: 'Product not found' },
        { status: 404 }
      );
    }
    throw error;
  }
}

export async function DELETE(request: NextRequest, { params }: RouteParams) {
  await productService.delete(params.id);
  return new NextResponse(null, { status: 204 });
}
```

## GraphQL Integration

### Query Definitions

```graphql
# queries/products.graphql
query GetProducts($first: Int!, $after: String, $category: String) {
  products(first: $first, after: $after, category: $category) {
    edges {
      node {
        id
        name
        slug
        price
        image {
          url
          alt
        }
        category {
          name
        }
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
    totalCount
  }
}

query GetProductBySlug($slug: String!) {
  product(slug: $slug) {
    id
    name
    slug
    description
    price
    images {
      url
      alt
    }
    category {
      name
      slug
    }
    variants {
      id
      name
      sku
      price
    }
  }
}

mutation AddToCart($productId: ID!, $quantity: Int!) {
  addToCart(input: { productId: $productId, quantity: $quantity }) {
    cart {
      id
      items {
        product {
          name
        }
        quantity
      }
      total
    }
  }
}
```

### GraphQL Client Setup

```typescript
// lib/graphql/client.ts
import { GraphQLClient } from 'graphql-request';

const endpoint = process.env.GRAPHQL_ENDPOINT!;
const apiKey = process.env.API_KEY!;

export const graphqlClient = new GraphQLClient(endpoint, {
  headers: {
    'x-api-key': apiKey,
  },
});

// With error handling wrapper
export async function fetchGraphQL<T>(
  query: string,
  variables?: Record<string, unknown>
): Promise<T> {
  try {
    return await graphqlClient.request<T>(query, variables);
  } catch (error) {
    console.error('GraphQL Error:', error);
    throw new Error('Failed to fetch data');
  }
}
```

### React Hook for GraphQL

```typescript
// hooks/useProducts.ts
import { useQuery } from '@tanstack/react-query';
import { graphqlClient } from '@/lib/graphql/client';
import { GetProductsDocument } from '@/generated/graphql';

interface UseProductsOptions {
  category?: string;
  limit?: number;
}

export function useProducts({ category, limit = 10 }: UseProductsOptions = {}) {
  return useQuery({
    queryKey: ['products', { category, limit }],
    queryFn: async () => {
      const data = await graphqlClient.request(GetProductsDocument, {
        first: limit,
        category,
      });
      return data.products;
    },
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
}
```

## Server Components + Client Data

### Server Component with Data

```tsx
// app/products/page.tsx
import { getProducts } from '@/lib/api/products';
import { ProductGrid } from '@/components/ProductGrid';
import { ProductFilters } from '@/components/ProductFilters';

interface PageProps {
  searchParams: { category?: string; page?: string };
}

export default async function ProductsPage({ searchParams }: PageProps) {
  const page = parseInt(searchParams.page ?? '1');
  const products = await getProducts({
    category: searchParams.category,
    page,
    limit: 12,
  });

  return (
    <main className="container mx-auto py-8">
      <h1 className="text-3xl font-bold mb-6">Products</h1>
      
      {/* Client component for interactivity */}
      <ProductFilters initialCategory={searchParams.category} />
      
      {/* Server-rendered product grid */}
      <ProductGrid products={products.items} />
      
      {/* Client component for pagination */}
      <Pagination
        currentPage={page}
        totalPages={products.totalPages}
      />
    </main>
  );
}
```

### Client Component with Mutations

```tsx
'use client';

import { useState } from 'react';
import { useMutation, useQueryClient } from '@tanstack/react-query';

interface AddToCartButtonProps {
  productId: string;
  productName: string;
}

export function AddToCartButton({ productId, productName }: AddToCartButtonProps) {
  const [quantity, setQuantity] = useState(1);
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: async (data: { productId: string; quantity: number }) => {
      const response = await fetch('/api/cart', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });
      
      if (!response.ok) {
        throw new Error('Failed to add to cart');
      }
      
      return response.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
    },
  });

  return (
    <div className="flex items-center gap-4">
      <input
        type="number"
        min="1"
        max="10"
        value={quantity}
        onChange={(e) => setQuantity(parseInt(e.target.value))}
        className="w-20 px-3 py-2 border rounded"
      />
      <button
        onClick={() => mutation.mutate({ productId, quantity })}
        disabled={mutation.isPending}
        className="px-6 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50"
      >
        {mutation.isPending ? 'Adding...' : 'Add to Cart'}
      </button>
      {mutation.isError && (
        <span className="text-red-500">Failed to add item</span>
      )}
    </div>
  );
}
```

## REST API Patterns

### API Client

```typescript
// lib/api/client.ts
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL;

interface RequestOptions extends RequestInit {
  params?: Record<string, string>;
}

class ApiClient {
  private baseUrl: string;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  private async request<T>(
    endpoint: string,
    options: RequestOptions = {}
  ): Promise<T> {
    const { params, ...init } = options;
    
    let url = `${this.baseUrl}${endpoint}`;
    if (params) {
      const searchParams = new URLSearchParams(params);
      url += `?${searchParams.toString()}`;
    }

    const response = await fetch(url, {
      ...init,
      headers: {
        'Content-Type': 'application/json',
        ...init.headers,
      },
    });

    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new ApiError(response.status, error.message ?? 'Request failed');
    }

    return response.json();
  }

  async get<T>(endpoint: string, params?: Record<string, string>): Promise<T> {
    return this.request<T>(endpoint, { method: 'GET', params });
  }

  async post<T>(endpoint: string, data: unknown): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  async put<T>(endpoint: string, data: unknown): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  async delete(endpoint: string): Promise<void> {
    await this.request(endpoint, { method: 'DELETE' });
  }
}

export const api = new ApiClient(API_BASE_URL);
```

### Type-Safe API Hooks

```typescript
// hooks/api/useProducts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { api } from '@/lib/api/client';
import type { Product, CreateProductInput, UpdateProductInput } from '@/types';

export function useProducts(params?: { category?: string; page?: number }) {
  return useQuery({
    queryKey: ['products', params],
    queryFn: () => api.get<{ items: Product[]; total: number }>('/products', {
      category: params?.category,
      page: String(params?.page ?? 1),
    }),
  });
}

export function useProduct(id: string) {
  return useQuery({
    queryKey: ['products', id],
    queryFn: () => api.get<Product>(`/products/${id}`),
    enabled: !!id,
  });
}

export function useCreateProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateProductInput) =>
      api.post<Product>('/products', data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });
}

export function useUpdateProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateProductInput }) =>
      api.put<Product>(`/products/${id}`, data),
    onSuccess: (product) => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
      queryClient.setQueryData(['products', product.id], product);
    },
  });
}

export function useDeleteProduct() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: string) => api.delete(`/products/${id}`),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });
}
```

## Environment Variables

### Configuration

```bash
# .env.local
# Server-only (not exposed to browser)
DATABASE_URL=postgresql://user:pass@localhost:5432/db
API_SECRET_KEY=sk_live_xxxxx
GRAPHQL_ENDPOINT=https://api.example.com/graphql

# Public (exposed to browser via NEXT_PUBLIC_ prefix)
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_SITE_URL=https://example.com
NEXT_PUBLIC_ANALYTICS_ID=UA-XXXXX
```

### Type-Safe Environment

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  // Server-only
  DATABASE_URL: z.string().url(),
  API_SECRET_KEY: z.string().min(1),
  GRAPHQL_ENDPOINT: z.string().url(),
  
  // Public
  NEXT_PUBLIC_API_URL: z.string().url(),
  NEXT_PUBLIC_SITE_URL: z.string().url(),
});

// Validate at build time
const parsed = envSchema.safeParse(process.env);

if (!parsed.success) {
  console.error('Invalid environment variables:', parsed.error.flatten());
  throw new Error('Invalid environment variables');
}

export const env = parsed.data;

// For client-side only variables
export const publicEnv = {
  apiUrl: process.env.NEXT_PUBLIC_API_URL!,
  siteUrl: process.env.NEXT_PUBLIC_SITE_URL!,
};
```

## Error Handling

### Error Boundaries

```tsx
'use client';

import { useEffect } from 'react';

interface ErrorBoundaryProps {
  error: Error & { digest?: string };
  reset: () => void;
}

export default function ErrorBoundary({ error, reset }: ErrorBoundaryProps) {
  useEffect(() => {
    // Log to error tracking service
    console.error('Error:', error);
  }, [error]);

  return (
    <div className="flex flex-col items-center justify-center min-h-[400px] p-8">
      <h2 className="text-2xl font-bold text-red-600 mb-4">
        Something went wrong
      </h2>
      <p className="text-gray-600 mb-6">
        {error.message || 'An unexpected error occurred'}
      </p>
      <button
        onClick={reset}
        className="px-6 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Try again
      </button>
    </div>
  );
}
```

### API Error Handling

```typescript
// lib/errors.ts
export class ApiError extends Error {
  constructor(
    public status: number,
    message: string,
    public code?: string
  ) {
    super(message);
    this.name = 'ApiError';
  }

  static isApiError(error: unknown): error is ApiError {
    return error instanceof ApiError;
  }
}

// Usage in components
function ProductPage() {
  const { data, error, isLoading } = useProduct(id);

  if (isLoading) return <Skeleton />;
  
  if (error) {
    if (ApiError.isApiError(error) && error.status === 404) {
      return <NotFound message="Product not found" />;
    }
    return <ErrorMessage error={error} />;
  }

  return <ProductDetails product={data} />;
}
```

## Optimistic Updates

```typescript
// hooks/useToggleFavorite.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';

export function useToggleFavorite() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (productId: string) => {
      const response = await fetch(`/api/favorites/${productId}`, {
        method: 'POST',
      });
      return response.json();
    },
    
    // Optimistic update
    onMutate: async (productId) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['products'] });

      // Snapshot previous value
      const previousProducts = queryClient.getQueryData(['products']);

      // Optimistically update
      queryClient.setQueryData(['products'], (old: Product[]) =>
        old.map((product) =>
          product.id === productId
            ? { ...product, isFavorite: !product.isFavorite }
            : product
        )
      );

      return { previousProducts };
    },
    
    // Rollback on error
    onError: (err, productId, context) => {
      queryClient.setQueryData(['products'], context?.previousProducts);
    },
    
    // Refetch after success or error
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] });
    },
  });
}
```

## Real-Time Updates

### WebSocket Integration

```typescript
// lib/websocket.ts
import { useEffect, useRef } from 'react';
import { useQueryClient } from '@tanstack/react-query';

export function useWebSocket(url: string) {
  const ws = useRef<WebSocket | null>(null);
  const queryClient = useQueryClient();

  useEffect(() => {
    ws.current = new WebSocket(url);

    ws.current.onmessage = (event) => {
      const message = JSON.parse(event.data);

      switch (message.type) {
        case 'product:updated':
          queryClient.invalidateQueries({ 
            queryKey: ['products', message.productId] 
          });
          break;
        case 'cart:updated':
          queryClient.invalidateQueries({ queryKey: ['cart'] });
          break;
        case 'notification':
          // Handle notification
          break;
      }
    };

    ws.current.onclose = () => {
      // Reconnect logic
      setTimeout(() => {
        ws.current = new WebSocket(url);
      }, 3000);
    };

    return () => {
      ws.current?.close();
    };
  }, [url, queryClient]);

  return {
    send: (data: unknown) => {
      ws.current?.send(JSON.stringify(data));
    },
  };
}
```

## SSR vs SSG vs ISR

### Static Generation (SSG)

```typescript
// For content that rarely changes
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map((post) => ({ slug: post.slug }));
}

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);
  return <Article post={post} />;
}
```

### Incremental Static Regeneration (ISR)

```typescript
// For content that updates periodically
// app/products/page.tsx
export const revalidate = 60; // Revalidate every 60 seconds

export default async function ProductsPage() {
  const products = await getProducts();
  return <ProductGrid products={products} />;
}
```

### Server-Side Rendering (SSR)

```typescript
// For personalized or real-time data
// app/dashboard/page.tsx
export const dynamic = 'force-dynamic';

export default async function DashboardPage() {
  const user = await getCurrentUser();
  const data = await getDashboardData(user.id);
  
  return <Dashboard user={user} data={data} />;
}
```

## Preview Mode / Draft Content

```typescript
// app/api/preview/route.ts
import { draftMode } from 'next/headers';
import { redirect } from 'next/navigation';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const secret = searchParams.get('secret');
  const slug = searchParams.get('slug');

  if (secret !== process.env.PREVIEW_SECRET) {
    return NextResponse.json({ error: 'Invalid token' }, { status: 401 });
  }

  // Enable draft mode
  draftMode().enable();

  // Redirect to the page
  redirect(slug ?? '/');
}

// app/api/exit-preview/route.ts
export async function GET() {
  draftMode().disable();
  redirect('/');
}
```

```typescript
// lib/api/content.ts
import { draftMode } from 'next/headers';

export async function getContent(slug: string) {
  const { isEnabled: isDraft } = draftMode();
  
  // Fetch draft or published content based on mode
  const content = await cms.getContent(slug, {
    preview: isDraft,
  });
  
  return content;
}
```
