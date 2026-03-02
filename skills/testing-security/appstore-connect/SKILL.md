---
name: appstore-connect
description: App Store Connect 자동화 스킬. JWT API/Playwright 하이브리드 방식으로 앱 정보, 빌드, TestFlight 배포, 스크린샷 업로드, 앱 제출 지원. "ASC", "TestFlight", "앱스토어" 키워드로 활성화.
trigger-keywords: App Store Connect, ASC, TestFlight, 앱스토어, 앱 제출, 앱 배포, 스크린샷 업로드, 테스트플라이트, iOS 배포, Apple Developer
allowed-tools: Bash, Read, Edit, Write, Skill
tags: [ios, app-store, testflight, deployment, automation, apple]
---

# App Store Connect Automation Skill

## Overview

App Store Connect API와 Playwright 브라우저 자동화를 결합한 하이브리드 스킬입니다.
API로 가능한 작업은 API를 사용하고, API 미지원 기능은 브라우저 자동화로 처리합니다.

### 핵심 기능

- **앱 정보 관리**: 앱 목록, 버전, 메타데이터 조회 및 수정
- **빌드 관리**: 빌드 상태 조회, 만료 처리
- **TestFlight**: 테스터/그룹 관리, 빌드 배포
- **스크린샷 업로드**: 앱 스크린샷 일괄 업로드
- **앱 제출**: 전체 릴리스 워크플로우 자동화

## When to Use

**명시적 요청:**
- "App Store Connect에서 앱 정보 조회해줘"
- "TestFlight에 빌드 배포해줘"
- "앱 스크린샷 업로드해줘"
- "앱스토어에 제출해줘"

**자동 활성화 키워드:**
- "App Store Connect", "ASC", "앱스토어 커넥트"
- "TestFlight", "테스트플라이트"
- "앱 제출", "앱 배포", "스토어 업로드"
- "스크린샷 업로드", "메타데이터"
- "iOS 배포", "앱스토어 배포"

## 환경변수

이 스킬은 `jelly-dotenv`에서 관리하는 환경변수를 사용합니다.

### 필수 환경변수 (API 인증)

```env
# App Store Connect API (JWT 인증)
APPSTORE_ISSUER_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
APPSTORE_KEY_ID=XXXXXXXXXX
APPSTORE_PRIVATE_KEY_PATH=/path/to/AuthKey_XXXXXXXXXX.p8
```

### 선택 환경변수

```env
# Apple ID (브라우저 세션 인증 - API 미지원 기능용)
APPLE_ID=your_apple_id@example.com

# TestFlight 자동화
TESTFLIGHT_DEFAULT_GROUP_ID=your_default_group_id
AUTO_TESTFLIGHT_DISTRIBUTE=false
```

### API 키 발급 방법

