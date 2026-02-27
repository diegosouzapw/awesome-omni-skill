---
name: clarify
description: 모호함 식별 및 명확화 질문 생성
user-invocable: false
---

# Skill: Clarify

## 목적

사용자 요구사항에서 모호한 부분을 식별하고 명확화 질문 생성

---

## 입력

- 사용자 요구사항 (원문)
- 이미 해결된 모호함 목록 (선택)

---

## 출력

- 탐지된 모호함 목록 (우선순위 정렬)
- AskUserQuestion 형식의 질문

---

## 모호함 탐지 카테고리

| 카테고리 | 탐지 패턴 | 질문 예시 |
|----------|----------|----------|
| **범위** | "회원", "주문" 같은 추상적 용어 | "회원은 어떤 정보를 가지나요?" |
| **기능** | "번역", "추천" 같은 동작 | "번역 결과를 저장하나요?" |
| **제약** | "지원", "허용" 같은 제한 | "언어 지원 범위는?" |
| **관계** | "A와 B" 같은 연결 | "회원과 주문의 관계는?" |
| **권한** | "누가", "어떻게" | "프로필 수정은 누가?" |
| **생명주기** | "생성", "삭제" | "탈퇴 시 데이터는?" |

---

## 사용 방법

### 1. 초기 분석

```python
from skills.clarify import identify_ambiguities

ambiguities = identify_ambiguities(
    user_request="외국인 여행자용 번역 앱 만들어줘"
)

# 결과:
# [
#     {
#         "category": "범위",
#         "ambiguity": "회원은 어떤 정보를 가지나요?",
#         "priority": "high"
#     },
#     {
#         "category": "기능",
#         "ambiguity": "번역 결과를 저장하나요?",
#         "priority": "high"
#     },
#     ...
# ]
```

### 2. 질문 생성

```python
from skills.clarify import generate_questions

questions = generate_questions(
    ambiguities=ambiguities[:4],  # 최대 4개
    multiselect_threshold=3  # 3개 이상 옵션이면 multiSelect
)

# AskUserQuestion 형식으로 반환
```

---

## 질문 생성 규칙

### 옵션 수 결정

| 옵션 수 | multiSelect | 이유 |
|---------|-------------|------|
| 2개 | False | 예/아니오 |
| 3-4개 | True | 여러 선택 가능 |
| 5개 이상 | True + "기타" | 확장 가능 |

### 옵션 설명 작성

**좋은 설명**:
- 구체적: "한국어, 영어만" (좋음) vs "기본 언어" (나쁨)
- 결과 명시: "무료 50,000 MAU" (좋음) vs "무료 티어" (나쁨)
- 한글 사용: "일반 소비자" (좋음) vs "B2C" (나쁨)

---

## 기술 질문 필터링

**금지 키워드**:
```python
TECH_KEYWORDS = [
    "FastAPI", "PostgreSQL", "MySQL",
    "JWT", "OAuth", "REST", "GraphQL",
    "S3", "GCS", "Firebase", "Supabase",
    "Docker", "Kubernetes",
    "UUID", "VARCHAR", "INT"
]
```

**검사**:
```python
def is_technical_question(question: str) -> bool:
    """기술 질문인지 확인"""
    lower_q = question.lower()
    for keyword in TECH_KEYWORDS:
        if keyword.lower() in lower_q:
            return True
    return False
```

---

## 예시

### 입력
```
"외국인 여행자용 번역 앱 만들어줘"
```

### 탐지된 모호함
```python
[
    {
        "category": "범위",
        "text": "회원 정보 범위?",
        "priority": "high"
    },
    {
        "category": "기능",
        "text": "번역 히스토리 저장?",
        "priority": "high"
    },
    {
        "category": "제약",
        "text": "언어 지원 범위?",
        "priority": "high"
    },
    {
        "category": "권한",
        "text": "인증 필요 여부?",
        "priority": "medium"
    }
]
```

### 생성된 질문
```python
{
    "questions": [
        {
            "question": "회원 프로필에 어떤 정보가 필요한가요?",
            "header": "회원 정보",
            "multiSelect": True,
            "options": [
                {
                    "label": "기본 (이름, 이메일)",
                    "description": "최소한의 정보만"
                },
                {
                    "label": "프로필 사진",
                    "description": "선택적 이미지 추가"
                },
                {
                    "label": "선호 언어",
                    "description": "자동 언어 전환용"
                }
            ]
        },
        {
            "question": "번역 결과를 저장하나요?",
            "header": "번역 히스토리",
            "multiSelect": False,
            "options": [
                {
                    "label": "영구 저장",
                    "description": "모든 번역 기록 보관"
                },
                {
                    "label": "세션만",
                    "description": "앱 종료 시 삭제"
                },
                {
                    "label": "저장 안 함",
                    "description": "실시간만"
                }
            ]
        }
    ]
}
```
