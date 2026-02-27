---
name: ppt-to-video-workflow
description: PPT/슬라이드를 나레이션과 자막이 포함된 영상으로 변환합니다. PPTX 파일 또는 slides.json에서 슬라이드 이미지를 추출/렌더링하고, TTS로 나레이션을 생성하며, 자막을 추가하여 최종 MP4 영상을 만듭니다. "PPT를 영상으로 만들어줘", "발표 영상 생성", "자막 포함 영상 만들기" 요청 시 사용합니다.
---

# PPT to Video Workflow

PPT/슬라이드를 나레이션과 자막이 포함된 영상으로 변환하는 종합 스킬입니다.

## 트리거

다음과 같은 요청 시 이 스킬을 사용합니다:
- "PPT를 영상으로 만들어줘"
- "슬라이드 영상 생성해줘"
- "발표 영상 만들어줘"
- "나레이션이 포함된 영상 생성"
- "자막 포함 영상 만들어줘"
- "PPTX 파일을 영상으로 변환"

---

## 전체 워크플로우

```
┌─────────────────────────────────────────────────────────────────────┐
│                        입력 소스 (택1)                               │
├─────────────────────────────────────────────────────────────────────┤
│  [PPTX 파일]              또는           [slides.json]              │
│       │                                       │                     │
│       ▼                                       ▼                     │
│  이미지 추출 (python-pptx)          슬라이드 렌더링 (Pillow)         │
│       │                                       │                     │
│       └───────────────┬───────────────────────┘                     │
│                       ▼                                             │
│              slide_01.png, slide_02.png, ...                        │
└─────────────────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      나레이션 생성                                   │
├─────────────────────────────────────────────────────────────────────┤
│  [slides.json + reference_audio.wav]                                │
│       │                                                             │
│       ▼                                                             │
│  Chatterbox TTS API (슬라이드별 음성 생성)                          │
│       │                                                             │
│       ▼                                                             │
│  narration_01.mp3, narration_02.mp3, ... + timing.json              │
└─────────────────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      자막 생성 (선택)                                │
├─────────────────────────────────────────────────────────────────────┤
│  [timing.json]                                                      │
│       │                                                             │
│       ▼                                                             │
│  문장 분할 + ASS 자막 생성                                          │
│       │                                                             │
│       ▼                                                             │
│  subtitle.ass                                                       │
└─────────────────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      영상 합성                                       │
├─────────────────────────────────────────────────────────────────────┤
│  [slide_*.png + narration_*.mp3 + subtitle.ass]                     │
│       │                                                             │
│       ▼                                                             │
│  FFmpeg (페이드 전환 + 자막 하드코딩)                               │
│       │                                                             │
│       ▼                                                             │
│  final_video.mp4                                                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 입력 형식

### 옵션 1: PPTX 파일
기존 PPTX 파일에서 슬라이드 이미지를 추출합니다.

```python
from pptx import Presentation
from pptx.enum.shapes import MSO_SHAPE_TYPE
from PIL import Image
import io

prs = Presentation("presentation.pptx")
for i, slide in enumerate(prs.slides):
    for shape in slide.shapes:
        if shape.shape_type == MSO_SHAPE_TYPE.PICTURE:
            image = Image.open(io.BytesIO(shape.image.blob))
            image.save(f"slide_{i+1:02d}.png")
```

### 옵션 2: slides.json
구조화된 JSON에서 슬라이드를 렌더링합니다.

```json
{
  "slides": [
    {
      "slide": 1,
      "title": "슬라이드 제목",
      "content": ["항목1", "항목2", "항목3"],
      "narration": "이 슬라이드에서 읽을 전체 대본 텍스트입니다."
    }
  ]
}
```

### 필수 파일
- **reference_audio.wav**: 음성 클로닝을 위한 참조 음성 (3-10초)

---

## 스크립트

### 1. render_slides.py
slides.json을 1920x1080 이미지로 렌더링합니다.

```bash
# Academic 테마 (기본, 흰색 배경)
python scripts/render_slides.py slides.json -i generated_images/ -o rendered_slides/

# RDF 다크 테마 (골드/청록 강조)
python scripts/render_slides.py slides.json -i generated_images/ -o rendered_slides/ -t rdf
```

**옵션**:
- `-i, --images`: AI 생성 이미지 디렉토리
- `-o, --output`: 출력 디렉토리
- `-t, --theme`: 테마 선택 (`academic` 또는 `rdf`)

**출력**: `rendered_slides/slide_01.png`, `slide_02.png`, ...

### 2. generate_narration.py
슬라이드별 TTS 음성을 생성합니다.

```bash
python scripts/generate_narration.py slides.json reference_audio.wav -o narrations/
```

**옵션**:
- `-o, --output`: 출력 디렉토리
- `-l, --language`: 언어 코드 (기본: `ko`)

**출력**:
- `narrations/narration_01.mp3`, `narration_02.mp3`, ...
- `narrations/timing.json` (시간 정보 + 전체 텍스트)

**timing.json 구조**:
```json
{
  "total_slides": 17,
  "total_duration": 374.96,
  "slides": [
    {
      "slide": 1,
      "audio_file": "narrations/narration_01.mp3",
      "duration": 11.76,
      "text": "전체 나레이션 텍스트..."
    }
  ]
}
```

### 3. generate_subtitles.py
timing.json에서 ASS 자막 파일을 생성합니다.

```bash
python scripts/generate_subtitles.py narrations/timing.json -o subtitle.ass
```

**자막 분할 규칙**:
- 한 자막당 최대 2줄
- 줄당 최대 40자 (한글 기준)
- 구두점(., !, ?) 기준 분할
- 최소 2초 표시 시간 보장

**출력**: `subtitle.ass`

### 4. create_video.py
이미지와 나레이션을 합성하여 영상을 생성합니다.

```bash
# 기본 (자막 없음)
python scripts/create_video.py rendered_slides/ narrations/ -o final_video.mp4