1. [App Store Connect](https://appstoreconnect.apple.com) 로그인
2. Users and Access > Keys > App Store Connect API
3. Generate API Key (Admin 또는 Developer 권한)
4. Issuer ID 복사
5. Key ID 복사
6. AuthKey_*.p8 파일 다운로드 (한 번만 가능!)

## 사전 준비

### 1. 의존성 설치

```bash
cd skills/jelly-appstore-connect
npm install
```

### 2. Playwright 브라우저 설치 (브라우저 기능 사용 시)

```bash
npx playwright install chromium
```

### 3. 환경변수 설정

```bash
# jelly-dotenv/.env 또는 프로젝트 루트 .env에 추가
APPSTORE_ISSUER_ID=your-issuer-id
APPSTORE_KEY_ID=your-key-id
APPSTORE_PRIVATE_KEY_PATH=/path/to/AuthKey.p8
```

---

## 사용 방법

### CLI 명령어

```bash
cd skills/jelly-appstore-connect
npm run asc -- <command> [options]
```

### 인증 명령어

```bash
# JWT API 연결 테스트
npm run asc -- auth test-api

# 브라우저 로그인 (2FA 포함, 최초 1회)
npm run asc -- auth login --headed

# 세션 상태 확인
npm run asc -- auth status

# 로그아웃 (세션 삭제)
npm run asc -- auth logout
```

### 앱 관리

```bash
# 앱 목록 조회
npm run asc -- apps list

# 앱 상세 정보
npm run asc -- apps info <app-id>

# 앱 버전 목록
npm run asc -- apps versions <app-id>
```

### 빌드 관리

```bash
# 빌드 목록
npm run asc -- builds list <app-id>

# 빌드 상세 정보
npm run asc -- builds info <build-id>

# 빌드 처리 완료 대기
npm run asc -- builds wait <build-id> --timeout 600

# 빌드 만료
npm run asc -- builds expire <build-id>
```

### TestFlight

```bash
# 테스터 목록
npm run asc -- testflight testers list <app-id>

# 테스터 초대
npm run asc -- testflight testers invite <app-id> user@example.com

# 베타 그룹 목록
npm run asc -- testflight groups list <app-id>

# 빌드 배포 (그룹에)
npm run asc -- testflight distribute <build-id> --group <group-id>

# 외부 테스터 베타 리뷰 제출
npm run asc -- testflight submit <build-id>
```

### 스크린샷 관리

```bash
# 스크린샷 목록
npm run asc -- screenshots list <app-id> <version-id>

# 스크린샷 업로드 (단일)
npm run asc -- screenshots upload <app-id> <version-id> \
  --locale ko-KR \
  --display iphone_6_7 \
  --file /path/to/screenshot.png

# 스크린샷 일괄 업로드
npm run asc -- screenshots upload-batch <app-id> <version-id> \
  --dir /path/to/screenshots/ \
  --locale ko-KR

# 스크린샷 삭제
npm run asc -- screenshots delete <screenshot-id>
```

### 메타데이터

```bash
# 메타데이터 조회
npm run asc -- metadata get <app-id> --locale ko-KR

# 메타데이터 업데이트
npm run asc -- metadata update <app-id> \
  --locale ko-KR \
  --description "앱 설명" \
  --keywords "키워드1,키워드2"

# 파일에서 메타데이터 업데이트
npm run asc -- metadata update-from-file <app-id> ./metadata.json
```

### 앱 제출

```bash
# 제출 생성
npm run asc -- submit create <version-id>

# 제출 상태 확인
npm run asc -- submit status <submission-id>

# 전체 릴리스 워크플로우
npm run asc -- submit full-release <app-id> \
  --version 1.2.0 \
  --build <build-id> \
  --metadata ./metadata.json \
  --screenshots ./screenshots/
```

### 공통 옵션

```bash
--json          # JSON 형식 출력
--quiet         # 최소 출력
--verbose       # 상세 출력
--dry-run       # 시뮬레이션 (실제 변경 없음)
--timeout <ms>  # 작업 타임아웃
```

---

## 인증 전략

### 1. JWT API 인증 (권장)

대부분의 작업에 사용됩니다. 2FA가 필요 없습니다.

- App Store Connect API Key 사용
- ES256 알고리즘으로 JWT 생성
- 15분 만료, 자동 갱신

**지원 작업:**
- 앱/빌드/버전 정보 조회
- TestFlight 테스터/그룹 관리
- 메타데이터 조회/수정
- 앱 제출

### 2. 브라우저 세션 인증

API가 지원하지 않는 기능에 사용됩니다.

- 최초 1회 수동 로그인 (2FA 포함)
- Playwright storageState로 세션 저장
- 이후 요청에서 저장된 세션 재사용

**지원 작업:**
- 스크린샷 업로드 (API 폴백)
- 일부 고급 설정

### 2FA 처리

1. `npm run asc -- auth login --headed` 실행
2. 브라우저가 열리고 Apple ID 로그인 페이지 표시
3. ID/PW 입력 후 2FA 코드 요청 시 콘솔에 안내 메시지
4. 신뢰된 기기에서 코드 확인 후 브라우저에 입력
5. 로그인 성공 시 세션 자동 저장
6. 이후 요청에서 저장된 세션 사용 (2FA 불필요)

---

## jelly-ios-skill 통합

### 빌드 후 자동 검증

jelly-ios-skill에서 Fastlane 배포 후 빌드 상태를 자동으로 확인할 수 있습니다.

```bash
# iOS 빌드 후 App Store Connect에서 빌드 상태 확인
npm run asc -- builds wait <build-number> --timeout 600

# 빌드 처리 완료 후 자동 TestFlight 배포
npm run asc -- testflight distribute <build-id> --group <group-id>
```

### 환경변수 공유

jelly-ios-skill과 동일한 `APPLE_ID` 환경변수를 사용합니다.

---

## 에러 처리

### 일반적인 에러

**AuthenticationError (401)**
- API 키 확인: APPSTORE_ISSUER_ID, APPSTORE_KEY_ID, APPSTORE_PRIVATE_KEY_PATH
- 키 파일 경로 및 권한 확인

**ForbiddenError (403)**
- API 키 권한 확인 (Admin 또는 Developer)
- 앱 접근 권한 확인

**RateLimitError (429)**
- 잠시 후 재시도 (자동 backoff 적용)

**SessionExpiredError**
- `npm run asc -- auth login --headed`로 재로그인

### 디버깅

```bash
# 상세 로그 출력
npm run asc -- apps list --verbose

# API 응답 확인
npm run asc -- apps info <app-id> --json
```

---

## 디렉토리 구조

```
skills/jelly-appstore-connect/
├── SKILL.md              # 이 문서
├── package.json
├── tsconfig.json
├── bin/
│   └── asc.ts            # CLI 엔트리포인트
├── src/
│   ├── index.ts          # 메인 export
│   ├── types.ts          # 타입 정의
│   ├── auth/             # 인증 모듈
│   │   ├── jwt-auth.ts   # JWT 토큰 관리
│   │   ├── browser-auth.ts # 브라우저 세션 관리
│   │   └── session-store.ts # 세션 저장/복원
│   ├── api/              # REST API 클라이언트
│   │   ├── client.ts     # 기본 클라이언트
│   │   ├── apps.ts       # 앱 API
│   │   ├── builds.ts     # 빌드 API
│   │   ├── testflight.ts # TestFlight API
│   │   └── metadata.ts   # 메타데이터 API
│   ├── browser/          # 브라우저 자동화
│   │   ├── manager.ts    # BrowserManager
│   │   ├── login-flow.ts # 로그인 플로우
│   │   └── screenshots-upload.ts
│   ├── cli/              # CLI
│   │   ├── index.ts      # CLI 라우터
│   │   └── commands/     # 명령어들
│   └── utils/            # 유틸리티
│       ├── errors.ts
│       └── config.ts
├── scripts/              # 독립 실행 스크립트
│   └── full-release.ts
├── data/                 # 세션 데이터 (gitignored)
└── references/           # 참고 문서
```

---

## 참고 자료

- [App Store Connect API Documentation](https://developer.apple.com/documentation/appstoreconnectapi)
- [Generating API Tokens](https://developer.apple.com/documentation/appstoreconnectapi/generating-tokens-for-api-requests)
- [Playwright Authentication](https://playwright.dev/docs/auth)

---

## 제한 사항

- JWT API 키 발급에 Apple Developer Program 멤버십 필요
- 일부 기능은 브라우저 자동화 필요 (macOS 권장)
- 2FA는 최초 1회 수동 처리 필요
- Rate limiting으로 대량 요청 시 지연 발생 가능

---

## Workflow

### Step 1: 환경 설정 확인

```bash
# API 키 설정 확인
npm run asc -- auth test-api
```

### Step 2: 작업 유형에 따른 분기

**앱 정보 조회:**
1. `apps list` → 앱 목록 확인
2. `apps info <app-id>` → 상세 정보

**빌드 관리:**
1. `builds list <app-id>` → 빌드 목록
2. `builds wait <build-id>` → 처리 완료 대기
3. `testflight distribute <build-id>` → TestFlight 배포

**앱 제출:**
1. `metadata update` → 메타데이터 준비
2. `screenshots upload-batch` → 스크린샷 업로드
3. `submit full-release` → 전체 릴리스 실행

---

## Examples

### 예시 1: 앱 목록 조회

```
사용자: "App Store Connect에서 내 앱 목록 보여줘"

Claude:
npm run asc -- apps list

→ 앱 목록:
| 앱 이름 | Bundle ID | 상태 |
|---------|-----------|------|
| MyApp | com.example.myapp | Ready for Sale |
```

### 예시 2: TestFlight 빌드 배포

```
사용자: "최신 빌드를 TestFlight에 배포해줘"

Claude:
1. npm run asc -- builds list <app-id>  # 최신 빌드 확인
2. npm run asc -- testflight groups list <app-id>  # 그룹 확인
3. npm run asc -- testflight distribute <build-id> --group <group-id>

→ 빌드 1.2.3 (build 45)가 "Internal Testers" 그룹에 배포되었습니다.
```

### 예시 3: 앱 제출 워크플로우

```
사용자: "앱 1.3.0 버전을 앱스토어에 제출해줘"

Claude:
npm run asc -- submit full-release <app-id> \
  --version 1.3.0 \
  --build <build-id> \
  --metadata ./metadata.json \
  --screenshots ./screenshots/

→ 앱 제출 완료. 현재 상태: Waiting for Review
```

---

## Best Practices

**DO:**
- API 키 발급 후 .p8 파일 안전하게 보관 (재다운로드 불가)
- `--dry-run` 옵션으로 먼저 시뮬레이션
- 메타데이터 JSON 파일로 버전 관리
- 빌드 처리 완료 후 TestFlight 배포
- 스크린샷 규격 준수 (기기별 해상도)

**DON'T:**
- API 키를 코드에 하드코딩하지 않기
- 2FA 세션 만료 후 자동화 시도하지 않기
- 빌드 처리 중 다른 작업 요청하지 않기
- Rate limit 초과하도록 연속 요청하지 않기
- 동일 빌드를 여러 번 제출하지 않기

---

**Last Updated**: 2025-12 (App Store Connect API 3.4)
