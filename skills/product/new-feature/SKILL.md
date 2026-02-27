---
name: new-feature
description: Use when starting new feature development - sets up Agent Teams, gathers requirements, creates plan, implements with parallel agents, runs code review/QA/security checks, and commits with conventional commits
---

새로운 기능 개발을 **Agent Teams 협업** 방식으로 진행합니다.
**단독 작업이 아닌 Agent Teams (TeamCreate + SendMessage) 협업으로 진행한다.**
아래 Phase를 순서대로 진행해줘.

⚠️ **핵심 원칙: Agent Teams 필수 (절대 위반 금지)**
- 단독 작업 금지 → 반드시 Agent Teams 방식 (TeamCreate + SendMessage)
- 모든 에이전트는 반드시 TeamCreate로 팀을 먼저 만들고, Task의 `team_name` 파라미터와 함께 생성 (팀원으로 등록)
- **절대로 team_name 없이 Task를 실행하지 않음** (sub-agent 모드에서는 에이전트 간 메시지 교환이 불가)
- 에이전트 간 협업은 반드시 SendMessage로 수행
- 작업 관리는 TaskCreate/TaskUpdate/TaskList로 팀 공유 Task List 활용

---

## Phase 0: 사전 조건 검증

아래 파일들이 모두 존재하는지 확인해줘.

### 필수 Agent 정의 (.claude/agents/)
- `cto-lead.md`
- `frontend-senior.md`
- `backend-senior.md`
- `uiux-senior.md`
- `security-senior.md`
- `qa-senior.md`

### 필수 Skills (.claude/skills/)
- `new-feature.md` (이 파일)

### 검증 결과 처리

누락된 파일이 있으면:
1. 누락 목록을 보여주고
2. AskUserQuestion: "누락된 파일이 있습니다. 어떻게 할까요?"
   - "중단" → 여기서 멈추고 누락 파일을 먼저 준비하라고 안내
   - "계속 진행" → 누락 파일 없이 가능한 범위에서 진행

모두 존재하면 → `.claude/templates/agents/`에 템플릿이 있는데 `.claude/agents/`에 없는 파일이 있으면 복사 후 "사전 조건 검증 완료" 표시

---

## Phase 1: 프로젝트 구조 감지 + 설정 생성

### 1-1: 디렉토리 자동 감지

현재 작업 디렉토리(CWD)의 하위 디렉토리를 스캔해줘:

| 파일 | 추정 type |
|------|-----------|
| `build.gradle` + Spring 의존성 | `spring-boot` |
| `pom.xml` + Spring 의존성 | `spring-boot` |
| `nuxt.config.ts` 또는 `nuxt.config.js` | `nuxt` |
| `next.config.*` | `nextjs` |
| `vite.config.*` | `vite` |
| `Cargo.toml` | `rust` |
| `go.mod` | `go` |
| `package.json`만 있음 | `node` |
| 기타 | `unknown` |

감지 결과를 테이블로 보여줌.

### 1-2: 사용자 확인

AskUserQuestion으로 각 디렉토리의 설정을 확인:
- 감지된 type이 있으면 추천으로 표시, "직접 입력" 옵션도 제공
- 작업 대상에서 제외할 디렉토리 선택 가능

### 1-3: project-config.json 생성

확인된 정보로 `.claude/project-config.json`을 생성/갱신:

```json
{
  "directories": [
    { "name": "backend", "path": "backend", "type": "spring-boot" },
    { "name": "frontend", "path": "frontend", "type": "nuxt" }
  ],
  "baseBranch": "master",
  "branchPrefix": "feature/"
}
```

### 1-4: 미커밋 변경사항 확인

대상 디렉토리들에서 각각 `git status --short` 실행.

변경사항이 있는 경우:
- 변경된 파일 목록을 보여주고
- AskUserQuestion으로 각 디렉토리별 처리:
  - "stash로 저장" → `git stash push -m "auto-stash: $(date +%Y%m%d-%H%M%S)"` 실행
  - "무시하고 진행" → 그대로 진행
  - "작업 중단" → 여기서 멈춤

