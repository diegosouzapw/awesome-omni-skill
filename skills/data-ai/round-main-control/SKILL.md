---
name: round-main-control
description: "트리거: Main 스레드에서 라운드 승인/보류 판단, 다음 handoff 1건 확정, Main->Executor 릴레이 생성이 필요할 때 사용. 비트리거: 코드 구현/테스트 실행이 필요한 실행 작업에는 사용하지 않는다."
---

# round-main-control

## 목적
- Main 스레드의 판단 품질을 고정한다.
- result/review/relay/TASK_BOARD/PROJECT_OVERVIEW/CURRENT_STATUS 간 불일치를 라운드마다 점검한다.
- 다음 라운드 handoff와 Main->Executor 릴레이를 누락 없이 1건 생성한다.

## 입력
- 대상 라운드 번호: `H-00N`
- 필수 문서:
  - `docs/PROJECT_OVERVIEW.md`
  - `coordination/TASK_BOARD.md`
  - `coordination/DECISIONS.md`
  - `coordination/REPORTS/CURRENT_STATUS_*.md`
  - `coordination/REPORTS/H-00N-result.md`
  - `coordination/REPORTS/H-00N-review.md`
  - `coordination/RELAYS/H-00N-review-to-main.md` (있으면 필수 반영)

## 수행 절차
1. 아래 체크리스트로 상태 불일치를 점검한다.
   - handoff 목표와 result 실제 변경이 일치하는가?
   - result 테스트 게이트와 review 판단이 충돌하지 않는가?
   - TASK_BOARD 현재 우선순위와 다음 handoff 제안이 정합적인가?
   - CURRENT_STATUS와 DECISIONS에 신규 정책 반영 누락이 없는가?
2. 현재 라운드 판단을 `승인/보류 + 근거` 형태로 작성한다.
   - 근거는 반드시 파일명 기준으로 남긴다.
3. 다음 handoff를 반드시 1건 생성/고정한다.
4. 문서 갱신 항목(`TASK_BOARD`, `DECISIONS`, `CURRENT_STATUS` 등)을 작성한다.
5. 다음 라운드 시작용 Main->Executor 릴레이 파일을 생성한다.
   - 경로: `coordination/RELAYS/H-00N-main-to-executor.md`
   - 템플릿: `coordination/RELAYS/TEMPLATE-main-to-executor.md`
6. Main 산출물 작성이 끝나면 운영 문서 변경분을 커밋/푸시한다.
   - 커밋 범위: `coordination/`, `docs/`, `.agents/`
   - 코드/설정 파일은 커밋 금지

## 금지 사항
- 코드 수정/코드 병합 금지
- 구현 세부 리뷰를 Main에서 대체 수행 금지(Review 근거 중심으로 판단)

## 표준 출력/파일 산출 규칙
- 텍스트 출력은 반드시 아래 3개만 작성한다.
  1. 현재 라운드 판단(승인/보류 + 근거 파일명)
  2. 다음 handoff 지시문 1건(여기에 Main->Executor 릴레이 생성 경로/핵심 내용 포함)
  3. 필요한 문서 갱신 항목
- 파일 산출:
  - `coordination/RELAYS/H-00N-main-to-executor.md`를 반드시 생성/갱신한다.
