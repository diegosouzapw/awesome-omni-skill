---
name: frontend-developer
description: React components, UI library, state management ve user experience geliştirmek için kullanılır. Zustand, TanStack Query, Tailwind CSS ve accessibility konularında uzman.
---

# Frontend Developer Skill

React components, UI library, state management ve user experience geliştirir.

## When to Use

- React component oluştururken veya düzenlerken
- State management (Zustand) implementasyonu yaparken
- Form handling (React Hook Form + Zod) eklerken
- Data fetching (TanStack Query) implementasyonu yaparken
- Loading states ve skeleton screens eklerken
- Empty states ve error boundaries yazarken
- Responsive design (Tailwind CSS) yaparken
- PWA features geliştirirken
- Accessibility (ARIA) iyileştirmeleri yaparken

## Instructions

### Görevler
- React component development (ui, shared, features)
- State management (Zustand stores)
- Form handling (React Hook Form + Zod)
- Data fetching (TanStack Query)
- Loading states ve skeleton screens
- Empty states ve error boundaries
- Responsive design (Tailwind CSS)
- PWA features (manifest, service worker)
- Accessibility (ARIA labels, keyboard navigation)

### Kurallar
- Components functional components olmalı
- TypeScript strict mode kullan
- Tailwind CSS class names organized (use clsx + tailwind-merge)
- Component props TypeScript interfaces ile tanımlı
- No prop drilling (use Zustand or React Context)
- Error boundaries critical components'lerde
- Loading states tüm async operations'da
- Turkish language strings (no hardcoded English)

### Kod Kalitesi
- Component names PascalCase
- File names kebab-case (my-component.tsx)
- Props destructuring
- useMemo ve useCallback optimization için
- Consistent naming conventions
- Reusable components (DRY principle)
- Accessibility best practices

### Dosya Yapısı

```
src/components/
├── ui/              # Radix UI components
│   ├── button.tsx
│   ├── input.tsx
│   ├── dialog.tsx
│   └── ...
├── shared/          # Shared application components
│   ├── data-table.tsx
│   ├── stat-card.tsx
│   ├── empty-state.tsx
│   └── ...
└── features/        # Feature-specific components
    ├── members/
    ├── donations/
    └── social-aid/
```

### Component Example

```tsx
import { cn } from "@/lib/utils";

interface StatCardProps {
  title: string;
  value: string | number;
  description?: string;
  icon?: React.ReactNode;
  trend?: "up" | "down" | "neutral";
  className?: string;
}

export function StatCard({
  title,
  value,
  description,
  icon,
  trend,
  className,
}: StatCardProps) {
  return (
    <div
      className={cn(
        "rounded-lg border bg-card p-6 shadow-sm",
        className
      )}
    >
      <div className="flex items-center justify-between">
        <p className="text-sm font-medium text-muted-foreground">
          {title}
        </p>
        {icon && (
          <div className="text-muted-foreground">{icon}</div>
        )}
      </div>
      <div className="mt-2">
        <p className="text-2xl font-bold">{value}</p>
        {description && (
          <p className="text-xs text-muted-foreground mt-1">
            {description}
          </p>
        )}
      </div>
    </div>
  );
}
```

### TanStack Query Example

```tsx
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";

// Fetch members
export function useMembers() {
  return useQuery({
    queryKey: ["members"],
    queryFn: async () => {
      const res = await fetch("/api/members");
      if (!res.ok) throw new Error("Üyeler yüklenemedi");
      return res.json();
    },
  });
}

// Create member mutation
export function useCreateMember() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async (data: CreateMemberInput) => {
      const res = await fetch("/api/members", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data),
      });
      if (!res.ok) throw new Error("Üye oluşturulamadı");
      return res.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["members"] });
    },
  });
}
```

### Zustand Store Example

```typescript
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface SidebarState {
  isOpen: boolean;
  toggle: () => void;
  setOpen: (open: boolean) => void;
}

export const useSidebarStore = create<SidebarState>()(
  persist(
    (set) => ({
      isOpen: true,
      toggle: () => set((state) => ({ isOpen: !state.isOpen })),
      setOpen: (open) => set({ isOpen: open }),
    }),
    {
      name: "sidebar-storage",
    }
  )
);
```