변경사항이 없으면 "깨끗합니다" 표시 후 다음으로.

### 1-5: 기본 브랜치 선택

AskUserQuestion으로 base 브랜치 선택:
- "master (기본)"
- "현재 브랜치 유지" → 현재 브랜치명을 표시하고 그대로 사용
- "직접 입력" → 사용자가 브랜치명 입력

선택된 base 브랜치로:
1. `git checkout {base-branch}`
2. `git pull origin {base-branch}` (실패해도 진행)

이 base 브랜치명을 Phase 6에서 사용하므로 기억해둔다.

### 1-6: 기능 이름 수집 + 브랜치 생성

⚠️ **반드시 브랜치를 먼저 생성한 후 작업을 시작한다. master에서 직접 작업하지 않는다.**

1. AskUserQuestion으로 기능 이름을 수집:
   - 기능 이름 (영문 kebab-case 추천, 예: `user-profile-edit`)

2. AskUserQuestion으로 브랜치명 확인:
   - "feature/{feature-name} (추천)"
   - "직접 입력"

3. `git checkout -b {new-branch}` 실행

4. 브랜치 생성 확인:
   ```
   ✅ 브랜치 생성 완료: {new-branch}
   이후 모든 작업은 이 브랜치에서 진행됩니다.
   ```

---

## Phase 2: Agent Teams — 요구사항 수집 + Plan

### 2-1: 요구사항 수집 + 팀 생성

1. AskUserQuestion으로 기능의 상세 정보를 수집:
   - 상세 요구사항
   - 제약 조건 (기술적/비즈니스)
   (기능 이름은 Phase 1-6에서 이미 확정됨)

2. **TeamCreate**로 팀 생성:
   ```
   team_name: "{feature-name}"
   description: "{기능 설명 요약}"
   ```

3. 팀원을 **Task(team_name 필수)**로 생성:

   | name | subagent_type | 역할 |
   |------|---------------|------|
   | cto-lead | general-purpose | 전체 아키텍처 + 오케스트레이션 |
   | frontend-senior | general-purpose | 프론트엔드 구현 |
   | backend-senior | general-purpose | 백엔드 구현 |
   | uiux-senior | general-purpose | 화면 설계 (필요 시) |
   | security-senior | general-purpose | 보안 검증 |
   | qa-senior | general-purpose | 테스트/QA |

   각 에이전트 생성 시 prompt에:
   - `.claude/agents/{name}.md` 파일 내용을 포함
   - 프로젝트의 CLAUDE.md 규칙 준수 지시
   - 팀 내 다른 에이전트와 SendMessage로 소통하라는 지시

   ⚠️ **team_name 없이 Task 실행 = sub-agent = 에이전트 간 메시지 불가 = 절대 금지**

### 2-2: 화면 필요 여부 판단

CTO Lead가 요구사항을 분석하여:
- 화면이 필요한 기능 → UI/UX Senior에게 SendMessage로 화면 설계 요청
- 화면 불필요 → UI/UX Senior 건너뜀

### 2-3: 협업 Plan 작성

에이전트들이 SendMessage로 소통하며 Plan 초안 작성:
- CTO Lead → 전체 아키텍처 + 작업 분배 초안
- Backend Senior → 백엔드 구현 계획 (API 스펙 포함)
- Frontend Senior → 프론트 구현 계획 (Backend Senior의 API 스펙 참조)
- UI/UX Senior → 화면 설계안 (Phase 2-2에서 필요 판정 시)

### 2-4: Plan 통합 + 확정

CTO Lead가 각 초안을 취합하여 최종 Plan 확정:
→ `docs/plan/features/{feature}.plan.md` 생성
→ feature 이름 최종 확정

---

## Phase 3: Plan 커밋 + 병렬 구현

### 3-1: Plan 문서 첫 커밋

(브랜치는 Phase 1-6에서 이미 생성됨)

