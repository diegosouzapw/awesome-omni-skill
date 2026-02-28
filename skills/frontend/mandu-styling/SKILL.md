---
name: Mandu Styling
description: Tailwind CSS v4 integration and Island styling patterns for Mandu Framework
metadata:
  version: "2.0.0"
  author: mandu
  tailwind: "v4.1+"
globs:
  - "**/*.css"
  - "**/*.module.css"
  - "app/globals.css"
---

# Mandu Styling Skill

Mandu Island 아키텍처에 최적화된 Tailwind CSS v4 스타일링 가이드입니다.

## 핵심 원칙

1. **Zero-Runtime**: 빌드 타임 CSS 생성 (SSR 호환)
2. **CSS-First**: JavaScript 설정 대신 CSS에서 테마 정의
3. **Island 격리**: 컴포넌트 간 스타일 충돌 방지
4. **Auto Integration**: Mandu가 Tailwind v4 자동 감지 및 빌드

## 권장 스택

```
Primary:    Tailwind CSS v4 + clsx/tailwind-merge
Alternative: CSS Modules (최소 의존성)
```

## 빠른 시작

### 1. 설치

```bash
bun add -d tailwindcss@^4.1 @tailwindcss/cli@^4.1
bun add clsx tailwind-merge
```

### 2. app/globals.css

```css
@import "tailwindcss";

@theme {
  --color-primary: hsl(222.2 47.4% 11.2%);
  --color-primary-foreground: hsl(210 40% 98%);
  --color-background: hsl(0 0% 100%);
  --color-foreground: hsl(222.2 84% 4.9%);
  --radius-md: 0.5rem;
  --font-sans: 'Inter', system-ui, sans-serif;
}

body {
  background-color: var(--color-background);
  color: var(--color-foreground);
}
```

### 3. Mandu 자동 통합

```
mandu dev 실행 시:
├─ @import "tailwindcss" 감지
├─ Tailwind CLI --watch 자동 시작
├─ 출력: .mandu/client/globals.css
├─ SSR에서 <link> 자동 주입
└─ CSS 변경 시 HMR 핫 리로드
```

## Island 스타일링

```tsx
// src/client/widgets/counter/Counter.island.tsx
"use client";

import { cn } from "@/shared/lib/utils";

export function CounterIsland({ variant = "default" }) {
  return (
    <button className={cn(
      "px-4 py-2 rounded-md font-medium transition-colors",
      variant === "default"
        ? "bg-primary text-primary-foreground hover:bg-primary/90"
        : "border border-primary text-primary hover:bg-primary/10"
    )}>
      Count: 0
    </button>
  );
}
```

## cn 유틸리티

```typescript
// src/client/shared/lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

## Custom Utilities

```css
/* v4 방식: @utility 사용 */
@utility text-balance {
  text-wrap: balance;
}

@utility scrollbar-hide {
  -ms-overflow-style: none;
  scrollbar-width: none;
  &::-webkit-scrollbar { display: none; }
}
```

## v4 주요 변경사항

| v3 | v4 |
|----|-----|
| `@tailwind base/utilities` | `@import "tailwindcss"` |
| `tailwind.config.ts` | `@theme { }` in CSS |
| `!bg-red-500` | `bg-red-500!` |
| `@layer utilities` | `@utility` |
| `bg-[--var]` | `bg-(--var)` |

> 자세한 내용은 `rules/style-tailwind-v4-gotchas.md` 참조

## 규칙 카테고리

| Category | Description | Rules |
|----------|-------------|-------|
| Setup | Tailwind v4 설정 | 1 |
| Gotchas | v4 주의사항/마이그레이션 | 1 |
| Island | Island 스타일 패턴 | 3 |
| Component | 컴포넌트 스타일 | 3 |
| Performance | 최적화 | 2 |
| Theme | 테마/다크모드 | 1 |

→ 세부 규칙은 `rules/` 폴더 참조

## Browser Support

```
Safari 16.4+ | Chrome 111+ | Firefox 128+
```

> IE 및 구형 브라우저 미지원. 레거시 프로젝트는 Tailwind v3 유지 권장.

Reference: [Tailwind CSS v4](https://tailwindcss.com/docs)
