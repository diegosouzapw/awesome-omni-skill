---
name: frontend-dev-guidelines
description: React/Next.js enterprise best practices including component patterns, Link/Image optimization, shared components, and TypeScript conventions. Use this skill when creating components, pages, or working with Next.js features.
---

# Frontend Development Guidelines - React & Next.js

## Purpose

Ce skill fournit les meilleures pratiques entreprise pour le développement frontend avec React et Next.js, incluant:
- Conventions de codage TypeScript/React
- Utilisation optimale des composants Next.js (Link, Image)
- Architecture de composants réutilisables
- Patterns de performance et accessibilité

## When to Use This Skill

- Création de nouveaux composants React
- Utilisation de `Link` ou `Image` de Next.js
- Organisation de composants partagés
- Mise en place de pages Next.js
- Styling avec CSS Modules ou Tailwind
- Gestion d'état et data fetching

## Quick Reference

### Next.js Link - Usage Correct

```tsx
// CORRECT - Next.js 13+
import Link from 'next/link';

// Simple link
<Link href="/about">About</Link>

// Avec styling
<Link href="/about" className="text-blue-600 hover:underline">
  About
</Link>

// Lien dynamique
<Link href={`/posts/${post.id}`}>
  {post.title}
</Link>

// INCORRECT - N'utilisez JAMAIS
<Link href="/about">
  <a>About</a>  // Pas de <a> imbriqué depuis Next.js 13
</Link>
```

### Next.js Image - Usage Correct

```tsx
import Image from 'next/image';

// Image avec dimensions connues
<Image
  src="/hero.jpg"
  alt="Hero image"
  width={800}
  height={600}
  priority  // Pour les images above the fold
/>

// Image responsive (fill container)
<div className="relative w-full h-64">
  <Image
    src="/hero.jpg"
    alt="Hero image"
    fill
    className="object-cover"
    sizes="(max-width: 768px) 100vw, 50vw"
  />
</div>

// Image externe (nécessite configuration)
<Image
  src="https://example.com/image.jpg"
  alt="External image"
  width={400}
  height={300}
  unoptimized  // Si pas de loader configuré
/>
```

### Structure de Composant Standard

```tsx
// components/Button/Button.tsx
import { type ComponentPropsWithoutRef, forwardRef } from 'react';
import { cn } from '@/lib/utils';
import styles from './Button.module.css';

interface ButtonProps extends ComponentPropsWithoutRef<'button'> {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant = 'primary', size = 'md', isLoading, children, disabled, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={cn(
          styles.button,
          styles[variant],
          styles[size],
          isLoading && styles.loading,
          className
        )}
        disabled={disabled || isLoading}
        {...props}
      >
        {isLoading ? <Spinner size={size} /> : children}
      </button>
    );
  }
);

Button.displayName = 'Button';
```

## Resource Files

### Core Patterns
- [component-patterns.md](resources/component-patterns.md) - Architecture des composants
- [nextjs-components.md](resources/nextjs-components.md) - Link, Image, Head, Script
- [shared-components.md](resources/shared-components.md) - Composants réutilisables
- [typescript-conventions.md](resources/typescript-conventions.md) - Conventions TypeScript

### Styling & Performance
- [styling-patterns.md](resources/styling-patterns.md) - CSS Modules, Tailwind, best practices
- [performance.md](resources/performance.md) - Optimisation et lazy loading

### Data & State
- [data-fetching.md](resources/data-fetching.md) - Server Components, client fetching
- [state-management.md](resources/state-management.md) - useState, Context, Zustand

## Critical Rules

### 1. Toujours typer les props

```tsx
// CORRECT
interface CardProps {
  title: string;
  description?: string;
  onClick?: () => void;
}

// INCORRECT
const Card = (props: any) => { ... }
```

### 2. Utiliser les Server Components par défaut (Next.js 13+)

