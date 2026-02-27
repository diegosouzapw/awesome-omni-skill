---
name: setup-claude-code
description: Claude Code 프로젝트 설정 생성
user-invocable: true
---

# Claude Code 프로젝트 설정 생성

**어떤 프로젝트, 어떤 언어에서든** Claude Code 설정을 자동으로 생성합니다.

지원 언어: TypeScript, JavaScript, Python, Go, Rust, Java, C#, Ruby, PHP 등

## 사용법

```
/setup-claude-code [옵션]
```

옵션:
- `minimal`: 최소 설정 (CLAUDE.md + 훅)
- `standard`: 표준 설정 (+ Commands, Agents)
- `full`: 전체 설정 (+ Skills, MCP, GitHub Action)
- 생략 시: 프로젝트 분석 후 자동 결정

## 왜 이 커맨드가 필요한가?

```
문제: Claude Code 설정이 없거나 부족한 프로젝트에서 처음부터 수동 설정은 번거로움
해결: 프로젝트 분석 → 언어/구조에 맞는 초기 설정 자동 생성
결과: CLAUDE.md, .claude/, hooks, permissions 등 한 번에 설정
```

## 수행 작업

### 1단계: 마스터 가이드 참조

**반드시** 다음 문서를 먼저 읽습니다:

```
.claude/docs/CLAUDE-CODE-MASTERY.md           # 목차 및 핵심 개념
.claude/docs/mastery/01-settings-guide.md     # 설정 요소별 상세 가이드
.claude/docs/mastery/02-language-templates.md # 언어별 설정 템플릿
.claude/docs/mastery/03-project-structures.md # 프로젝트 구조별 가이드
```

필요한 내용에 따라 해당 파일을 참조합니다.

### 2단계: 프로젝트 분석

```bash
# 분석 대상
1. package.json → 패키지 매니저, 스크립트 확인
2. 폴더 구조 → 모노레포 여부, 모듈 구조
3. 기존 .claude/ → 이미 설정된 항목 확인
4. tsconfig.json → TypeScript 설정
5. .eslintrc / biome.json → 린트 설정
6. .prettierrc → 포맷터 설정
```

### 3단계: 기술 스택 파악

| 항목 | 확인 방법 |
|------|----------|
| 패키지 매니저 | lockfile 확인 (pnpm-lock.yaml, package-lock.json, bun.lockb) |
| 언어 | tsconfig.json 유무, 파일 확장자 |
| 프레임워크 | package.json dependencies |
| 모노레포 | pnpm-workspace.yaml, turbo.json, lerna.json |
| 테스트 도구 | vitest, jest, mocha 등 |

### 4단계: 설정 규모 결정

```markdown
## 규모 결정 기준

### Minimal (소규모, 1-3명)
- 단일 패키지
- 간단한 프로젝트

### Standard (중규모, 4-10명)
- 모노레포 또는 복잡한 단일 패키지
- 여러 개발자 협업

### Full (대규모, 10명+)
- 대규모 모노레포
- 여러 도메인
- 외부 시스템 연동 필요
```

### 5단계: 파일 생성

#### 5.1 CLAUDE.md 생성 (항상)

```markdown
# Development Workflow

## 패키지 관리
- **항상 `{detected_package_manager}` 사용**

## 개발 순서
1. 변경 사항 작성
2. 타입체크: `{typecheck_command}`
3. 테스트: `{test_command}`
4. 린트: `{lint_command}`
5. 빌드: `{build_command}`

## 코딩 컨벤션
- `type` 선호, `interface` 자제
- **`enum` 절대 금지** → 문자열 리터럴 유니온 사용
- Zod 스키마로 타입 정의

## 프로젝트 구조
{detected_structure}

## 금지 사항
- ❌ console.log 대신 logger 사용
- ❌ any 타입 사용 금지
```

