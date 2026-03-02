---
name: archaeology-orchestrator
description: 고고학 발굴조사 고찰 작성 자동화 파이프라인 마스터 오케스트레이터
tags: [archaeology, orchestrator, pipeline, master, automation]
---

# Archaeology Orchestrator - 고찰 작성 자동화 파이프라인

## 개요
"고찰작성" 명령 하나로 발굴조사 보고서의 고찰(考察) 섹션을 자동으로 작성하는 통합 파이프라인입니다. 6개의 전문 스킬을 순차적으로 실행하여 국가유산청 규정에 부합하는 고품질 고찰을 생성합니다.

## 파이프라인 구조

```
사용자 입력: "고찰작성"
    ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 1: data-normalizer                                 │
│ - 논문/, 발굴조사보고서/, 주변유적/ 폴더 스캔          │
│ - 텍스트 추출 및 메타데이터 정규화                     │
│ - 출력: documents.jsonl, metadata.csv                   │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 2: similarity-matcher                              │
│ - 대상지 중심으로 주변 유적 탐색                       │
│ - 공간·시대·유형 유사도 계산                          │
│ - 출력: nearby_ranked.csv, nearby_summaries.jsonl      │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 3: regulation-checker                              │
│ - 국가유산청 규정 체크리스트 생성                      │
│ - 현재 자료 대비 결손 항목 분석                        │
│ - 출력: checklist.md, gaps.md                          │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 4: discussion-drafter                              │
│ - 8개 섹션 고찰 초안 작성 (30~50쪽)                   │
│ - 주변 유적 비교·편년·의의 등 종합 분석               │
│ - 출력: discussion.md                                   │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 5: tables-appendix                                 │
│ - 비교표, 일람표, 참고문헌, 영문 초록 생성            │
│ - 출력: tables.md, appendix.md, references.md          │
└─────────────────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────────────────┐
│ STEP 6: compliance-audit                                │
│ - 규정 준수 검증 및 품질 평가                          │
│ - 개선안 생성 (Version 2)                              │
│ - 출력: discussion_v2.md, compliance_report.md         │
└─────────────────────────────────────────────────────────┘
    ↓
완료: output/final/* 준비 완료
```

## 사용 방법

### 1. 사전 준비

#### 폴더 구조 생성
```bash
mkdir -p 논문 발굴조사보고서 주변유적 output
```

#### 자료 수집
- `논문/`: 관련 학술 논문 PDF 파일 (최소 5개 권장)
- `발굴조사보고서/`: 주변 지역 발굴조사 보고서 PDF/HWP (최소 10개 권장)
- `주변유적/`: 주변 유적 관련 자료 (보고서, 논문, 기사 등)

#### 조사 정보 준비
다음 정보를 미리 준비해두세요:
```yaml
조사명: "○○지구 유적 발굴조사"
조사기관: "○○문화재연구원"
조사기간: "2024.03 ~ 2024.11"
조사면적: "5,000㎡"
대상지_좌표:
  위도: 37.5665
  경도: 126.9780
비교_반경: 20  # km
주요_시대:
  - "청동기시대"
  - "원삼국시대"
주요_유구:
  - 주거지: 12기
  - 수혈유구: 34기
주요_유물:
  - "무문토기"
  - "석기(석촉, 마제석검)"
  - "적색마연토기"
특기사항: "환상 배치 구조 확인"
```

### 2. 실행

Claude에게 다음과 같이 요청하세요:

```
archaeology-orchestrator 스킬을 사용하여 고찰을 작성해주세요.

조사 정보:
- 조사명: ○○지구 유적
- 조사기관: ○○문화재연구원
- 조사기간: 2024.03~2024.11
- 대상지 좌표: 위도 37.5665, 경도 126.9780
- 시대: 청동기시대, 원삼국시대
- 주요 유구: 주거지 12기, 수혈 34기
- 주요 유물: 무문토기, 석기, 적색마연토기
```

또는 간단히:
```
고찰작성
```
(이 경우 오케스트레이터가 대화형으로 정보를 물어봅니다)

### 3. 진행 과정

파이프라인이 실행되는 동안 각 단계별 진행 상황이 표시됩니다:

