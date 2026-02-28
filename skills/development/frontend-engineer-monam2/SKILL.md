---
name: Frontend Engineer
description: 프론트엔드 개발 시 준수해야 할 아키텍처, 컨벤션, 워크플로우 가이드
---

# Agent: Frontend Engineer

## Mandate

SSOT 원칙과 접근성을 준수하는 사용자 인터페이스 구현

> **Codex 필수**: 모든 코드 작성 및 수정은 Codex MCP를 호출해 수행

---

## 개발 워크플로우

**컴포넌트 우선 개발 (Component First)**

```
1. 컴포넌트 스펙 정의
   └─ Props, 상태, 동작 정의

2. 컴포넌트 구현
   └─ 기능 구현 + 스타일링

3. Storybook 문서화
   └─ 스토리 작성 + 인터랙션 테스트

4. 페이지에 통합
   └─ 실제 데이터 연결
```

### 순서

| 단계 | 작업          | 산출물          |
| ---- | ------------- | --------------- |
| 1    | 컴포넌트 스펙 | 인터페이스 정의 |
| 2    | 컴포넌트 생성 | `*.tsx`         |
| 3    | Storybook     | `*.stories.tsx` |
| 4    | 페이지 통합   | `page.tsx`      |

- 스토리북 검증 후 페이지에 적용

---

## Architecture Standards

### 도메인 기반 구조 (Domain-Driven)

FSD(Feature-Sliced Design)와 유사하나 일부 계층을 단순화한 **도메인 중심 아키텍처**를 채택한다.

1. **`src/` 구조 도입**: 모든 소스 코드는 `src` 디렉토리 하위에 위치하며, `@` alias는 `src`를 가리킨다.
2. **`app/` (Pages)**:
   - 각 페이지는 하나의 도메인을 대변하거나 도메인들의 조합.
   - `page.tsx`(진입점)는 **오직 도메인 컴포넌트와 Shared 컴포넌트의 배치(Layout) 역할**만 수행.
   - 비즈니스 로직을 직접 포함하지 않음.

3. **`domains/`**:
   - 프로젝트 핵심 비즈니스 로직과 UI가 응집된 곳.
   - 구조: `/src/domains/[DomainName]/`
     - `components/`: 해당 도메인 전용 UI
     - `hooks/`: 해당 도메인 전용 로직
     - `entities/`: 타입 정의 (Interface, Type)
     - `schema/`: Zod 스키마
     - `api/`: API 호출 함수
     - `utils/`: 도메인 전용 유틸리티

4. **`shared/`**:
   - 프로젝트 전반에서 사용되는 범용 요소.
   - 구조: `/src/shared/`
     - `components/`: UI 라이브러리(Shadcn), 공통 모달 등
     - `hooks/`: 범용 훅 (useDebounce 등)
     - `utils/`: 날짜 포맷팅 등 순수 함수
     - `entities/`: 전역 타입 (API Response 등)

### 코딩 컨벤션

#### Import 정렬 규칙

길이 기준 **내림차순** 정렬:

1. React 및 외부 패키지
2. Domain 계층 (`@/domains/...`)
3. Shared 계층 (`@/shared/...`)
4. 상대 경로 (`./`, `../`)

#### Barrel Export

- 각 디렉토리(`components`, `hooks` 등)에 `index.ts`를 두어 외부에서 깔끔하게 import 하도록 함.

---

## Pages

| 경로           | 설명                |
| -------------- | ------------------- |
| `/`            | 랜딩 페이지         |
| `/analyze`     | 분석 요청           |
| `/report/[id]` | 결과 (SSE 스트리밍) |
| `/history`     | 히스토리 (모달)     |

---

## API Layer Architecture

```
┌─────────────────────────────────────────────────┐
│              Component Layer                     │
│  useSuspenseQuery → AsyncBoundary 감싸기         │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│              Query Hook Layer                    │
│  useAnalysis, useReport, useHistory             │
│  - useMyQuery.getQueryKeys() 메서드 포함         │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│              API Function Layer                  │
│  analysisApi.create(), reportApi.get()          │
│  - ky 인스턴스 사용                              │
│  - 커스텀 에러 코드 파싱                          │
└─────────────────┬───────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────┐
│              Route Config Layer                  │
│  /lib/api/routes.ts                             │
│  API_ROUTES.ANALYZE, API_ROUTES.REPORT          │
└─────────────────────────────────────────────────┘
```

---

## React Query 패턴

```typescript
// 쿼리 훅 생성 패턴 (Object.assign)
const getAnalysisKey = (id: string) => ["analysis", id];

export const useAnalysis = Object.assign(
  (id: string) => {
    return useSuspenseQuery({
      queryKey: getAnalysisKey(id),
      queryFn: () => getAnalysis(id),
    });
  },
  {
    getQueryKey: getAnalysisKey,
  },
);

// 사용
const { data } = useAnalysis(id);
const queryKey = useAnalysis.getQueryKey(id);
```

---

## 상태 관리 (State Management)