#### 5.2 settings.local.json 생성 (항상)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "{format_command} || true"
          }
        ]
      }
    ]
  },
  "permissions": {
    "allow": [
      "Bash({package_manager} run:*)",
      "Bash({package_manager} build:*)",
      "Bash({package_manager} test:*)",
      "Bash({package_manager} lint:*)",
      "Bash({package_manager} typecheck:*)"
    ],
    "deny": [],
    "ask": []
  }
}
```

#### 5.3 Commands 생성 (Standard, Full)

`.claude/commands/` 폴더에 다음 파일 생성:

1. `commit-push-pr.md` - 커밋 → 푸시 → PR
2. `typecheck-all.md` - 전체 타입체크
3. `test-module.md` - 모듈별 테스트
4. `lint-fix.md` - 린트 자동 수정
5. `build-all.md` - 전체 빌드

#### 5.4 Agents 생성 (Standard, Full)

`.claude/agents/` 폴더에 다음 파일 생성:

1. `build-validator.md` - 빌드 검증
2. `code-reviewer.md` - 코드 리뷰
3. `test-runner.md` - 테스트 실행 분석
4. `api-doc-generator.md` - API 문서 생성

#### 5.5 Skills 생성 (Standard, Full) - 하이브리드 방식

`.claude/skills/` 폴더에 **3단계**로 스킬을 생성합니다.

##### 1단계: 공통 스킬 (항상 생성)

| 스킬 | 용도 |
|------|------|
| `{project}-architecture` | 전체 아키텍처, 폴더 구조, 의존성 방향 |
| `{project}-testing` | 테스트 패턴, 단위/통합 테스트, 실행 방법 |

##### 2단계: 프로젝트 유형 감지

```bash
# 프로젝트 구조 분석
if modules/ or domains/ exist → Hexagonal/DDD
if packages/ or apps/ exist → Monorepo
if components/ or pages/ exist → Frontend
if controllers/ or routes/ exist → MVC Backend
if services/ exist → Microservices
if src/ only → Library/Simple
```

##### 3단계: 유형별 스킬 제안 (사용자 확인)

**Hexagonal/DDD 프로젝트**:

| 감지 | 제안 스킬 |
|------|----------|
| `modules/{name}/` | `{project}-{name}` (도메인별) |
| MongoDB 사용 | `{project}-database` |
| Fastify/Express | `{project}-api-conventions` |
| Zod 사용 | `{project}-validation` |

**모노레포**:

| 감지 | 제안 스킬 |
|------|----------|
| `packages/{name}/` | `{project}-{name}` (패키지별) |
| `apps/{name}/` | `{project}-{name}-app` |
| 공통 설정 | `{project}-shared` |

**프론트엔드**:

| 감지 | 제안 스킬 |
|------|----------|
| `components/` | `{project}-components` |
| `pages/` or `routes/` | `{project}-routing` |
| `hooks/` | `{project}-hooks` |
| `stores/` or `context/` | `{project}-state` |

**MVC 백엔드**:

| 감지 | 제안 스킬 |
|------|----------|
| `controllers/` | `{project}-controllers` |
| `models/` | `{project}-models` |
| `routes/` | `{project}-api-conventions` |
| `middleware/` | `{project}-middleware` |

**마이크로서비스**:

| 감지 | 제안 스킬 |
|------|----------|
| `services/{name}/` | `{project}-{name}` (서비스별) |
| Docker 사용 | `{project}-deployment` |
| 메시지 큐 | `{project}-messaging` |

**라이브러리**:

| 감지 | 제안 스킬 |
|------|----------|
| `src/` | `{project}-api` |
| `examples/` | `{project}-examples` |

##### 사용자 확인 (AskUserQuestion)

```
프로젝트 유형: Hexagonal/DDD (modules/ 감지)

다음 스킬을 생성할까요?

공통 스킬 (자동 생성):
 ✅ {project}-architecture
 ✅ {project}-testing

프로젝트별 스킬 (선택):
 [x] {project}-auth (modules/auth/)
 [x] {project}-user (modules/user/)
 [x] {project}-transaction (modules/transaction/)
 [ ] {project}-marketing (선택 해제 가능)
 [x] {project}-database (MongoDB 감지)
 [x] {project}-api-conventions (Fastify 감지)
```

##### 스킬 템플릿

```yaml
---
name: {project}-{name}
description: {name} 관련 작업 시 적용. Use when working with {keywords}.
---

# {Name} Skill

## 개요
[자동 분석된 설명]

## 핵심 파일
[해당 영역의 주요 파일 경로]

## 핵심 규칙
[코드베이스에서 추출한 패턴]

