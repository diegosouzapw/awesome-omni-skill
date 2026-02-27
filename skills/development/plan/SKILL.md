---
name: plan
description: 아키텍처 설계 단계. specify 체크포인트를 참조하여 기술적 설계를 수행하고 plan 체크포인트를 생성합니다.
---

# Plan Skill

기능 명세를 바탕으로 아키텍처와 기술적 설계를 수행합니다.

## When to Use

- `/plan` 명령 시
- specify 완료 후 자동 전환 시
- "설계", "아키텍처", "구조" 키워드 시

## Prerequisites

- `docs/.checkpoints/{feature}-specify.md` 존재 (status: ready)
- 없으면 `/specify` 먼저 실행 안내

## Process

```mermaid
flowchart TD
    A[Specify 체크포인트 로드] --> B[Plan 체크포인트 생성]
    B --> C[Clarify 호출]
    C --> D{모호함 해소?}
    D -->|No| C
    D -->|Yes| E[기술 결정 정리]
    E --> F[체크포인트 Ready]
    F --> G[완료 메시지]
    G --> H{다음 단계?}
    H -->|yes| I[/task 실행]
    H -->|no| J[대기]
```

## Step 1: Load Specify Checkpoint

```
docs/.checkpoints/{feature}-specify.md 로드

- Feature Overview 확인
- Functional Requirements 확인
- Acceptance Criteria 확인
```

## Step 2: Create Plan Checkpoint

`docs/.checkpoints/{feature}-plan.md` 생성

```markdown
# auth-login - Plan Checkpoint

## Metadata
- feature: auth-login
- stage: plan
- status: in_progress
- created: 2026-01-29
- depends_on: 
  - docs/.checkpoints/auth-login-specify.md

## Clarifications
### Session 2026-01-29

## Coverage Map
| Category | Status | Notes |
|----------|--------|-------|
| Functional Scope | Clear | (from specify) |
| Domain & Data Model | Missing | |
| Constraints | Missing | |
...
```

## Step 3: Run Clarify

clarify skill을 호출하여 설계 관련 모호함을 해소합니다.

**Clarify 포커스 카테고리** (plan 단계):
- Domain & Data Model (필수)
- Non-Functional (필수)
- Integration
- Constraints (필수)
- Misc

### 예상 질문들

```markdown
**기술 스택 선택**
**Recommended:** Option A - 프로젝트 요구사항에 가장 적합

| Option | Description |
|--------|-------------|
| A | React + TypeScript + Vite |
| B | Next.js + TypeScript |
| C | Vue + TypeScript |

---

**데이터베이스 선택**
**Recommended:** Option B - 관계형 데이터에 적합, 확장성 우수

| Option | Description |
|--------|-------------|
| A | MongoDB (NoSQL) |
| B | PostgreSQL (RDBMS) |
| C | MySQL (RDBMS) |

---

**인증 방식**
**Recommended:** Option A - stateless, 확장 용이

| Option | Description |
|--------|-------------|
| A | JWT + Refresh Token |
| B | Session-based |
| C | OAuth only |
```

## Step 4: Document Technical Decisions

clarify 완료 후 기술 결정을 정리합니다.

```markdown
## Content

### Architecture Overview
{전체 아키텍처 다이어그램 또는 설명}

### Technology Stack
| Layer | Technology | Reason |
|-------|-----------|--------|
| Frontend | React + TypeScript | 타입 안정성, 생태계 |
| Backend | Node.js + Express | 빠른 개발, JS 통합 |
| Database | PostgreSQL | ACID, 관계형 데이터 |
| Auth | JWT | Stateless, 확장성 |

### Component Design
- Component A: {역할}
- Component B: {역할}

### Data Model
{엔티티 관계 다이어그램 또는 설명}

### File Structure
```
src/
├── components/
├── hooks/
├── services/
├── types/
└── utils/
```

### Technical Decisions
| Decision | Options | Selected | Reason |
|----------|---------|----------|--------|
| Auth | JWT vs Session | JWT | Stateless |
| DB | Postgres vs Mongo | Postgres | ACID |
```

## Step 5: Completion Message

```markdown
## Plan 완료

**체크포인트**: `docs/.checkpoints/auth-login-plan.md`
**상태**: Ready

### Coverage Summary
| Category | Status |
|----------|--------|
| Domain & Data Model | Clear |
| Non-Functional | Clear |
| Constraints | Clear |
| ... | ... |

### Technical Decisions (4)
- Auth: JWT + Refresh Token
- Database: PostgreSQL
- Frontend: React + TypeScript
- API: REST

---

**다음 단계**: /task - 태스크 분리

| Option | Action |
|--------|--------|
| yes | /task 바로 실행 |
| no | 대기 (나중에 수동 실행) |

Reply: yes, no, or another command
```

## Step 6: Next Stage Transition

사용자 응답에 따라:
- `yes` → /task skill 자동 실행
- `no` → 대기
- 다른 명령 → 해당 명령 실행

## Output

- 생성: `docs/.checkpoints/{feature}-plan.md`
- 상태: Ready
- 다음 단계: /task 추천

## Error Handling

- specify 체크포인트 없음 → `/specify` 먼저 실행 안내
- specify status가 ready가 아님 → `/specify` 완료 필요 안내
- 기술 결정 충돌 → 추가 clarify 질문

## Integration

```
/specify (완료) 
    ↓
/plan (현재)
    ├── specify 체크포인트 참조
    ├── clarify 호출
    └── plan 체크포인트 생성
    ↓
/task (다음)
```