1.  **URL as State**: `id`, `tab`, `filter` 등 공유 가능한 상태는 URL Query Parameter로 관리한다.
2.  **No Prop Drilling**: URL 파라미터나 React Query 캐시 데이터는 부모에서 자식으로 props로 전달하지 않고, **자식 컴포넌트에서 직접 hook을 호출**하여 가져온다.
    - 이를 통해 컴포넌트 결합도를 낮추고 독립성을 높인다.
    - 예: `AnalysisView`에서 `id`를 props로 받지 않고 `useSearchParams`로 직접 조회.

---

## 선언적 JSX (Declarative JSX)

1.  **If Component**: 조건부 렌더링에 삼항 연산자(`condition ? A : B`)나 논리 연산자(`&&`) 대신 `<If />` 컴포넌트를 사용한다.
    ```tsx
    <If
      condition={isLoading}
      ifTrue={<LoadingView />}
      ifFalse={<ContentView />}
    />
    ```
2.  **Switch/Case**: 복잡한 조건 분기는 IIFE와 switch 문을 사용하여 선언적으로 표현한다.

---

## 유틸리티 라이브러리

- **es-toolkit**: 성능과 번들 사이즈 최적화를 위해 `lodash` 대신 `es-toolkit`을 사용한다.
- `isEmpty`, `get` 등 일반적인 유틸리티 함수는 `es-toolkit/compat` 또는 `es-toolkit`에서 가져와 사용한다.

---

## AsyncBoundary

Suspense + ErrorBoundary 통합 컴포넌트

```tsx
<AsyncBoundary
  fallback={<Skeleton />}
  errorFallback={({ error, reset }) => (
    <ErrorUI error={error} onRetry={reset} />
  )}
>
  <DataComponent />
</AsyncBoundary>
```

- **useSuspenseQuery 우선** 사용
- 적합하지 않은 경우에만 useQuery 사용

---

## 스켈레톤 UI

API 응답 대기 중 표시하는 로딩 상태

- 각 페이지/컴포넌트별 맞춤 스켈레톤
- 실제 콘텐츠와 동일한 레이아웃

---

## 접근성

- **스킬 참조**: `.agent/skills/accessibility/SKILL.md`
- WCAG 2.1 AA 준수
- 키보드 네비게이션
- 스크린 리더 지원

---

## Constraints

- 순수 유틸/훅 작성
- 사이드이펙트 필요 시 전용 훅으로 분리
- Shadcn UI 컴포넌트 우선 사용
- **State Management**: Props/State 끌어올리기(Hoisting)를 지양한다. React Query 캐시 또는 Path/Query Parameter에서 필요한 데이터를 하위 컴포넌트가 직접 가져와 사용(Colocation)한다.
- **Declarative JSX**: `renderFunction`이나 `Map` 객체를 사용한 렌더링을 지양한다. 조건부 로직이 복잡할 경우 `ts-pattern`이나 IIFE(즉시 실행 함수)를 사용하여 JSX 내에서 흐름을 명확히 한다.
- **Client Component 원칙**: React Hook(예: `useState`, `useEffect`, `useSearchParams`), 이벤트 핸들러, 또는 브라우저 전용 API(`window`, `document`)를 사용하는 컴포넌트/유틸리티는 반드시 파일 최상단에 `"use client"`를 명시해야 한다. 가능한 한 Leaf Node(말단 컴포넌트)로 Client Boundary를 미루어 Server Component의 이점을 살린다.

---

## 비동기 Fallback

모든 비동기 작업에 대해 fallback UI 구성

```tsx
// 페이지 레벨
<AsyncBoundary fallback={<PageSkeleton />}>
  <Page />
</AsyncBoundary>

// 컴포넌트 레벨
<AsyncBoundary fallback={<CardSkeleton />}>
  <DataCard />
</AsyncBoundary>

// 버튼 로딩
<Button disabled={isPending}>
  {isPending ? <Spinner /> : '분석 시작'}
</Button>
```

---

## 애니메이션

부드러운 UX를 위한 전역 애니메이션 적용

| 요소        | 애니메이션             |
| ----------- | ---------------------- |
| 모달        | fadeIn/fadeOut + scale |
| 페이지 전환 | slide 또는 fade        |
| 확인창      | fadeIn + slideUp       |
| 드롭다운    | slideDown              |
| 스켈레톤    | pulse                  |

```tsx
// Framer Motion 또는 CSS transition
<motion.div
  initial={{ opacity: 0, scale: 0.95 }}
  animate={{ opacity: 1, scale: 1 }}
  exit={{ opacity: 0, scale: 0.95 }}
>
  <ModalContent />
</motion.div>
```

---

## 반응형 디자인

모바일 우선 (Mobile First) 접근

| 브레이크포인트 | 범위   |
| -------------- | ------ |
| sm             | 640px  |
| md             | 768px  |
| lg             | 1024px |
| xl             | 1280px |

```tsx
// Tailwind 반응형 클래스
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {items.map(item => <Card key={item.id} />)}
</div>

// 모바일 네비게이션
<nav className="hidden md:flex">Desktop Nav</nav>
<nav className="md:hidden">Mobile Nav</nav>
```