```bash
git add docs/plan/features/{feature}.plan.md
git commit -m "docs: {feature} Plan 문서 작성

기능 요구사항 및 범위 정의.

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>"
```

### 3-3: 병렬 구현

Agent Teams 기반으로 구현 진행:
- CTO Lead가 TaskCreate로 작업 항목 생성 → 각 Senior에게 SendMessage로 할당
- Frontend Senior ↔ Backend Senior가 **SendMessage로 API 인터페이스 계약 조율**
- 각 Senior는 독립적으로 작업하되, 필요 시 SendMessage로 실시간 협업
- 구현 완료 시 각 Senior가 CTO Lead에게 SendMessage로 보고

---

## Phase 4: 병렬 검증

구현 완료 후 다음을 진행:

| 에이전트 | 작업 | 실패 시 |
|----------|------|---------|
| Frontend Senior | 프론트 코드리뷰 | **지적사항 기록** → 수정 → 수정내역 기록 |
| Backend Senior | 백엔드 코드리뷰 | **지적사항 기록** → 수정 → 수정내역 기록 |
| Security Senior | 보안 점검 (OWASP Top 10, 인증/인가) | 해당 Senior에게 SendMessage로 수정 요청 |
| QA Senior | 테스트 작성 + 실행 | 수정 → 재실행 (최대 3회) |
| CTO Lead | 요구사항 충족률 확인 | < 90%: 미충족 항목 수정 → 재분석 |

### Frontend 테스트 후 프로세스 정리 (필수)

⚠️ `npm run test:unit` (vitest) 실행 후 fork worker 프로세스가 남아있을 수 있다.
QA Senior는 frontend 테스트 실행 완료 후 **반드시** 잔여 프로세스를 정리해야 한다:

```bash
# vitest/node 잔여 프로세스 확인 및 종료
pkill -f "vitest" 2>/dev/null || true
# fork worker 프로세스 정리
pgrep -f "node.*vitest" | xargs kill 2>/dev/null || true
```

테스트 완료 후 `pgrep -f "vitest"` 로 잔여 프로세스가 없는지 확인한다.

### 코드리뷰 결과 추적 (필수)

각 Senior는 코드리뷰 결과를 CTO Lead에게 SendMessage로 아래 형식으로 보고:

```
## 코드리뷰 결과 — {Frontend/Backend}

### 지적사항
1. [파일:라인] 내용 — 심각도(Critical/Major/Minor)
2. ...

### 수정 내역
1. [파일:라인] 수정 내용 — 관련 지적사항 번호
2. ...

### 미수정 사유 (있을 경우)
1. [지적사항 번호] 사유
```

CTO Lead는 이 정보를 Phase 5 최종 요약에 포함한다.

---

## Phase 5: 작업 요약 + 커밋

### 5-1: 완료 보고

모든 검증 통과 후 아래 형식으로 보고:

```
작업 완료!
─────────────────────────────────────────────────
Feature: {feature-name}
Branch: feature/{feature-name}
─────────────────────────────────────────────────
대상 디렉토리:
  {name}: {type} — {path}
─────────────────────────────────────────────────
검증 결과:
  Code Review:  Frontend ✅ / Backend ✅
  Security:     보안 점검 통과
  QA:           N/N 시나리오 통과
  요구사항:     충족률 {N}%
─────────────────────────────────────────────────
코드리뷰 지적사항 요약:
  [Frontend]
    Critical (N건):
      - 요약...
    Major (N건):
      - 요약...
    Minor (N건):
      - 요약...
  [Backend]
    Critical (N건):
      - 요약...
    Major (N건):
      - 요약...
    Minor (N건):
      - 요약...
  수정 완료: N/N건
  미수정: N건 (사유 포함)
─────────────────────────────────────────────────
Stash: {stash 내역 또는 "없음"}
Plan: docs/plan/features/{feature}.plan.md
─────────────────────────────────────────────────
```

### 5-2: 커밋 (Conventional Commits 규칙)

아래 규칙을 **반드시** 준수하여 커밋:

