---
name: upgrade-claude-code
description: Claude Code 설정 업그레이드
user-invocable: true
---

# Claude Code 설정 업그레이드

현재 설정을 분석하고 **최신 트렌드를 반영**하여 업그레이드를 제안합니다.
**어떤 프로젝트, 어떤 언어에서든** 동일하게 작동합니다.

## 사용법

```
/upgrade-claude-code
```

## 왜 이 커맨드가 필요한가?

```
문제: AI 도구는 빠르게 변화하므로 설정이 금방 구식이 됨
해결: WebSearch로 최신 트렌드 조사 + 현재 설정 분석 + 개선 제안
결과: 항상 최신 베스트 프랙티스 유지 (정기적으로 실행 권장)
```

## 수행 작업

### 1단계: 마스터 가이드 참조

**반드시** 다음 문서를 먼저 읽습니다:

```
.claude/docs/CLAUDE-CODE-MASTERY.md           # 목차 및 핵심 개념
.claude/docs/mastery/04-curriculum.md         # 레벨 시스템 정의 (단일 소스)
.claude/docs/mastery/05-advanced.md           # 최신 트렌드 분석 방법
.claude/docs/mastery/01-settings-guide.md     # 설정 요소별 상세 (MCP 추천 포함)
```

### 2단계: 현재 설정 분석

```bash
# 분석 대상
1. CLAUDE.md - 규칙 수, 상세도
2. .claude/settings.local.json - 훅, 권한 설정
3. .claude/commands/ - 커맨드 수, 품질
4. .claude/agents/ - 에이전트 수, 전문성
5. .claude/skills/ - 스킬 수, 도메인 커버리지
6. .mcp.json - 연결된 외부 도구
7. .github/workflows/ - CI/CD 자동화
```

### 3단계: 레벨 및 체크리스트 평가

**중요**: 레벨 정의, 항목별 체크리스트, 대시보드 형식은 모두 `.claude/docs/mastery/04-curriculum.md`의 "4.0 Claude Code 설정 현황 시스템"을 참조합니다.

해당 문서에서 다음 내용을 확인:
- 레벨 정의 (0-5)
- 항목별 체크리스트 (8개 항목)
- 대시보드 출력 형식
- 빠른 레벨업 가이드

### 4단계: 시간대별 트렌드 조사 (WebSearch)

AI 도구는 빠르게 변화하므로, 시간대별로 검색을 세분화합니다.

#### 4.1 년도 트렌드 (연간 베스트 프랙티스)

검색어:
```
- "Claude Code best practices {current_year}"
- "Claude Code setup guide {current_year}"
- "Boris Cherny Claude Code tips"
```

#### 4.2 월간 트렌드 (최근 업데이트)

검색어:
```
- "Claude Code updates {current_month} {current_year}"
- "Claude Code new features {current_month}"
- "Anthropic Claude Code changelog {current_month}"
```

#### 4.3 주간 트렌드 (핫 토픽)

검색어:
```
- "Claude Code this week"
- "Claude Code latest news"
- "MCP servers new releases"
```

#### 4.4 카테고리별 검색 (선택적)

| 카테고리 | 검색어 | 중요도 |
|----------|--------|:------:|
| Features | "Claude Code new feature" | 높음 |
| Breaking | "Claude Code breaking change deprecated" | 높음 |
| Integration | "Claude MCP server new {current_year}" | 중간 |

#### 4.5 플랫폼별 검색 (선택적)

| 플랫폼 | 검색어 | 특징 |
|--------|--------|------|
| Reddit | site:reddit.com Claude Code {current_month} | 실사용 리뷰, 팁 |
| HN | site:news.ycombinator.com Claude Code | 기술적 토론, 비판적 시각 |
| Discord | Anthropic Discord 채널 직접 확인 | 실시간 기술 지원 |
| GitHub | "Claude Code" in:readme stars:>50 pushed:>{last_week} | 인기 프로젝트 |

### 5단계: 개선점 식별

```markdown
## 개선점 우선순위

### 높음 (레벨업 가능)
- [x] CLAUDE.md 규칙 상세화 → 레벨 1 달성
- [x] Skills 폴더 추가 → 레벨 4 달성
- [x] 누락된 Agents 추가 → 레벨 3 달성

### 중간 (체크리스트 완성)
- [ ] 새로운 MCP 서버 연결
- [ ] 커맨드 추가
- [ ] 훅 고도화

### 낮음 (세부 요소 보완)
- [ ] 기존 설정 최적화
- [ ] 문서 개선
```

### 6단계: 사용자에게 제안