## 자주 사용하는 명령어
[관련 테스트/빌드 명령어]
```

#### 5.6 MCP 서버 연결 (Standard, Full) - 선택적

**중요**: MCP 서버는 보안상 사용자가 명시적으로 선택해야 합니다.

> "Use third party MCP servers at your own risk" - Anthropic

##### 의존성 분석 후 추천

프로젝트 의존성(package.json, requirements.txt 등)을 분석하여 관련 MCP 서버를 추천합니다.

| 감지된 의존성 | 추천 MCP 서버 |
|--------------|--------------|
| mongoose, mongodb | MongoDB MCP |
| pg, postgres | PostgreSQL MCP |
| stripe | Stripe MCP |
| @sentry/* | Sentry MCP |
| redis, ioredis | Redis MCP |

**상세 추천 전략**: `.claude/docs/mastery/01-settings-guide.md` 참조

##### 사용자 선택 (AskUserQuestion)

MCP 서버 연결 여부를 사용자에게 질문합니다:

```
프로젝트 분석 결과, 다음 MCP 서버를 추천합니다:

협업 도구 (선택):
[ ] GitHub - PR/이슈 관리
[ ] Linear - 이슈 트래킹
[ ] Jira - 프로젝트 관리

의존성 감지 (선택):
[x] MongoDB (mongoose 감지됨)
[ ] Redis (ioredis 감지됨)

기타:
[ ] 건너뛰기 - MCP 연결 안 함
```

##### .mcp.json 생성 (선택한 서버만)

사용자가 선택한 서버만 포함합니다:

```json
{
  "mcpServers": {
    "mongodb": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-mongodb"],
      "env": {
        "MONGODB_URI": "${MONGODB_URI}"
      }
    }
  }
}
```

##### MCP 서버 CLI 명령어

```bash
# CLI로 직접 추가
claude mcp add github

# 상태 확인
claude
/mcp
```

#### 5.7 GitHub Action 생성 (Full만)

`.github/workflows/claude-docs-update.yml` 생성

### 6단계: 기존 설정과 병합

- 이미 존재하는 파일은 덮어쓰지 않음
- 새로운 항목만 추가하거나 사용자에게 확인 요청

## 결과 출력

```
✅ Claude Code 설정 완료!

📁 생성된 파일:
- CLAUDE.md ✅
- .claude/settings.local.json ✅
- .claude/commands/ (5개) ✅
- .claude/agents/ (4개) ✅
- .claude/skills/ (4개) ✅
- .mcp.json (선택 시)
- .github/workflows/claude-docs-update.yml ✅

📊 설정 규모: {minimal|standard|full}
🔧 기술 스택: {detected_stack}

🎯 다음 단계:
1. CLAUDE.md 내용 확인 및 수정
2. /learn-claude-code 로 사용법 학습
3. /upgrade-claude-code 로 최신 트렌드 확인
```

## 주의사항

- 기존 파일이 있으면 백업 후 병합을 시도합니다
- package.json이 없으면 기본 npm 설정으로 생성합니다
- 커스텀 설정이 필요하면 생성 후 수동으로 수정하세요

## 에러 처리

### 마스터 가이드를 찾을 수 없는 경우

```
⚠️ 마스터 가이드 문서를 찾을 수 없습니다.

해결 방법:
1. bkit-starter 저장소에서 .claude/docs/ 폴더를 복사하세요
2. 최소 설정으로 진행하려면: /setup-claude-code minimal
```

### 프로젝트 유형 감지 실패 시

```
⚠️ 프로젝트 유형을 감지할 수 없습니다.

확인할 사항:
- package.json, tsconfig.json 등 설정 파일이 있는지 확인
- 프로젝트 루트 디렉토리에서 실행했는지 확인

수동 지정하려면:
→ /setup-claude-code minimal (최소 설정)
→ /setup-claude-code standard (표준 설정)
→ /setup-claude-code full (전체 설정)
```

### 파일 생성 실패 시

```
⚠️ 파일 생성에 실패했습니다: {파일명}

확인할 사항:
- 쓰기 권한이 있는지 확인
- 디스크 공간이 충분한지 확인

수동 생성하려면:
→ 01-settings-guide.md의 해당 섹션 참조하여 직접 생성
```

## 참고 문서

- .claude/docs/CLAUDE-CODE-MASTERY.md
- .claude/docs/mastery/01-settings-guide.md (MCP 추천 전략 포함)
