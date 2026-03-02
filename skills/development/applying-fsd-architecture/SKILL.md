---
name: applying-fsd-architecture
description: Feature-Sliced Design(FSD) 아키텍처를 적용한 프론트엔드 프로젝트 개발 지원. FSD 레이어, 슬라이스, 세그먼트 구조 설계, 의존성 규칙 적용, 마이그레이션 시 사용.
---

# Feature-Sliced Design Architecture Skill

Feature-Sliced Design(FSD) 아키텍처를 적용한 프론트엔드 프로젝트 개발을 지원합니다.

## Description

이 스킬은 FSD 아키텍처에 대한 깊은 이해를 바탕으로 개발자가 올바른 구조를 설계하고 유지할 수 있도록 돕습니다. 레이어, 슬라이스, 세그먼트의 개념과 의존성 규칙을 적용하여 확장 가능하고 유지보수가 쉬운 코드베이스를 구축합니다.

## When to Use

이 스킬은 다음과 같은 상황에서 자동으로 활성화됩니다:

- FSD 아키텍처 관련 질문 (레이어, 슬라이스, 세그먼트)
- 프로젝트 폴더 구조 설계
- 의존성 규칙 및 import 방향 문의
- shared, entities, features, widgets, pages 등의 레이어 사용법
- FSD 마이그레이션 또는 리팩토링

## Trigger Patterns

- "FSD", "Feature-Sliced Design" 언급
- "레이어 구조", "슬라이스 생성", "세그먼트 추가"
- "shared 레이어", "entities 레이어", "features 레이어" 등
- "의존성 규칙", "import 방향"
- "프론트엔드 아키텍처", "폴더 구조 설계"

## Core Concepts

### 레이어 (Layers)

FSD는 7개의 표준화된 레이어를 정의합니다 (상위 → 하위):

1. **app** - 애플리케이션 진입점, 라우팅, 프로바이더
2. **processes** - (deprecated) 복잡한 페이지 간 시나리오
3. **pages** - 라우트 기준 화면 단위
4. **widgets** - 독립적인 대규모 UI 블록
5. **features** - 사용자 가치를 제공하는 기능 단위
6. **entities** - 비즈니스 엔티티 (User, Product 등)
7. **shared** - 재사용 가능한 유틸리티, UI 컴포넌트

### 의존성 규칙

```
app → pages → widgets → features → entities → shared
         ↓
   상위 레이어는 하위 레이어만 import 가능
   같은 레이어 내 슬라이스 간 import 금지
```

### 세그먼트 (Segments)

각 슬라이스 내부는 역할별로 구분됩니다:

- `ui/` - UI 컴포넌트, 스타일
- `api/` - API 요청 함수
- `model/` - 비즈니스 로직, 상태 관리
- `lib/` - 유틸리티 함수
- `config/` - 설정값

## References

- [layers.md](./references/layers.md) - 레이어별 상세 가이드
- [slices.md](./references/slices.md) - 슬라이스 패턴 및 예시
- [segments.md](./references/segments.md) - 세그먼트 구조화
- [dependency-rules.md](./references/dependency-rules.md) - 의존성 규칙
- [migration-guide.md](./references/migration-guide.md) - 마이그레이션 전략

## External Documentation

최신 공식 문서는 WebFetch를 통해 참조합니다:

```
https://feature-sliced.design/kr/docs/get-started/overview
https://feature-sliced.design/kr/docs/reference/layers
https://feature-sliced.design/kr/docs/reference/slices-segments
```