```markdown
## 업그레이드 제안

### 현재 레벨: {N} ({레벨명}) - {X}/8 항목

### 권장 업그레이드

1. **[높음]** Skills 폴더 추가
   - 효과: 레벨 4 달성
   - 설명: 도메인별 전문 컨텍스트 제공

2. **[중간]** 새 MCP 서버 연결
   - 효과: MCP 체크리스트 완성
   - 설명: 프로젝트 의존성 기반 연동

3. **[낮음]** 커맨드 추가
   - 효과: Commands 세부 요소 완성
   - 설명: 추가 자동화

### 트렌드 분석 결과

🔥 **년도 트렌드 ({current_year})**:
| 트렌드 | 설명 | 적용 권장 |
|--------|------|:--------:|
| {트렌드 1} | {설명} | 높음/중간/낮음 |
| {트렌드 2} | {설명} | 높음/중간/낮음 |

⚡ **월간 트렌드 ({current_month})**:
| 업데이트 | 날짜 | 영향도 |
|----------|------|:------:|
| {업데이트 1} | {날짜} | 높음/중간/낮음 |

📢 **주간 핫토픽**:
- 🔥 [높음] {핫토픽 1}
- ⚡ [중간] {핫토픽 2}
- 📢 [정보] {핫토픽 3}
```

### 7단계: 사용자 승인 대기

```
업그레이드를 적용하시겠습니까?

1. 전체 적용
2. 선택적 적용 (번호 선택)
3. 취소
```

### 8단계: 업그레이드 적용

사용자 승인 시:
1. 백업 생성 (기존 파일)
2. 새 설정 적용
3. 변경 사항 요약

### 9단계: 지속적 업데이트 알림 (선택)

프로젝트에 트렌드 체크 리마인더를 제안합니다:

```markdown
## CLAUDE.md에 추가할 내용 (선택)

## 트렌드 체크 일정
- 마지막 체크: {today}
- 다음 체크 권장: {next_check_date}
- 실행: /upgrade-claude-code
```

**권장 체크 주기**:

| 프로젝트 유형 | 권장 주기 | 이유 |
|--------------|:--------:|------|
| 활발한 개발 | 주 1회 | AI 도구 변화가 빠름 |
| 유지보수 | 월 1회 | 주요 업데이트만 확인 |
| 안정화 단계 | 분기 1회 | Breaking Changes만 확인 |

## 결과 출력

```
🚀 Claude Code 업그레이드 완료!

📊 레벨 변화: {N} ({레벨명}) → {N+1} ({새레벨명})
📈 진행 현황: {X}/8 → {Y}/8 항목

📝 적용된 변경:
- Skills 폴더 생성 (4개) ✅
- 새 MCP 서버 연결 (MongoDB) ✅
- CLAUDE.md 규칙 추가 ✅

🔥 반영된 최신 트렌드:
- {트렌드 1}
- {트렌드 2}

🎯 다음 단계:
- 새로운 설정 테스트
- 팀원과 공유
- /learn-claude-code 로 새 기능 학습
```

## 롤백 방법

```bash
# 백업에서 복원
git checkout HEAD~1 -- .claude/
git checkout HEAD~1 -- CLAUDE.md
git checkout HEAD~1 -- .mcp.json
```

## 에러 처리

### 마스터 가이드를 찾을 수 없는 경우

```
⚠️ 마스터 가이드 문서를 찾을 수 없습니다.

해결 방법:
1. bkit-starter 저장소에서 .claude/docs/ 폴더를 복사하세요
2. /learn-claude-code 로 현재 상태 확인
```

### WebSearch 실패 시

```
⚠️ 트렌드 검색에 실패했습니다.

가능한 원인:
- 네트워크 연결 문제
- WebSearch 도구 사용 불가

대안:
→ 공식 채널에서 직접 확인 (아래 참고 문서 참조)
→ 트렌드 분석 없이 현재 설정만 평가: /learn-claude-code
```

### 업그레이드 적용 실패 시

```
⚠️ 업그레이드 적용에 실패했습니다.

복구 방법:
→ git checkout HEAD~1 -- .claude/ (위 롤백 방법 참조)

수동 적용하려면:
→ 01-settings-guide.md 참조하여 직접 수정
```

## 참고 문서

### 내부 문서
- .claude/docs/CLAUDE-CODE-MASTERY.md
- .claude/docs/mastery/04-curriculum.md **(레벨 시스템 정의 - 단일 소스)**
- .claude/docs/mastery/05-advanced.md (트렌드 분석)
- .claude/docs/mastery/01-settings-guide.md (MCP 추천 전략 포함)

### 공식 채널
- [Anthropic Engineering Blog](https://www.anthropic.com/engineering)
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- [MCP Registry](https://registry.modelcontextprotocol.io/)

### 커뮤니티 채널
- [Anthropic Discord](https://discord.gg/anthropic) - API 지원, MCP 논의
- [r/ClaudeAI](https://reddit.com/r/ClaudeAI) - Claude 관련 토론
- [Hacker News](https://news.ycombinator.com) - 기술적 토론
