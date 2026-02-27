---
name: frontend-guidelines
description: Core architectural rules, coding conventions, and tech stack guidelines for the Dog-Love Frontend. Reference this for all frontend code changes.
---

# Dog-Love Frontend — Agent Instructions

## Tech Stack

- Next.js 16 (App Router, Hybrid SPA)
- React 19, TypeScript 5 (strict)
- Tailwind CSS 4 + shadcn/ui
- React Query + Zustand + React Hook Form + Zod
- pnpm (필수, npm/yarn 사용 금지)

## Commands

```bash
pnpm dev            # 개발 서버 (http://localhost:3000)
pnpm build          # 프로덕션 빌드
pnpm lint           # ESLint 검사
pnpm typecheck      # TypeScript 타입 검사 (tsc --noEmit)
pnpm format:check   # Prettier 포맷 검사
pnpm format:fix     # Prettier 자동 수정
pnpm test           # vitest 유닛 테스트
pnpm test:watch     # vitest watch 모드
pnpm test:coverage  # 커버리지 포함 테스트 (70% 임계값)
pnpm test:e2e       # Playwright E2E 테스트
pnpm validate       # 전체 검증 (lint + typecheck + test + build)
```

## 커밋 규칙

- **Conventional Commits** 권장:
  - `feat`: 새로운 기능
  - `fix`: 버그 수정
  - `refactor`: 리팩토링
  - `style`: 스타일/포맷 변경
  - `docs`: 문서
  - `chore`: 설정 및 기타


## 코딩 컨벤션

- 컴포넌트: `PascalCase` named export (`export function MyComponent`)
- 유틸/훅: `camelCase` named export
- Named Export만 사용 (default export 금지) — tree-shaking 및 일관성
- Name (kebab-case)
- Path alias: `@/*` (프로젝트 루트)
- UI 컴포넌트: `components/ui/` (shadcn/ui)
- 전역 상태: `store/`
- React Query 훅: `hooks/query/`
- **`any` 사용 금지** — 반드시 `unknown` + 타입 가드 사용
  ```typescript
  // ❌ FORBIDDEN
  function handleData(data: any) { return data.id }

  // ✅ CORRECT
  function handleData(data: User) { return data.id }
  function processUnknown(data: unknown) {
    if (isUser(data)) return data.id
  }
  ```
- 환경변수는 반드시 `lib/env.ts`의 Zod 검증 값 사용 (`process.env` 직접 접근 금지)
  ```typescript
  // ❌ FORBIDDEN
  const key = process.env.API_KEY

  // ✅ CORRECT
  import { env } from '@/lib/env'
  const url = env.NEXT_PUBLIC_API_URL
  ```
- 함수형 프로그래밍 선호 — 클래스 사용 금지 (ErrorBoundary 제외)
- `cn()` 유틸리티로 조건부 클래스명 처리
- 보조동사 네이밍 사용 (e.g., `isLoading`, `hasError`, `canSubmit`)

## 아키텍처 (Hybrid SPA)

> **중요:** 이 프로젝트는 **하이브리드 SPA**입니다!
> - UI/페이지: Client Components (`'use client'`)
> - 인증(HttpOnly 쿠키): SSR 허용 (보안상 필수)

### 필수 규칙
- 모든 UI 컴포넌트와 페이지에 `'use client'` 디렉티브 필수
- 데이터 페칭은 React Query로 클라이언트에서 수행
- 백엔드 API는 별도 서버 (Spring Boot 등)
- 빌드 모드: `output: 'standalone'` (Node.js 서버 배포)

### SSR 예외 (인증/보안 전용)
- **API Routes** (`app/api/[...path]/route.ts`) — 백엔드 프록시 + 토큰 갱신용
- **Server Actions** (`app/actions/auth-actions.ts`) — HttpOnly 쿠키 관리용
- **Server-side cookies** (`next/headers`) — 인증 관련 함수에서만 사용

### 금지 사항 (UI/페이지에서)
- Server Components로 UI 렌더링 금지 — 모든 UI는 `'use client'` 필수
- 페이지에서 서버 사이드 데이터 페칭 금지 — React Query 사용
- `useEffect`로 데이터 페칭 금지 — React Query 사용

### 데이터 페칭 & 상태관리
- **React Query**: 모든 서버 상태 관리. `staleTime`/`gcTime` 적절히 설정
- **Zustand**: 글로벌 클라이언트 상태 (auth, UI). `persist` 미들웨어로 localStorage 지속
- **React Hook Form + Zod**: 모든 폼 처리

## 프로젝트 디렉토리 구조

```
app/
├── (auth)/              # 인증 라우트 그룹
│   ├── login/page.tsx
│   └── register/page.tsx
├── (admin)/              # 메인 앱 라우트 그룹 (보호됨)
│   ├── layout.tsx       # Layout with Header, Sidebar
│   ├── dashboard/page.tsx
│   └── users/
│       ├── page.tsx
│       └── [id]/page.tsx
├── layout.tsx           # Root layout (Providers)
└── page.tsx

components/
├── ui/                  # shadcn/ui (수정 금지)
└── common/              # Header, Sidebar, Loading, ErrorBoundary

lib/
├── api-client.ts        # Axios 인스턴스
├── auth/                # 인증 (auth-service, session-manager)
├── env.ts               # 환경변수 (Zod 검증)
└── utils.ts             # cn, format 등

hooks/                   # React Query 훅, 커스텀 훅
store/                   # Zustand stores

types/                   # TypeScript 타입 정의
public/                  # 정적 자산
tests/ & playwright/     # 테스트 설정
```

## 주의사항

- `"use client"` 디렉티브: 클라이언트 컴포넌트에만 사용
- 불필요한 코드, 주석, docstring 추가 금지
- 보안 취약점 (XSS, injection 등) 주의
- Early return 패턴 사용 — 에러 조건을 먼저 처리
- Guard clause로 사전 조건 검증
- ErrorBoundary로 앱 전체 에러 래핑