```tsx
// app/products/page.tsx - Server Component (par défaut)
async function ProductsPage() {
  const products = await fetchProducts(); // Fetch côté serveur
  return <ProductList products={products} />;
}

// components/AddToCart.tsx - Client Component (quand nécessaire)
'use client';
import { useState } from 'react';

export function AddToCart({ productId }: { productId: string }) {
  const [isAdding, setIsAdding] = useState(false);
  // ...
}
```

### 3. Optimiser les images avec priority et sizes

```tsx
// Above the fold - utiliser priority
<Image src="/hero.jpg" alt="" width={1200} height={600} priority />

// Responsive - toujours spécifier sizes
<Image
  src="/product.jpg"
  alt=""
  fill
  sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"
/>
```

### 4. Composants atomiques et composables

```tsx
// Bon - Composants petits et réutilisables
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>Content</CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>

// Éviter - Composant monolithique avec trop de props
<Card
  title="Title"
  content="Content"
  buttonText="Action"
  onButtonClick={() => {}}
  showFooter
  footerVariant="centered"
/>
```

### 5. Conventions de nommage

| Type | Convention | Exemple |
|------|------------|---------|
| Components | PascalCase | `UserProfile.tsx` |
| Hooks | camelCase, préfixe `use` | `useAuth.ts` |
| Utilities | camelCase | `formatDate.ts` |
| Constants | SCREAMING_SNAKE | `API_BASE_URL` |
| Types/Interfaces | PascalCase | `UserProfile` |
| CSS Modules | camelCase | `styles.cardHeader` |

## Anti-Patterns to Avoid

### DON'T: Wrapper <a> dans Link

```tsx
// INCORRECT (Next.js 13+)
<Link href="/about">
  <a className="link">About</a>
</Link>

// CORRECT
<Link href="/about" className="link">About</Link>
```

### DON'T: Images sans dimensions

```tsx
// INCORRECT - Cause layout shift
<Image src="/photo.jpg" alt="" />

// CORRECT
<Image src="/photo.jpg" alt="" width={400} height={300} />
// ou avec fill + container dimensionné
```

### DON'T: Props spreading sans restriction

```tsx
// INCORRECT - Risque de sécurité et props non voulues
const Input = (props: any) => <input {...props} />;

// CORRECT
interface InputProps extends ComponentPropsWithoutRef<'input'> {
  label?: string;
}
const Input = ({ label, className, ...props }: InputProps) => (
  <input className={cn('input', className)} {...props} />
);
```

### DON'T: useEffect pour le data fetching (Next.js)

```tsx
// INCORRECT - Dans Next.js 13+
'use client';
function Products() {
  const [products, setProducts] = useState([]);
  useEffect(() => {
    fetch('/api/products').then(r => r.json()).then(setProducts);
  }, []);
  return <ProductList products={products} />;
}

// CORRECT - Server Component
async function Products() {
  const products = await fetch('https://api.example.com/products').then(r => r.json());
  return <ProductList products={products} />;
}
```

## File Organization

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Route groups
│   │   ├── login/
│   │   └── register/
│   ├── products/
│   │   ├── [id]/
│   │   │   └── page.tsx
│   │   └── page.tsx
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ui/                # Composants de base réutilisables
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.module.css
│   │   │   └── index.ts
│   │   ├── Card/
│   │   ├── Input/
│   │   └── index.ts       # Barrel export
│   ├── features/          # Composants spécifiques aux features
│   │   ├── auth/
│   │   └── products/
│   └── layouts/           # Layouts partagés
├── hooks/                 # Custom hooks
├── lib/                   # Utilities et configurations
├── types/                 # Types TypeScript globaux
└── styles/               # Styles globaux
```

## Performance Checklist

- [ ] Images avec `priority` pour above-the-fold
- [ ] `sizes` défini pour toutes les images responsive
- [ ] Server Components utilisés par défaut
- [ ] `'use client'` seulement quand nécessaire
- [ ] Dynamic imports pour les composants lourds
- [ ] Pas de dépendances inutiles dans useEffect
- [ ] Keys stables sur les listes (pas d'index)
- [ ] Memoization (useMemo, useCallback) quand justifié

## Related Skills

- Use **error-tracking** for Sentry integration in React
- Use **route-tester** for API testing
