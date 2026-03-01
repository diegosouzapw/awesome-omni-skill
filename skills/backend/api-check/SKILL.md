---
name: api-check
description: FastAPI 엔드포인트 상태 점검. async 패턴, Pydantic 모델, 캐싱, 에러 핸들링 검토
allowed-tools: Read, Grep, Glob, Bash
---

# API Health Check

## Instructions
1. 모든 FastAPI 라우터 스캔
2. 다음 항목 점검:
   - async/await 패턴 일관성
   - Pydantic v2 모델 사용
   - Redis 캐싱 적용
   - 에러 핸들링
   - 응답 모델 정의
3. 성능 개선 포인트 식별
4. 개선 보고서 생성

## Check Items
- [ ] 모든 I/O에 async/await 사용
- [ ] Pydantic v2 ConfigDict 사용
- [ ] 적절한 HTTP 상태 코드
- [ ] 일관된 응답 형식
- [ ] 에러 응답 표준화
- [ ] 캐싱 전략 적용
- [ ] Rate limiting 고려

## Output Format
```
## API Health Report

### Endpoints Analyzed: N

### Issues Found
| Endpoint | Issue | Severity |
|----------|-------|----------|
| /api/x   | ...   | High     |

### Recommendations
- [개선 권장사항]

### Good Practices Found
- [잘 적용된 패턴들]
```