#### 규칙
1. **하나의 커밋 = 하나의 논리적 변경** — 여러 성격의 변경을 하나에 섞지 않음
2. **Conventional Commits 형식** 사용:
   | prefix | 용도 |
   |--------|------|
   | `feat:` | 새 기능 |
   | `fix:` | 버그 수정 |
   | `refactor:` | 리팩토링 (동작 변경 없음) |
   | `chore:` | 빌드/설정 변경 |
   | `test:` | 테스트 추가/수정 |
   | `docs:` | 문서 변경 |
3. **커밋 메시지 본문에 "왜" 변경했는지** 반드시 포함
4. **리팩토링과 기능 변경을 절대 같은 커밋에 섞지 않기**
5. **커밋 단위 리뷰** — 변경사항을 분석하고 논리적 단위로 분리:
   - `git diff --stat`으로 변경 파일 확인
   - 성격이 다른 변경은 별도 커밋으로 분리
   - 사용자에게 커밋 분리 계획을 보여주고 확인 받음

#### 커밋 절차
1. `git status`와 `git diff --stat`으로 전체 변경사항 파악
2. 변경사항을 논리적 단위로 그룹핑
3. 사용자에게 커밋 분리 계획 표시:
   ```
   커밋 분리 계획:
   1. feat: 사용자 프로필 편집 API 추가 — backend/...
   2. feat: 프로필 편집 화면 구현 — frontend/...
   3. test: 프로필 편집 테스트 추가 — backend/..., frontend/...
   ```
4. 확인 후 각 커밋 실행 (각 커밋에 Co-Authored-By 포함)

#### 커밋 메시지 예시
```
feat: 사용자 프로필 편집 API 추가

프로필 편집 기능의 백엔드 구현.
PUT /api/users/profile 엔드포인트를 추가하여
이름, 이메일, 프로필 이미지 수정을 지원.

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

---

## Phase 6: 푸시 / 머지 결정

AskUserQuestion으로 다음 단계를 물어봄:
- "원격에 푸시" → `git push -u origin {feature-branch}`
- "로컬에서 {base-branch}로 머지" → `git checkout {base-branch} && git merge {feature-branch}`
  ({base-branch}는 Phase 1-5에서 선택한 기본 브랜치)
- "아무것도 하지 않음" → 현재 상태 유지

---

## Phase 7: 서버 종료 + 팀 해산

### 7-1: 서버 및 테스트 프로세스 종료

QA/테스트 중 실행했던 서버들과 잔여 프로세스를 종료:
- `lsof -ti :8080 | xargs kill -9 2>/dev/null || true` (backend)
- `lsof -ti :3000 | xargs kill -9 2>/dev/null || true` (frontend)
- `pkill -f "vitest" 2>/dev/null || true` (vitest 잔여 프로세스)
- `pgrep -f "node.*vitest" | xargs kill 2>/dev/null || true` (vitest fork worker)
- 기타 백그라운드 프로세스 정리

### 7-2: 팀 해산

1. 모든 팀원에게 SendMessage(type: "shutdown_request") 전송
2. 모든 팀원 shutdown 확인 후 TeamDelete로 팀 리소스 정리

---

## ⚠️ Agent Teams 필수 규칙 (이 섹션은 모든 Phase에 적용)

1. **팀 생성**: Phase 2에서 TeamCreate로 반드시 팀을 먼저 생성
2. **팀원 생성**: Task 도구 사용 시 `team_name` 파라미터를 **반드시** 포함
3. **에이전트 간 소통**: SendMessage 도구로만 수행 (sub-agent는 상호 소통 불가)
4. **작업 관리**: TaskCreate/TaskUpdate/TaskList로 팀 공유 Task List 활용
5. **CTO Lead가 오케스트레이터**: 모든 작업 할당과 결과 검증은 CTO Lead를 통해 수행
6. **sub-agent 금지**: team_name 없이 Task를 실행하면 에이전트 간 소통이 불가하므로 **절대 사용하지 않음**
