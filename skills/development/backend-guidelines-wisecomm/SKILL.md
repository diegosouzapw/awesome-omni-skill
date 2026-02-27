---
name: backend-guidelines
description: Core architectural rules, coding conventions, and tech stack guidelines for the Dog-Love Backend. Reference this for all backend code changes.
---

# Dog-Love Backend — Agent Instructions

## Tech Stack

- **Java 21** (LTS)
- **Spring Boot 3.5.10**
- **Gradle** (Groovy DSL)
- **MyBatis 3** (w/ PageHelper) — *No JPA*
- **Database**: PostgreSQL + Flyway (Migration)
- **Auth**: JWT (jjwt 0.12.6) + Spring Security
- **Utils**: MapStruct, Lombok, Jasypt (Encryption), Swagger (SpringDoc), Apache POI (Excel)
- **Logging**: Log4j2 (Logback Excluded)

## Commands

```bash
./gradlew build          # 빌드 (테스트 포함)
./gradlew build -x test  # 빌드 (테스트 제외, 빠른 검증)
./gradlew test           # 유닛 테스트 실행
./gradlew bootRun        # 로컬 서버 실행
```

## 코드 수정 후 필수 검증 절차

코드를 수정한 후 반드시 다음 순서로 검증할 것:

1. `./gradlew build -x test` — 컴파일 오류 없음 확인
2. `./gradlew test` — 유닛 테스트 통과 확인

## 커밋 규칙

- **커밋 시 해당 스크립트 사용 권장** (스코프 제한 커밋):

```bash
./backend/scripts/committer.sh "feat: add user api" src/main/java/com/example/springrest/domain/user/
```

- committer가 자동으로 수행하는 것:
  1. conventional commit 메시지 형식 검증
  2. 지정 파일만 스테이징
  3. build + test 검증 (실패 시 차단)

## 코딩 컨벤션

- **Domain-Driven Packaging**: `com.example.springrest.domain.{feature}`
  - 예: `domain/user`, `domain/auth`
- **MyBatis Mapper**:
  - Interface: `Mapper.java` (@Mapper)
  - XML: `src/main/resources/mapper` (namespace 일치 필수)
- **DTO 필수**: Entity/VO 직접 반환 금지. `MapStruct` 사용하여 변환.
- **Logging**: `@Slf4j` 대신 **Log4j2** 사용 권장 (설정 확인 필요)
- **API Spec**: Controller에 Swagger Annotation (`@Tag`, `@Operation`) 필수

## 디렉토리 구조

```
src/main/java/com/example/springrest/
├── Teacher.java (Main)
├── common/                  # 공통 유틸/상수
├── global/                  # 전역 설정 (Config, Exception, Security)
└── domain/                  # 비즈니스 도메인 (기능별)
    ├── auth/
    │   ├── controller/
    │   ├── service/
    │   └── mapper/
    └── user/
        ├── controller/
        ├── service/
        ├── mapper/      # MyBatis Interface
        ├── dto/
        └── vo/          # DB Value Object
```

## 주의사항

- **Lombok + MapStruct**: 빌드 시 Annotation Processor 순서 주의 (`build.gradle` 참조)
- **MyBatis XML**: 쿼리 수정 시 `resultMap` 매핑 주의
- **Flyway**: DB 스키마 변경 시 `V{version}__{description}.sql` 추가 필수
