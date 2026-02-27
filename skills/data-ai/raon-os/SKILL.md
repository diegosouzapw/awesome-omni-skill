---
name: raon-os
version: 0.7.10
description: "AI-powered startup companion for Korean founders. Evaluate business plans, match government funding programs (TIPS/DeepTech/Global TIPS), connect with 3,972+ TIPS-selected startups, get investor recommendations, and integrate with Kakao i OpenBuilder. Features Agentic RAG (HyDE, Multi-Query, CRAG), structured extraction, and Track B financial matching."
metadata:
  openclaw:
    env:
      - name: GEMINI_API_KEY
        description: "Google Gemini API key (recommended for embeddings + LLM)"
        required: false
      - name: OPENROUTER_API_KEY
        description: "OpenRouter API key (multi-model access)"
        required: false
      - name: ANTHROPIC_API_KEY
        description: "Anthropic Claude API key"
        required: false
      - name: OPENAI_API_KEY
        description: "OpenAI API key"
        required: false
      - name: KAKAO_CALLBACK_SECRET
        description: "Kakao i OpenBuilder webhook HMAC secret (optional)"
        required: false
      - name: RAON_API_URL
        description: "Managed API endpoint (optional, for SaaS mode)"
        required: false
      - name: RAON_API_KEY
        description: "Managed API key (optional, for SaaS mode)"
        required: false
    requires:
      bins: ["python3", "node"]
    notes: "At least one LLM API key (GEMINI, OPENROUTER, ANTHROPIC, or OPENAI) is recommended. Falls back to local Ollama if no keys are set. API keys are stored in ~/.openclaw/.env (user-managed, chmod 600 recommended). The skill includes a local HTTP server (port 8400) and crawlers for public government data collection."
---

# Raon OS — Startup Companion (v0.7.10)

## 설치 요구사항

- **Python 3.9+** (macOS 기본 내장, 별도 설치 불필요)
- **Node.js 18+** (`npx @yeomyeonggeori/raon-os` 실행용)
- **LLM API 키** (아래 중 하나, 우선순위 순):

| 환경변수 | 설명 | 비고 |
|----------|------|------|
| `OPENROUTER_API_KEY` | **1순위** — OpenClaw 지원 모든 모델 | 추천 |
| `GEMINI_API_KEY` | **2순위** — Google Gemini + 임베딩 | |
| `ANTHROPIC_API_KEY` | **3순위** — Claude | |
| `OPENAI_API_KEY` | **4순위** — GPT + 임베딩 | |
| Ollama 로컬 | 자동 감지 (키 없을 때) | 선택 — `raon.sh install-model` |

- **벡터 검색**: `GEMINI_API_KEY` 또는 `OPENAI_API_KEY` 있으면 자동 활성화. 없으면 BM25 키워드 검색으로 동작.

## 빠른 시작

```bash
# 1. OpenClaw 설치
npm install -g openclaw

# 2. 스킬 설치
openclaw skill install @yeomyeonggeori/raon-os

# 3. API 키 설정 (권장: OpenRouter)
echo "OPENROUTER_API_KEY=sk-or-..." >> ~/.openclaw/.env
chmod 600 ~/.openclaw/.env  # 보안: 소유자만 읽기/쓰기

# 4. 모델 override (선택) — 기본은 프로바이더별 최적 모델 자동 선택
echo "RAON_MODEL=anthropic/claude-opus-4-5" >> ~/.openclaw/.env

# 5. 연결 테스트
python3 scripts/raon_llm.py --detect
```

## LLM 설정 (`raon_llm.py`)

모든 API 키는 `~/.openclaw/.env` 에 저장 (환경변수 우선):

```bash
# ~/.openclaw/.env 예시
OPENROUTER_API_KEY=sk-or-v1-xxxx   # 1순위 (추천)
GEMINI_API_KEY=AIzaSy-xxxx          # 2순위 + 임베딩
ANTHROPIC_API_KEY=sk-ant-xxxx       # 3순위
OPENAI_API_KEY=sk-xxxx              # 4순위 + 임베딩
RAON_MODEL=google/gemini-2.5-flash  # 모델 강제 지정 (선택)
RAON_LLM_PROVIDER=openrouter       # 프로바이더 강제 지정 (선택)
```

스타트업 파운더를 위한 AI 동료. 아이디어를 사업으로 만드는 전 과정을 지원한다.

## 기능

### 1. biz-plan — 사업계획서 평가
사업계획서(PDF/텍스트)를 분석하여 점수 + 개선안을 제공한다.

```bash
# PDF 평가
{baseDir}/scripts/raon.sh biz-plan evaluate --file /path/to/plan.pdf

# 텍스트 평가
{baseDir}/scripts/raon.sh biz-plan evaluate --text "사업 아이디어 설명..."

# JSON 형식 출력
{baseDir}/scripts/raon.sh biz-plan evaluate --file /path/to/plan.pdf --json

# 결과를 파일로 저장
{baseDir}/scripts/raon.sh biz-plan evaluate --file /path/to/plan.pdf --output result.md

# 두 사업계획서 비교 분석
{baseDir}/scripts/raon.sh biz-plan evaluate --file plan_a.pdf --file plan_b.pdf

# 개선안 생성
{baseDir}/scripts/raon.sh biz-plan improve --file /path/to/plan.pdf

# 평가 히스토리 조회
{baseDir}/scripts/raon.sh history
```

