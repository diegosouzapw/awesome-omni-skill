---
name: fullstack-mirror-arch
description: 풀스택 미러 아키텍처 규칙. BE↔FE 1:1 타입 동기화, 레이어 의존 규칙, barrel re-export, API 클라이언트 패턴, 상태관리 분리 규칙을 적용. 풀스택 프로젝트 설계 시 사용.
---

# Fullstack Mirror Architecture (풀스택 미러 아키텍처)

백엔드와 프론트엔드 간 **1:1 미러 구조**를 유지하기 위한 규칙과 패턴.
기술 스택과 무관한 **개념 규칙**을 정의하고, 구현 예시는 참고용으로 표기한다.

---

## 1. 미러 구조 원칙

### 규칙
- 도메인별 **모델/타입 파일을 1:1 대응**시킨다
- BE 모델 변경 시 FE 타입을 **반드시 동시 수정**한다
- 새 도메인 추가 시 양쪽 barrel re-export를 함께 갱신한다
- 도메인 이름(파일명)은 BE/FE 양쪽에서 **동일하게** 유지한다

### 구조 매핑

```
BE: models/{domain}      ↔  FE: types/{domain}        (타입/모델 정의)
BE: api/{domain}/router   ↔  FE: lib/api/{domain}      (API 엔드포인트)
BE: services/{domain}     ↔  (FE에서 직접 대응 없음)   (비즈니스 로직)
```

### 예시 (참고)
| BE (FastAPI/Pydantic) | FE (Next.js/TypeScript) |
|---|---|
| `models/session.py` | `types/session.ts` |
| `api/session/router.py` | `lib/api/session.ts` |
| `models/schemas.py` (barrel) | `types/index.ts` (barrel) |

> Django: `serializers/{domain}.py`, Express: `models/{domain}.ts`, Spring: `dto/{domain}.java` 등으로 대체 가능

---

## 2. Barrel Re-export 패턴

### 규칙
- BE와 FE 각각 **하나의 barrel 파일**에서 모든 도메인 모델/타입을 re-export
- 새 도메인 추가 시 barrel 파일에 **반드시** 추가
- 새 코드에서는 도메인 모듈 직접 import 권장, barrel은 하위 호환용 유지

### 구조

```
BE barrel:
  models/schemas.py     → 모든 도메인 모델 re-export + __all__

FE barrel:
  types/index.ts        → 모든 도메인 타입 re-export
  lib/api/endpoints.ts  → 모든 API 클라이언트 re-export
```

### 예시 (참고)

```typescript
// types/index.ts
export * from './common';
export * from './session';
export * from './upload';
export * from './labeling';
```

```python
# models/schemas.py
from .common import *
from .session import *
from .upload import *
from .labeling import *
```

---

## 3. 자동 케이스 변환 패턴

### 규칙
- BE의 네이밍 컨벤션(snake_case)과 FE의 네이밍 컨벤션(camelCase)을 **자동 변환**한다
- DB 컬럼명(snake_case) → API 응답(camelCase) → FE 타입(camelCase)
- 수동 변환은 금지 — 반드시 프레임워크/라이브러리의 자동 변환 기능 사용

### 예시 (참고)

| 스택 | 구현 방법 |
|------|-----------|
| Pydantic (Python) | `CamelModel` 기반 클래스 + `alias_generator=to_camel` |
| Django REST | `CamelCaseJSONRenderer` / `djangorestframework-camel-case` |
| Spring (Java) | `@JsonNaming(PropertyNamingStrategies.LowerCamelCaseStrategy.class)` |
| Express (Node) | 미들웨어에서 `camelCase ↔ snake_case` 변환 |
| Go | `json:"fieldName"` 태그 |

---

## 4. Backend 레이어 규칙

### 의존 방향 (바깥 → 안쪽)

```
api/ (라우터/컨트롤러)
  ↓ 의존
services/ (비즈니스 로직)
  ↓ 의존
models/ (도메인 모델/스키마)
  ↓ 의존
core/ (설정, DB 클라이언트, 인증)
```

