---
name: youtube-generate
description: 유튜브 영상 올인원 생성 - 소재 파일 → 리서치 → 대본 → 재료 생성 → 블로그
argument-hint: "<소재파일경로>"
allowed-tools: "Bash(python *), Read, AskUserQuestion"
---

# YouTube 영상 제작 프로세스

소재 파일 하나로 유튜브 영상 제작에 필요한 모든 재료를 생성합니다.

## 프로세스

### Step 1: 경쟁 영상 리서치 (Gemini)

소재와 유사한 인기 유튜브 영상을 검색하고 대본/구조를 분석합니다.
youtube-transcript-api로 자막을 추출하고, yt-dlp로 메타데이터를 가져온 뒤, Gemini가 구조 분석을 수행합니다.

```bash
python orchestrator.py --research $ARGUMENTS
```

실행 디렉토리: C:\YouTube

완료 후 사용자에게 보고:
- 검색된 영상 수와 제목
- 분석 결과 요약
- output/research/[타임스탬프]/ 폴더 경로

사용자에게 "리서치 결과를 확인하셨나요? 대본 생성으로 넘어갈까요?" 확인 후 Step 2로.

### Step 2: 올인원 스크립트 생성 (Claude)

소재 + Step 1 리서치 결과를 참고하여 제목, 썸네일 문구, 대본을 한 번에 생성합니다.

```bash
python orchestrator.py --write $ARGUMENTS
```

완료 후:
1. 생성된 title.txt 내용을 사용자에게 보여주기
2. 생성된 thumbnail.txt 내용을 사용자에게 보여주기
3. 생성된 script.txt 내용을 사용자에게 보여주기
4. **반드시** "대본 내용이 괜찮으신가요? 수정할 부분이 있으면 말씀해주세요." 확인

사용자가 수정 요청하면 script.txt를 수정합니다.
사용자가 OK하면 Step 3으로.

### Step 3: 영상 재료 생성

확정된 대본을 장면별로 분리한 뒤, 각 장면에 맞는 이미지 프롬프트, 음성, 편집 가이드, 업로드 정보(제목/설명/태그)를 생성합니다.

사용자에게 롱폼/숏폼 여부를 확인합니다.

```bash
# 롱폼
python orchestrator.py output/scripts/[타임스탬프]/script.txt

# 숏폼
python orchestrator.py output/scripts/[타임스탬프]/script.txt --short
```

원본 소재 파일이 있으면 `--source` 옵션으로 지정합니다 (블로그 생성 시 사용):
```bash
python orchestrator.py output/scripts/[타임스탬프]/script.txt --source input/topic.txt
```

완료 후 사용자에게 보고:
- 생성된 파일 목록 (음성, 이미지 프롬프트, 편집 가이드, 업로드 정보)
- output 폴더 경로
- "영상 편집 후 유튜브에 업로드하세요. 업로드 후 블로그 글이 필요하면 유튜브 URL을 알려주세요."

### Step 4: 블로그 글 생성 (선택)

사용자가 유튜브 URL을 제공하면 원본 소재를 기반으로 블로그 글을 생성합니다.
(대본이 아닌 원본 소재 + 유튜브 URL로 블로그를 작성합니다.)

```bash
python orchestrator.py --blog <유튜브URL> output/[Step3 폴더경로]
```

완료 후 사용자에게 보고:
- 생성된 블로그 글 미리보기
- 파일 경로

## 사용 예시

```
/youtube-generate input/topic.txt
/youtube-generate C:\YouTube\input\건강정보_소재.txt
```

## 관련 파일

- 오케스트레이터: `orchestrator.py`
- 경쟁 분석: `modules/competitor_analyzer.py`
- 스크립트 생성: `modules/script_writer.py`
- 음성 생성: `modules/voice_generator.py`
- 이미지 프롬프트: `modules/prompt_generator.py`
- 편집 가이드: `modules/guide_generator.py`
- 업로드 정보: `modules/metadata_generator.py`
- 블로그: `modules/blog_generator.py`
- 설정: `config/settings.yaml`
