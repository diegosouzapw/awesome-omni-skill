---
name: youtube-shorts-automation
description: YouTube Shorts 자동 생성 및 업로드 파이프라인. Deevid AI Agent로 이미지→영상(BGM+음성 포함) 생성 후 YouTube에 업로드. 크론잡으로 매일 자동 실행 가능. Use when generating short-form vertical videos, creating AI-generated video content, uploading to YouTube Shorts, or automating daily video content pipelines.
---

# YouTube Shorts Automation

Deevid AI로 이미지/영상 생성 → YouTube Shorts 업로드 자동화 스킬.

## 전체 파이프라인

```
1. 이미지 생성 (Deevid AI)
2. Agent 영상 생성 (Deevid Agent — 오디오 포함)
3. 영상 다운로드
4. YouTube 업로드
5. (선택) Telegram으로 결과 전송
```

## 핵심 규칙

- **⚠️ "이미지를 동영상으로" 도구 사용 금지** → 무음 영상만 나옴
- **✅ Deevid Agent** (`https://deevid.ai/ko/agent`) 사용 → BGM+대사 포함
- 영상 비율: **9:16** (세로, Shorts 필수)
- 영상 길이: **60초 이하** (8~10초 권장)

## 설정 파일 구조

채널별 설정을 JSON으로 관리. 예시: [references/config_example.json](references/config_example.json)

필수 필드:
- `channel`: 채널명
- `deevid_prompt`: 이미지 생성 프롬프트 (영어, 9:16 명시)
- `youtube.title_template`: 업로드 제목 (`#shorts` 포함)
- `youtube.description_template`: 업로드 설명
- `youtube.tags`: 쉼표 구분 태그

## 단계별 실행

### 1. 이미지 생성
Deevid AI 웹에서 이미지 생성. 프롬프트에 `9:16 vertical format` 포함.
- 비용: 6 크레딧/장
- 상세: [references/deevid-agent-workflow.md](references/deevid-agent-workflow.md)

### 2. Agent 영상 생성
Deevid Agent에 이미지 업로드 + 프롬프트 → 영상 생성 (2-5분 소요).
- 비용: 20 크레딧/8초
- 모델: Start Image Master V2.0
- 상세: [references/deevid-agent-workflow.md](references/deevid-agent-workflow.md)

### 3. YouTube 업로드
```bash
python3 scripts/youtube_upload.py \
  --file video.mp4 \
  --title "제목 #shorts" \
  --description "설명" \
  --tags "tag1,tag2"
```
- 사전 설정: OAuth `client_secret.json` + `token.json` 필요
- 상세: [references/youtube-upload.md](references/youtube-upload.md)

### 4. 크론잡 등록 (OpenClaw)
매일 정해진 시간에 isolated session으로 파이프라인 실행.
크론잡 payload에 전체 워크플로 설명 + 환경 경로 포함.

## 트러블슈팅

| 문제 | 원인 | 해결 |
|------|------|------|
| 무음 영상 | "이미지를 동영상으로" 도구 사용 | Agent 사용으로 전환 |
| 업로드 실패 | token.json 만료 | 재인증 또는 refresh |
| Deevid 로그인 풀림 | 세션 만료 | 브라우저에서 재로그인 |
| 영상 URL 추출 실패 | SPA 렌더링 지연 | 대기 시간 늘리기 |
