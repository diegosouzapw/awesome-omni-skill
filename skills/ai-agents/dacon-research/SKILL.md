---
name: dacon-research
description: dacon-info 폴더를 분석해 ChatGPT Deep Research용 프롬프트 생성. "/dacon-research", "딥리서치", "deep research 프롬프트", "연구 프롬프트" 요청에 사용.
version: 1.0.0
---

# Dacon Deep Research Prompt Generator

dacon-info 폴더의 공모전 정보를 분석하여 ChatGPT Deep Research용 종합 프롬프트를 자동 생성합니다.

## Purpose

대회 참가 전략 수립을 위한 **체계적인 리서치 프롬프트 생성**:
- SOTA 모델 및 아키텍처 조사
- 공개 데이터셋 및 전처리 기법
- 최신 논문 및 연구 동향
- 대회 제약사항을 반영한 실전 가이드

## Execution

### Phase 1: dacon-info 폴더 확인 및 읽기

1. 현재 프로젝트 루트에서 `dacon-info/` 폴더 존재 확인

```bash
ls -la ./dacon-info/ 2>/dev/null
```

2. 폴더가 없으면 에러 처리:
```
⚠️ dacon-info 폴더가 존재하지 않습니다.
먼저 /dacon 스킬을 실행하여 공모전 정보를 수집해주세요.

예: /dacon
```

3. 폴더가 있으면 모든 .md 파일 읽기:
   - `overview.md` (대회 주제, 배경)
   - `evaluation.md` (평가 기준)
   - `rules.md` (제약사항)
   - `data.md` (데이터 설명)
   - `schedule.md` (일정)
   - `prize.md` (상금)

파일이 없어도 있는 파일만으로 진행. 단, overview.md가 없으면 경고 표시.

### Phase 2: 핵심 정보 추출

dacon-info 내용에서 다음 항목 자동 추출:

| 항목 | 추출 소스 | 설명 |
|------|----------|------|
| 대회명 | overview.md | ## 대회명 섹션 |
| 대회 주제 | overview.md | ## [주제] 섹션 |
| 배경 | overview.md | ## [배경] 섹션 |
| 평가 지표 | evaluation.md | ### 평가 산식 섹션 |
| 추론 환경 | rules.md | #### ② 추론 환경 사양 |
| 추론 시간 제한 | rules.md | #### ① 추론 시간 제한 |
| 모델 제약사항 | rules.md | ### 모델 앙상블 및 결합 제한 |
| 사전학습 모델 규칙 | rules.md | ### 사전 학습 모델 사용 규칙 |
| 데이터 특성 | data.md | 학습 데이터 제공 여부, 데이터 형태 |

### Phase 3: Deep Research 프롬프트 생성

추출한 정보를 바탕으로 다음 템플릿으로 프롬프트 생성:

```markdown
# ChatGPT Deep Research 요청

## 대회 개요
- **대회명**: {대회명}
- **주제**: {대회 주제}
- **평가 지표**: {평가 지표}
- **추론 환경**: {GPU/CPU/RAM 스펙}
- **추론 시간 제한**: {시간 제한}

## 배경
{배경 요약 - 2~3문장}

## 핵심 제약사항
{rules.md에서 추출한 제약사항 - 불릿 포인트 형식}

예시:
- 단일 모델만 사용 가능 (앙상블 금지)
- Test-Time Augmentation(TTA) 금지
- 영상은 프레임 단위로 분해하여 개별 추론
- 외부 API (OpenAI, Gemini 등) 학습/추론에 사용 금지
- 사전학습 모델은 {날짜} 이전 공개된 것만 사용 가능

## 데이터 특성
{data.md 내용 요약}

---

## 연구 요청 사항

### 1. SOTA 모델 및 아키텍처 조사

{주제}와 관련된 최신 딥러닝 모델들을 조사해주세요:

- 2023-2025년 발표된 주요 모델 아키텍처
- 각 모델의 성능 비교 (정확도, 추론 속도, 파라미터 수, VRAM 사용량)
- **단일 모델 제약** 하에서의 최적 선택지
- 사전학습 가중치가 공개된 모델 목록 (NC 라이선스 이상)
- {추론 환경}에서 실행 가능한 모델 권장

특히 다음 백본 네트워크 비교 분석:
- EfficientNet 계열
- ConvNeXt 계열
- Vision Transformer (ViT) 계열
- Swin Transformer 계열

### 2. 공개 데이터셋 및 전처리 기법

{주제} 관련 학습에 사용할 수 있는 데이터 조사:

- 공개 데이터셋 목록 (라이선스, 크기, 특성 명시)
  - FaceForensics++
  - Celeb-DF
  - DFDC (DeepFake Detection Challenge)
  - WildDeepfake
  - DeeperForensics-1.0
  - 기타 최신 데이터셋
- 효과적인 데이터 증강 기법
- 전처리 파이프라인 모범 사례
- 얼굴 검출/정렬 최적 방법

### 3. 최신 연구 동향

{주제} 분야의 연구 트렌드 분석:

- 2023-2025년 주요 학회(CVPR, ICCV, ECCV, NeurIPS, AAAI) 발표 논문
- 새로운 접근 방식 및 기법
  - Frequency domain 분석
  - Attention 기반 방법
  - Artifact 검출 기법
  - Cross-dataset generalization 전략
- 벤치마크 데이터셋에서의 SOTA 성능 추이
- 향후 연구 방향

### 4. 실전 적용 가이드

위 연구 결과를 바탕으로 다음을 정리해주세요:

- **권장 모델 아키텍처** (이유 포함)
- **추천 학습 전략**
  - Loss function 선택
  - Learning rate 스케줄
  - 정규화 기법
- **주의해야 할 함정 및 팁**
- **{추론 시간/VRAM 제약}을 만족하는 최적화 방안**
  - Mixed precision training/inference
  - Model pruning/quantization
  - Efficient data loading

## 참고 정보

- 추론 환경: {추론 환경 상세}
- 추론 시간 제한: {시간 제한}
- 사전학습 모델 기준일: {날짜}
- 사용 가능 언어: Python
- 학습 VRAM 제한: {학습 VRAM}

---

*이 프롬프트는 {대회명} 참가를 위한 Deep Research용으로 자동 생성되었습니다.*
```

### Phase 4: 출력 및 저장

1. 생성된 프롬프트를 화면에 전체 출력

2. AskUserQuestion으로 파일 저장 여부 확인:
```
question: "생성된 프롬프트를 파일로 저장하시겠습니까?"
header: "Save"
options:
  - label: "Yes (Recommended)"
    description: "dacon-info/research-prompt.md로 저장"
  - label: "No"
    description: "저장하지 않고 화면 출력만 유지"
```

3. "Yes" 선택 시:
   - `dacon-info/research-prompt.md`로 저장
   - 저장 완료 메시지 출력

## Error Handling

| 상황 | 대응 |
|------|------|
| dacon-info 폴더 없음 | 안내 메시지 + /dacon 스킬 실행 제안 |
| overview.md 없음 | 경고 표시 후 있는 파일만으로 진행 |
| 필수 정보 추출 실패 | 해당 섹션은 "[정보 없음]"으로 표시 |
| 파일 저장 실패 | 에러 메시지 출력 후 클립보드 복사 안내 |

## Notes

- 프롬프트는 ChatGPT Plus의 Deep Research 기능에 최적화되어 있습니다
- 생성된 프롬프트를 그대로 복사하여 ChatGPT에 붙여넣기 하면 됩니다
- 대회별 특성에 따라 프롬프트가 자동으로 커스터마이징됩니다