### 규칙
- **api/**: HTTP 요청 수신, 응답 반환만 담당. 비즈니스 로직 금지
- **services/**: 비즈니스 로직 캡슐화. DB 접근은 여기서 수행
- **models/**: 순수 데이터 정의. 비즈니스 로직/DB 접근 금지
- **core/**: 설정(config), DB 클라이언트, 인증 헬퍼. 싱글턴 패턴
- **역방향 의존 금지**: models/가 api/를 import하면 안 됨

### 라우터 등록
- `main` 파일에서 라우터 등록 순서를 **명시적으로 관리**
- 새 도메인 라우터 추가 시 등록 순서 문서화

### 예시 (참고)

| 스택 | api/ | services/ | models/ | core/ |
|------|------|-----------|---------|-------|
| FastAPI | `APIRouter` | 서비스 클래스/함수 | Pydantic Model | `BaseSettings` |
| Express | `Router` | 서비스 모듈 | Mongoose/Prisma | `config.ts` |
| Django | `views.py` | `services.py` | `models.py` | `settings.py` |
| Spring | `@Controller` | `@Service` | `@Entity`/DTO | `@Configuration` |

---

## 5. Frontend 레이어 규칙

### 의존 방향 (안쪽 → 바깥쪽)

```
types/       (순수 타입, 의존 없음)
  ↑ 의존
lib/         (API, 스토어, 훅, 유틸)
  ↑ 의존
components/  (UI 컴포넌트)
  ↑ 의존
app/         (라우트/페이지 엔트리)
```

### 규칙
- **types/**: 순수 타입 정의만. 런타임 코드/import 없음
- **lib/**: API 클라이언트, 상태 스토어, 커스텀 훅, 유틸. **app/이나 components/를 절대 import 금지**
- **components/**: UI 렌더링. lib/, types/ 의존 가능
  - `ui/`: 도메인 무관 공용 프리미티브 (Button, Modal, Toast 등)
  - `domain/`: 도메인 특화 재사용 뷰
  - `layout/`: 앱 셸 (사이드바, 탑바, 네비게이션)
  - `providers/`: 컨텍스트 래퍼 (Auth, Theme 등)
- **app/**: 라우트 엔트리. 모든 레이어 의존 가능

### Import 규칙
- 절대경로 alias 사용: `@/lib/api/session` (상대경로 지양)
- 순환 의존 금지

---

## 6. API 클라이언트 패턴

### 규칙
- **인증 래퍼 함수**로 토큰 주입을 자동화한다 (매 요청마다 수동 토큰 추가 금지)
- 도메인별로 API 클라이언트 파일을 분리한다
- API base URL은 **환경변수**로 관리한다
- 모든 API 에러는 catch 후 **의미 있는 메시지로 throw**한다

### 구조

```
lib/api/
├── auth-fetch.ts (또는 http-client.ts)   ← 인증 래퍼
├── {domain-1}.ts                          ← 도메인별 API
├── {domain-2}.ts
└── endpoints.ts                           ← barrel re-export
```

### 예시 (참고)

```typescript
// lib/api/auth-fetch.ts (개념)
export async function authFetch(url: string, options?: RequestInit) {
  const token = await getAccessToken(); // 프레임워크별 방법
  return fetch(url, {
    ...options,
    headers: { ...options?.headers, Authorization: `Bearer ${token}` },
  });
}
```

> axios interceptor, ky hooks, ofetch onRequest 등으로도 구현 가능

---

## 7. 상태관리 패턴

### 규칙
- **도메인별 독립 스토어**로 분리한다 (하나의 거대 스토어 금지)
- 스토어에 **UI 컴포넌트 import 금지**
- 스토어에 **fetch/navigation side effect를 직접 넣지 않기** (훅이나 미들웨어에서 처리)
- 스토어 간 결합은 **명확한 인터페이스**를 통해서만
- Toast/Modal 같은 UI 상태는 **전용 UI 스토어**에서 관리 (인라인 생성 금지)

### 스토어 분류 예시

| 스토어 | 역할 |
|--------|------|
| `ui-store` | 사이드바, 모달, 토스트, 로딩 상태 |
| `auth-store` | 현재 사용자, 인증 상태 |
| `{domain}-store` | 도메인별 데이터 및 상태 |

### 예시 (참고)

| 스택 | 구현 방법 |
|------|-----------|
| Zustand | `create<State>()` 패턴, 도메인별 파일 분리 |
| Redux Toolkit | `createSlice()`, 도메인별 slice 분리 |
| Pinia (Vue) | `defineStore()`, 도메인별 store 분리 |
| Svelte Store | `writable()`, 도메인별 파일 분리 |

---

## 8. BE↔FE 동기화 체크리스트

코드 변경 시 아래 항목을 확인한다:

- [ ] BE 모델 필드 추가/변경/삭제 → **FE 타입 동시 변경**
- [ ] 새 도메인 추가 → **양쪽 barrel re-export 갱신**
- [ ] 새 API 엔드포인트 → **FE API 클라이언트 + endpoints barrel 갱신**
- [ ] enum/상수 값 변경 → **BE/FE 양쪽 반영**
- [ ] 필드명 변경 → **케이스 변환 규칙 확인** (snake↔camel)
- [ ] 응답 구조 변경 → **FE에서 사용하는 모든 컴포넌트 확인**
- [ ] 에러 코드 추가 → **FE 에러 핸들링에 반영**