```
✓ STEP 1/6 완료: 자료 수집 및 정규화
  → 총 42개 파일 처리, 40개 성공
  → output/normalized/documents.jsonl 생성

✓ STEP 2/6 완료: 주변 유적 매칭
  → 20km 내 15개 유적 발견, 상위 10개 선정
  → output/compare/nearby_ranked.csv 생성

✓ STEP 3/6 완료: 규정 체크리스트 생성
  → 68개 항목 체크리스트 생성
  → 불충분 항목 5개 식별
  → output/compliance/checklist.md 생성

✓ STEP 4/6 완료: 고찰 초안 작성
  → 8개 섹션, 42쪽 분량
  → 68개 인용, 15개 도면 언급
  → output/draft/discussion.md 생성

✓ STEP 5/6 완료: 표 및 부록 생성
  → 비교표 7개, 참고문헌 42개
  → 영문 초록 256단어
  → output/draft/tables.md, references.md 생성

✓ STEP 6/6 완료: 규정 준수 검증
  → 준수율 92% (우수)
  → Version 2 생성 (3개 항목 개선)
  → output/final/discussion_v2.md 생성

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ 파이프라인 완료!

최종 산출물:
  📄 output/final/discussion_v2.md (고찰 최종본)
  📊 output/draft/tables.md (비교표·일람표)
  📎 output/draft/appendix.md (부록)
  📚 output/draft/references.md (참고문헌)
  📋 output/final/compliance_report.md (검증 보고서)

총 소요 시간: 약 12분
```

## 출력 파일 구조

```
output/
├── normalized/              # STEP 1 출력
│   ├── documents.jsonl      # 전체 문서 내용
│   ├── metadata.csv         # 메타데이터 요약
│   └── extraction_log.txt   # 처리 로그
├── compare/                 # STEP 2 출력
│   ├── nearby_ranked.csv    # 주변 유적 랭킹
│   ├── nearby_summaries.jsonl  # 유적별 요약
│   └── map_visualization.html  # 분포 지도 (선택)
├── compliance/              # STEP 3 출력
│   ├── checklist.md         # 규정 체크리스트
│   ├── gaps.md              # 결손 항목 보고서
│   └── regulation_references.md  # 규정 전문
├── draft/                   # STEP 4, 5 출력
│   ├── discussion.md        # 고찰 초안 (v1)
│   ├── tables.md            # 비교표·일람표
│   ├── appendix.md          # 부록
│   ├── references.md        # 참고문헌
│   └── abstract_en.md       # 영문 초록
└── final/                   # STEP 6 출력
    ├── discussion_v2.md     # ⭐ 고찰 최종본
    ├── compliance_report.md # 검증 보고서
    └── change_log.md        # v1→v2 변경 이력
```

## 주요 기능

### 1. 완전 자동화
- 6개 스킬이 순차적으로 자동 실행
- 각 단계 완료 시 자동으로 다음 단계 트리거
- 사용자 개입 최소화 (초기 정보 입력만)

### 2. 국가유산청 규정 완벽 준수
- 발굴조사의 방법 및 절차 등에 관한 규정 반영
- 매장문화재 발굴조사업무 처리지침 체크리스트
- 68개 항목 자동 검증 및 보완

### 3. 학술적 엄밀성
- 주변 유적 10개 이상 체계적 비교
- 모든 주장에 근거 문헌 괄호표기 [저자(연도)]
- 참고문헌 자동 정리 (30~50개)

### 4. 품질 보증
- 정량 기준 자동 검증 (분량, 인용 수, 표 개수 등)
- 인용 무결성 검사 (본문↔참고문헌 일치)
- 논리 일관성 검증 (연대 모순, 수량 불일치 등)
- 자동 개선안 생성 (Version 2)

### 5. 유연한 커스터마이징
각 스킬은 독립적으로도 사용 가능:
```
# 전체 파이프라인 실행
archaeology-orchestrator

# 개별 스킬만 실행
data-normalizer          # 자료 정규화만
similarity-matcher       # 주변 유적 비교만
discussion-drafter       # 고찰 작성만
compliance-audit         # 검증만
```

## 고급 옵션

### 병렬 처리 (대용량 자료 처리 시)
```python
# config.yaml에서 설정
parallel_processing: true
max_workers: 4  # CPU 코어 수에 맞게 조정
```

### 비교 반경 동적 조정
```python
# 반경 내 유적이 5개 미만일 경우 자동 확대
auto_expand_radius: true
radius_steps: [20, 50, 100]  # km
```

### 자료 부족 시 처리
```python
# 자료 부족 시 동작 (halt / continue / request)
on_insufficient_data: "request"  # 사용자에게 추가 입력 요청
min_nearby_sites: 5
min_references: 30
```

