---
name: common-rules
description: Tüm skill'ler için ortak kurallar ve standartlar. Git workflow, code style, documentation ve performance best practices içerir.
---

# Common Rules Skill

Tüm skill'ler için geçerli ortak kurallar ve standartlar.

## When to Use

- Git commit yaparken
- Pull request oluştururken
- Kod stili kontrol ederken
- Documentation yazarken
- Performance optimization yaparken
- Herhangi bir geliştirme yaparken referans olarak

## Instructions

### Git Workflow

```bash
# Branch naming conventions
feature/issue-description   # New features
bugfix/issue-description    # Bug fixes
hotfix/issue-description    # Production hotfixes
docs/issue-description      # Documentation updates
refactor/issue-description  # Code refactoring

# Conventional Commits
feat: add member search feature
fix: resolve pagination bug in donations list
docs: update API documentation
style: format code with prettier
refactor: simplify donation calculation logic
test: add unit tests for StatCard component
chore: update dependencies
perf: optimize member list query

# Examples
git checkout -b feature/member-search
git commit -m "feat: add member search with debounce"
git commit -m "fix: resolve date picker timezone issue"
git commit -m "docs: add API endpoint documentation"
```

### Pull Request Template

```markdown
## Açıklama
<!-- Bu PR ne yapıyor? -->

## Değişiklik Türü
- [ ] Yeni özellik (feat)
- [ ] Hata düzeltme (fix)
- [ ] Refactoring
- [ ] Dokümantasyon
- [ ] Test

## Test
- [ ] Unit testler geçiyor
- [ ] E2E testler geçiyor
- [ ] Manuel test yapıldı

## Checklist
- [ ] Kod stili kurallara uygun
- [ ] Self-review yapıldı
- [ ] Gerekli dokümantasyon eklendi
- [ ] Breaking change yok (veya belirtildi)

## Ekran Görüntüleri (varsa)
<!-- UI değişiklikleri için -->

## İlgili Issue
Closes #123
```

### Code Style

```typescript
// File naming: kebab-case
// my-component.tsx, use-debounce.ts, auth-middleware.ts

// Component naming: PascalCase
export function MemberListItem() { }
export function StatCard() { }

// Function naming: camelCase
function calculateTotalDonation() { }
function validateMemberInput() { }

// Constant naming: UPPER_SNAKE_CASE
const MAX_PAGE_SIZE = 100;
const API_TIMEOUT = 5000;

// TypeScript interfaces: PascalCase with I prefix (optional)
interface MemberProps { }
interface DonationFormData { }

// No console.log in production
// Use proper logging or remove before commit
if (process.env.NODE_ENV === 'development') {
  console.log('Debug:', data);
}
```

### ESLint & Prettier

```json
// .eslintrc.json
{
  "extends": ["next/core-web-vitals", "next/typescript"],
  "rules": {
    "no-console": "warn",
    "no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn"
  }
}

// .prettierrc
{
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80
}
```

### TypeScript Best Practices

```typescript
// Use strict mode
// tsconfig.json: "strict": true

// Define explicit types
interface Member {
  id: number;
  name: string;
  email: string;
  phone?: string;
  status: "active" | "inactive";
  createdAt: Date;
}

// Use utility types
type PartialMember = Partial<Member>;
type MemberWithoutId = Omit<Member, "id">;
type MemberKeys = keyof Member;

// Avoid 'any', use 'unknown' if needed
function parseJson(data: unknown): Member {
  // Type guard
  if (isMember(data)) {
    return data;
  }
  throw new Error("Invalid data");
}

// Type guards
function isMember(data: unknown): data is Member {
  return (
    typeof data === "object" &&
    data !== null &&
    "id" in data &&
    "name" in data
  );
}
```

### Documentation

```typescript
/**
 * Calculates the total donation amount for a member.
 * 
 * @param memberId - The ID of the member
 * @param options - Optional configuration
 * @param options.startDate - Start date for filtering
 * @param options.endDate - End date for filtering
 * @returns Promise with total amount and count
 * 
 * @example
 * const result = await calculateMemberDonations(123, {
 *   startDate: new Date('2024-01-01'),
 *   endDate: new Date('2024-12-31')
 * });
 * console.log(result.total); // 5000
 */
async function calculateMemberDonations(
  memberId: number,
  options?: {
    startDate?: Date;
    endDate?: Date;
  }
): Promise<{ total: number; count: number }> {
  // Implementation
}
```

### Performance Guidelines

```typescript
// 1. Code splitting (Next.js automatic)
import dynamic from "next/dynamic";

const HeavyComponent = dynamic(() => import("./HeavyComponent"), {
  loading: () => <Skeleton />,
  ssr: false,
});

// 2. Image optimization
import Image from "next/image";

<Image
  src="/photo.jpg"
  alt="Description"
  width={800}
  height={600}
  priority={isAboveFold}
  placeholder="blur"
  blurDataURL={blurUrl}
/>

// 3. Memoization
import { useMemo, useCallback, memo } from "react";

const MemoizedComponent = memo(function Component({ data }) {
  return <div>{data}</div>;
});

const expensiveValue = useMemo(() => {
  return computeExpensiveValue(deps);
}, [deps]);

const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);

// 4. Virtualization for long lists
import { useVirtualizer } from "@tanstack/react-virtual";

// 5. Debounce for search inputs
import { useDebounce } from "@/hooks/use-debounce";

const debouncedSearch = useDebounce(searchTerm, 300);
```

### Performance Targets

```markdown
Lighthouse Score Targets:
- Performance: > 90
- Accessibility: > 90
- Best Practices: > 90
- SEO: > 90

Core Web Vitals:
- LCP (Largest Contentful Paint): < 2.5s
- FID (First Input Delay): < 100ms
- CLS (Cumulative Layout Shift): < 0.1

First Contentful Paint: < 1.8s
Time to Interactive: < 3.0s
Bundle Size: Monitor and minimize
```

### Error Handling

```typescript
// Consistent error response format
interface ApiError {
  success: false;
  error: {
    code: string;
    message: string;
    details?: Record<string, string[]>;
  };
}

// Error codes
const ERROR_CODES = {
  VALIDATION_ERROR: "VALIDATION_ERROR",
  NOT_FOUND: "NOT_FOUND",
  UNAUTHORIZED: "UNAUTHORIZED",
  FORBIDDEN: "FORBIDDEN",
  INTERNAL_ERROR: "INTERNAL_ERROR",
  RATE_LIMITED: "RATE_LIMITED",
} as const;

// Error messages (Turkish)
const ERROR_MESSAGES = {
  VALIDATION_ERROR: "Girilen bilgiler geçersiz",
  NOT_FOUND: "Kayıt bulunamadı",
  UNAUTHORIZED: "Oturum açmanız gerekiyor",
  FORBIDDEN: "Bu işlem için yetkiniz yok",
  INTERNAL_ERROR: "Bir hata oluştu, lütfen tekrar deneyin",
  RATE_LIMITED: "Çok fazla istek, lütfen bekleyin",
};
```