**평가 항목:**
- 문제 정의 및 솔루션 적합성
- 시장 규모 및 경쟁 분석
- 비즈니스 모델 타당성
- 팀 역량
- 재무 계획
- 기술 차별성

**출력:** 100점 만점 종합 점수 + 항목별 점수 + 구체적 개선 제안

### 2. gov-funding — 정부 지원사업 매칭
스타트업 프로필 기반으로 적합한 정부 지원사업을 추천한다.

```bash
# 매칭 (사업계획서 기반)
{baseDir}/scripts/raon.sh gov-funding match --file /path/to/plan.pdf

# 매칭 (키워드 기반)
{baseDir}/scripts/raon.sh gov-funding match --industry "AI/SaaS" --stage "early" --region "서울"

# 지원사업 상세 정보
{baseDir}/scripts/raon.sh gov-funding info --program "TIPS"

# 지원서 초안 생성
{baseDir}/scripts/raon.sh gov-funding draft --program "TIPS" --file /path/to/plan.pdf

# 지원 준비 체크리스트
{baseDir}/scripts/raon.sh gov-funding checklist --program "TIPS" --file /path/to/plan.pdf
```

**지원 프로그램:** TIPS, K-Startup 그랜드챌린지, 창업성장기술개발, 예비창업패키지, 초기창업패키지 등

### 3. investor-match — 투자자 매칭 (추후 연동)
스타트업 단계/산업에 맞는 투자자를 추천한다.

```bash
# 투자자 추천
{baseDir}/scripts/raon.sh investor-match --stage "pre-a" --industry "AI" --amount "1M"
```

## PDF 처리 (중요)

사용자가 PDF를 직접 보내면 바이너리가 프롬프트에 들어가 토큰 한도를 초과한다.
**반드시 텍스트 추출 후 평가해야 한다.**

```bash
# 방법 1: evaluate.py가 내부에서 PDF 텍스트 추출
{baseDir}/scripts/raon.sh biz-plan evaluate --file /path/to/plan.pdf

# 방법 2: 전처리 스크립트
scripts/pdf-evaluate.sh /path/to/plan.pdf
```

**OpenClaw에서 PDF 파일이 첨부된 경우:**
1. PDF가 media/inbound/에 저장됨
2. PDF를 프롬프트에 인라인하지 말 것 (토큰 폭발)
3. exec 도구로 evaluate.py --file <경로>를 실행하여 평가
4. 결과를 사용자에게 전달

## 사용 흐름

일반적인 파운더 여정:

```
1. "내 사업계획서 평가해줘" → biz-plan evaluate
2. "어떻게 고치면 돼?" → biz-plan improve
3. "이걸로 지원할 수 있는 정부사업 있어?" → gov-funding match
4. "TIPS 지원서 초안 만들어줘" → gov-funding draft
5. "투자자도 연결해줘" → investor-match
```

## Investor Match

```bash
raon.sh investor-match profile --file <pdf>
# or
curl -X POST http://localhost:8400/v1/investor ...
```

투자자 관점에서 사업계획서를 분석하고 매력도 프로필을 생성한다.
- Deal Summary (1분 요약)
- 타겟 투자자 유형 (Seed/Pre-A, Sector)
- Investment Highlights & Red Flags
- 피칭 팁

## HTTP API 서버

로컬 REST API 서버를 띄워 웹챗이나 외부 서비스에서 연동할 수 있다:

```bash
raon.sh serve           # 기본 포트 8400
raon.sh serve 9000      # 커스텀 포트
```

엔드포인트:
- `GET /health` — 헬스체크
- `GET /v1/modes` — 지원 모드 목록
- `POST /v1/evaluate` — 사업계획서 평가
- `POST /v1/improve` — 사업계획서 개선
- `POST /v1/match` — 정부 지원사업 매칭
- `POST /v1/draft` — 지원서 초안 (program 필수)
- `POST /v1/checklist` — 지원 준비 점검 (program 필수)
- `POST /v1/investor` — 투자자 프로필 분석

요청 예시:
```bash
curl -X POST http://localhost:8400/v1/evaluate \
  -H "Content-Type: application/json" \
  -d '{"text": "사업계획서 내용..."}'
```

CORS 지원됨 (웹 프론트엔드 연동 가능).

## API 연동

현재 버전은 로컬 분석 (LLM 기반 RAG)으로 동작한다.
K-Startup AI API 연동 시 환경변수 설정:

```bash
export RAON_API_URL="https://api.k-startup.ai"
export RAON_API_KEY="your-api-key"
```

API가 설정되지 않으면 로컬 LLM + RAG 파이프라인으로 폴백한다.

## 평가 기준 참고

정부 지원사업별 심사 기준은 references/ 디렉토리 참조:
- `references/tips-criteria.md` — TIPS 심사 기준
- `references/gov-programs.md` — 주요 정부 지원사업 목록 및 자격 요건