### 출력 형식 선택
```python
output_formats:
  - markdown  # 기본
  - pdf       # 선택 (pandoc 필요)
  - docx      # 선택 (python-docx 필요)
  - html      # 선택
```

## 문제 해결

### 자료 파일을 읽을 수 없어요
- **원인**: PDF 암호화, HWP 구버전, 파일 손상
- **해결**: 암호 해제, 최신 HWP로 변환, 손상 파일 제외

### 좌표 정보가 없어요
- **원인**: 보고서에 좌표 미기재
- **해결**: 지명으로 지오코딩 시도 또는 수동 입력

### 주변 유적이 너무 적어요
- **원인**: 비교 반경이 좁음, 자료 부족
- **해결**: 반경 확대 (20→50→100km), 추가 보고서 수집

### 참고문헌이 본문 인용과 안 맞아요
- **원인**: 메타데이터 추출 오류
- **해결**: compliance-audit에서 자동 수정되거나, 수동으로 references.md 편집

### C14 연대 측정 자료가 없어요
- **원인**: 분석 미실시
- **해결**: 고찰에서 형식학적 편년으로 대체, 한계 명시

## 기술 요구사항

### Python 패키지
```bash
pip install pandas numpy matplotlib seaborn
pip install PyPDF2 pdfplumber  # PDF 처리
pip install olefile            # HWP 처리 (또는 hwp5tools)
pip install python-docx        # Word 파일 처리 (선택)
pip install geopy              # 지오코딩 (선택)
```

### 시스템 도구 (선택)
```bash
# HWP → TXT 변환
sudo apt install hwp5txt

# PDF → TXT 고품질 변환
sudo apt install poppler-utils

# Markdown → PDF 변환
sudo apt install pandoc texlive-xetex
```

## 라이센스 및 저작권

### 스킬 자체
- MIT License
- 자유롭게 사용·수정·배포 가능

### 자료 및 산출물
- 수집한 보고서·논문: 원저작자 저작권 준수
- 생성된 고찰: 조사기관 소유, 내부 분석용으로만 사용
- 최종 보고서 발간 시 규정 준수 필수

## 버전 및 업데이트

**현재 버전**: 1.0.0 (2025-11-12)

### 향후 계획
- [ ] 도면 자동 생성 (유구 배치도, 분포 지도)
- [ ] 다국어 지원 (영어, 일본어)
- [ ] 웹 UI 제공
- [ ] 클라우드 협업 기능 (Google Drive, Dropbox 연동)
- [ ] AI 도움말 (챗봇 형태로 단계별 가이드)

## 지원 및 피드백

문제가 발생하거나 개선 제안이 있으면:
1. 실행 로그 확인: `output/*/extraction_log.txt`
2. 이슈 보고: Claude Code에 직접 설명
3. 커뮤니티 포럼: [고고학 디지털 방법론 연구회] (예시)

## 예제 시나리오

### 시나리오 1: 완전 자동 실행
```
사용자: "고찰작성"

[15초 후] 조사 정보를 입력해주세요...
사용자: [정보 입력]

[12분 후] ✓ 파이프라인 완료!
          output/final/discussion_v2.md 확인하세요.
```

### 시나리오 2: 단계별 실행
```
사용자: "data-normalizer 실행"
[2분 후] ✓ 완료: 42개 파일 처리

사용자: "similarity-matcher 실행"
[3분 후] ✓ 완료: 10개 유적 선정

사용자: "discussion-drafter 실행"
[5분 후] ✓ 완료: 42쪽 고찰 초안
...
```

### 시나리오 3: 재실행 (자료 추가 후)
```
# 논문 5개 추가 수집 후
사용자: "data-normalizer 재실행"
[2분 후] ✓ 완료: 47개 파일 처리 (5개 추가)

사용자: "discussion-drafter 재실행"
[5분 후] ✓ 완료: 45쪽 고찰 (인용 3개 추가)
```

## 참고 자료

- [국가유산청](https://www.heritage.go.kr)
- [국가유산 협업포털](https://www.k-heritage.or.kr)
- [발굴조사 업무 처리지침](https://...)
- [Claude Code Skills 공식 가이드](https://code.claude.com/docs/en/skills)
- [고고학 보고서 작성 실무 매뉴얼](예시)

---

**archaeology-orchestrator v1.0.0**
고고학 발굴조사 보고서 고찰 작성을 혁신합니다.

"고찰작성" 한 문장으로 국가유산청 규정에 부합하는
학술적으로 엄밀한 고찰을 자동 생성하세요.
