---
name: code-quality
description: 코드 품질 규칙을 강제합니다. 500줄 제한과 주석 필수 규칙을 적용합니다.
user-invocable: false
allowed-tools: Read, Grep, Glob, Edit, Write
agent: calab-plugin:code-reviewer
agents:
  primary: calab-plugin:code-reviewer
  orchestration:
    write: [calab-plugin:code-reviewer]
    validate: [calab-plugin:validator]
    fix: [calab-plugin:reinforcer]
    review: [calab-plugin:code-reviewer, calab-plugin:validator]
---

# Code Quality Skill

## 목적

코드 품질 규칙(줄 수 제한, 주석 필수)을 **강제로** 적용하여 일관된 코드 품질을 유지합니다.

## 핵심 규칙 (절대 무시 금지)

### 규칙 0: 할루시네이션 방지 (Anthropic 공식)

> 출처: docs.anthropic.com - 코드 작업 전 필수 확인

```
⚠️ 코드 수정 전 반드시:
1. 해당 파일 먼저 읽기 (추측 금지)
2. 코드베이스 스타일/컨벤션 파악
3. 확실하지 않으면 "확인 필요" 인정
```

**절대 금지:**
- 열어보지 않은 파일 내용 추측
- 확인 없이 코드 구조 주장
- 근거 없는 수정 제안

### 규칙 1: 500줄 제한

```
모든 소스 파일은 반드시 500줄 이하로 작성
```

**500줄 초과가 예상될 때:**
1. 먼저 파일을 분리할 계획을 세움
2. 사용자에게 분리 방안 제안
3. 승인 후 분리하여 작성

### 규칙 2: 주석 필수

```
모든 함수/메서드에 JSDoc 또는 Docstring 필수
복잡한 로직(조건문 3개 이상, 루프 2중 이상)에 설명 주석 필수
```

## 코드 생성 프로토콜

### 1. 사전 계획 (코드 작성 전)

```
단계:
1. CODE_STYLE.md 규칙 확인
2. 예상 줄 수 계산
3. 500줄 초과 예상 시 → 파일 분리 계획 수립
4. 함수 목록 작성 → 각 함수에 주석 계획
```

### 2. 코드 작성 중

```
규칙:
1. 함수 시작 전 → JSDoc/Docstring 먼저 작성
2. 복잡한 로직 전 → 설명 주석 작성
3. 100줄마다 → 현재 줄 수 체크
4. 200줄 도달 시 → 남은 코드 분리 검토
```

### 3. 코드 작성 후 (자체 검증)

```
체크리스트:
1. 총 줄 수 확인 (500줄 이하)
2. 모든 함수에 주석 존재
3. 복잡 로직에 설명 주석 존재
4. 위반 시 즉시 수정
```

## 출력 형식

### 500줄 초과 예상 시

```
============================================
[CODE QUALITY] 파일 분리 필요
============================================

예상 줄 수: ~450줄
최대 허용: 500줄

분리 제안:
1. user-service.ts (인증 로직) - ~150줄
2. user-repository.ts (DB 접근) - ~120줄
3. user-types.ts (타입 정의) - ~80줄
4. user-utils.ts (유틸리티) - ~100줄

============================================
이렇게 분리해서 작성할까요?
```

### 주석 작성 예시

**TypeScript:**
```typescript
/**
 * 사용자 ID로 사용자 정보를 조회합니다.
 *
 * @param userId - 조회할 사용자의 고유 ID
 * @returns 사용자 객체 또는 존재하지 않으면 null
 * @throws DatabaseError 데이터베이스 연결 실패 시
 */
async function getUserById(userId: string): Promise<User | null> {
```

**Python:**
```python
async def get_user_by_id(user_id: str) -> Optional[User]:
    """
    사용자 ID로 사용자 정보를 조회합니다.

    Args:
        user_id: 조회할 사용자의 고유 ID

    Returns:
        User 객체 또는 존재하지 않으면 None

    Raises:
        DatabaseError: 데이터베이스 연결 실패 시
    """
```

## 참조 파일

- `.claude/memory/CODE_STYLE.md` - 상세 코드 스타일 규칙
- `.claude/memory/PROJECT_RULES.md` - 프로젝트 규칙