# 자막 포함
python scripts/create_video.py rendered_slides/ narrations/ -o final_video.mp4 --subtitle

# 페이드 효과 없이
python scripts/create_video.py rendered_slides/ narrations/ -o final_video.mp4 --no-fade
```

**옵션**:
- `-o, --output`: 출력 파일명
- `--subtitle`: 자막 포함 (ASS 파일 자동 생성)
- `--no-fade`: 페이드 전환 효과 비활성화

**출력**:
- `final_video.mp4`
- `final_video.ass` (--subtitle 사용 시)

---

## 전체 워크플로우 예시

### 케이스 1: slides.json에서 영상 생성

```bash
# 1. 슬라이드 렌더링
python ~/.claude/skills/ppt-to-video-workflow/scripts/render_slides.py \
    slides.json -i generated_images/ -o rendered_slides/ -t rdf

# 2. 나레이션 생성
python ~/.claude/skills/ppt-to-video-workflow/scripts/generate_narration.py \
    slides.json reference_audio.wav -o narrations/

# 3. 영상 합성 (자막 포함)
python ~/.claude/skills/ppt-to-video-workflow/scripts/create_video.py \
    rendered_slides/ narrations/ -o presentation.mp4 --subtitle
```

### 케이스 2: PPTX 파일에서 영상 생성

```python
# 1. PPTX에서 이미지 추출
from pptx import Presentation
from pptx.enum.shapes import MSO_SHAPE_TYPE
from PIL import Image
import io, os

os.makedirs("pptx_slides", exist_ok=True)
prs = Presentation("presentation.pptx")

for i, slide in enumerate(prs.slides):
    for shape in slide.shapes:
        if shape.shape_type == MSO_SHAPE_TYPE.PICTURE:
            img = Image.open(io.BytesIO(shape.image.blob))
            img = img.resize((1920, 1080), Image.LANCZOS)
            img.save(f"pptx_slides/slide_{i+1:02d}.png")
            break
```

```bash
# 2. 나레이션 생성 (slides.json 필요)
python ~/.claude/skills/ppt-to-video-workflow/scripts/generate_narration.py \
    slides.json audio.wav -o narrations/

# 3. 영상 합성
python ~/.claude/skills/ppt-to-video-workflow/scripts/create_video.py \
    pptx_slides/ narrations/ -o output.mp4 --subtitle
```

### 케이스 3: 기존 영상에 자막만 추가

```bash
# 1. 자막 파일 생성
python ~/.claude/skills/ppt-to-video-workflow/scripts/generate_subtitles.py \
    narrations/timing.json -o subtitle.ass

# 2. FFmpeg로 자막 하드코딩
ffmpeg -i input.mp4 -vf "ass=subtitle.ass" -c:a copy output_with_subtitle.mp4
```

---

## 테마

### Academic (기본)
- 흰색 배경
- 빨간색 강조 (#DC3545)
- 얇은 상단 바
- 심플한 불릿 포인트

### RDF Dark Theme
- 다크 헤더 (#1A1A1A) + 격자 도트 패턴
- 골드 강조 (#F5A623)
- 청록 보조 (#4ECDC4)
- 골드 세로바 섹션 제목
- 이미지에 그림자 효과
- 청록색 하단 바

---

## 설정

| 항목 | 값 |
|------|-----|
| 해상도 | 1920x1080 (Full HD) |
| FPS | 30 |
| 영상 형식 | MP4 (H.264 + AAC) |
| 전환 효과 | 페이드 (0.5초) |
| 한글 폰트 | AppleSDGothicNeo, NanumGothic |
| 테마 | academic, rdf |
| 자막 형식 | ASS (Advanced SubStation Alpha) |
| 자막 폰트 | AppleSDGothicNeo-Medium (42pt) |
| 자막 위치 | 하단 중앙 |
| 자막 배경 | 반투명 검정 (50% opacity) |

---

## 의존성

```bash
pip install pillow pydub gradio-client python-pptx
```

- **Python 3.x**
- **Pillow**: 이미지 렌더링
- **FFmpeg**: 영상 합성 (시스템 설치 필요)
- **pydub**: 오디오 처리
- **gradio-client**: Chatterbox TTS API
- **python-pptx**: PPTX 파일 처리

---

## 디렉토리 구조

```
ppt-to-video-workflow/
├── SKILL.md                    # 이 문서
└── scripts/
    ├── render_slides.py        # 슬라이드 → 이미지 렌더링
    ├── generate_narration.py   # TTS 나레이션 생성
    ├── generate_subtitles.py   # ASS 자막 생성
    └── create_video.py         # 영상 합성
```

---

## 관련 스킬

- `academic-ppt-generator`: slides.json 생성 및 PPTX 변환
- `chatterbox-tts-workflow`: TTS 생성 (단일 파일)

---

## 예상 결과물

**입력**:
- `slides.json` (제목, 내용, 나레이션)
- `reference_audio.wav` (참조 음성)

**출력**:
- `slide_*.png` (슬라이드 이미지들)
- `narration_*.mp3` (슬라이드별 음성)
- `timing.json` (타이밍 정보)
- `subtitle.ass` (자막 파일)
- `final_video.mp4` (최종 영상)
