---
name: nestjs-setup-guide
description: "NestJS 프로젝트 아키텍처, 기술 스택 선택, 모듈 구조에 대한 질문이나 계획 수립 시 자동으로 가이드 제공"
---

# NestJS 프로젝트 설계 가이드

## 자동 활성화 시점
- NestJS 프로젝트 구조나 아키텍처 질문
- "어떤 기술 스택을 사용해야 하나요?"
- "모듈 구조는 어떻게 구성하나요?"
- Modular 아키텍처 설명 필요 시

---

## 기술 스택 권장사항

### Core Framework
- **NestJS** (최신 stable 버전)
- **TypeScript** (strict mode)
- **Node.js** (LTS 버전)

### 데이터베이스
- **TypeORM** 또는 **Prisma** (ORM)
- **PostgreSQL**, **MySQL**, 또는 **MongoDB**

### 검증
- **class-validator** (DTO 검증)
- **class-transformer** (데이터 변환)

### 코드 품질 도구 선택 가이드

| 옵션 | 도구 | 적합한 상황 |
|------|------|-------------|
| A | **Biome** (통합 도구) | 신규 프로젝트, 빠른 설정, 대규모 코드베이스 |
| B | **ESLint + Prettier** | 기존 팀 표준 유지, NestJS 플러그인 필요, 세밀한 규칙 커스터마이징 |

### 테스팅
- **Jest** (단위/통합 테스트 - NestJS 기본)
- **Supertest** (E2E 테스트)

### 환경 관리
- **@nestjs/config** (환경 변수 관리)
- **fnm** 또는 **nvm** (Node.js 버전 관리)
- **pnpm** (패키지 매니저)

---

## Modular 아키텍처 패턴

### 디렉토리 구조

확장 가능하고 유지보수하기 쉬운 **Modular 아키텍처**를 기본으로 사용합니다.

```
src/
├── main.ts                 # 애플리케이션 진입점
├── app.module.ts           # 루트 모듈
├── modules/                # 기능별 모듈
│   ├── auth/              # 인증 모듈
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── dto/           # 데이터 전송 객체
│   │   ├── entities/      # 데이터베이스 엔티티
│   │   ├── guards/        # 가드 (인증/인가)
│   │   └── strategies/    # Passport 전략
│   └── ...
├── shared/                 # 공유 리소스
│   ├── decorators/        # 커스텀 데코레이터
│   ├── filters/           # 예외 필터
│   ├── interceptors/      # 인터셉터
│   ├── pipes/             # 파이프
│   ├── guards/            # 공용 가드
│   ├── utils/             # 유틸리티 함수
│   └── types/             # 공용 타입
├── config/                 # 설정 파일
│   ├── database.config.ts
│   └── app.config.ts
└── __tests__/             # E2E 테스트
```

### Modular 구조 원칙

1. **모듈 중심 설계**
   - 각 기능은 독립적인 모듈로 구성
   - 모듈은 자체 완결성을 유지
   - 모듈 간 의존성은 imports를 통해 명시

2. **계층 분리**
   - Controller: HTTP 요청 처리
   - Service: 비즈니스 로직
   - Repository/Entity: 데이터 접근

3. **의존성 주입**
   - NestJS DI 컨테이너 활용
   - Interface 기반 설계 권장
   - Provider를 통한 느슨한 결합

4. **공유 규칙**
   - 모듈 간 공유는 `shared/`를 통해서만
   - 공통 기능은 독립 모듈로 분리 (예: CommonModule)

---

## DTO 및 검증

### 핵심 원칙

모든 외부 입력은 **class-validator**를 통해 검증합니다.

### DTO 작성 규칙

```typescript
// ✅ 올바른 DTO
import { IsString, IsEmail, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;

  @IsString()
  name: string;
}
```

### ValidationPipe 설정

```typescript
// main.ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,        // DTO에 없는 속성 제거
    forbidNonWhitelisted: true, // 허용되지 않은 속성 있으면 에러
    transform: true,        // 타입 자동 변환
  })
);
```

---

## 예외 처리

### 표준 예외 사용

```typescript
// ✅ NestJS 표준 예외 사용
import { NotFoundException, BadRequestException } from '@nestjs/common';

@Injectable()
export class UsersService {
  async findOne(id: string): Promise<User> {
    const user = await this.usersRepository.findOne(id);
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    return user;
  }
}
```

### 커스텀 예외 필터

```typescript
// shared/filters/http-exception.filter.ts
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      message: exception.message,
    });
  }
}
```

---

## 환경 설정

### @nestjs/config 사용

```typescript
// app.module.ts
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: `.env.${process.env.NODE_ENV}`,
    }),
  ],
})
export class AppModule {}
```

### 설정 파일

```typescript
// config/database.config.ts
export default () => ({
  database: {
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT, 10) || 5432,
    username: process.env.DB_USERNAME,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
  },
});
```

---

## CLAUDE.md 템플릿

프로젝트 루트에 `CLAUDE.md` 파일을 생성하여 AI 어시스턴트에게 프로젝트 규칙을 전달합니다:

```markdown
# 프로젝트 개발 가이드

## 아키텍처
- Modular 구조 사용
- 각 기능은 독립적인 모듈로 구성
- 모듈 간 의존성은 imports를 통해 명시

## 계층 분리
- Controller: HTTP 요청 처리만
- Service: 비즈니스 로직 구현
- Repository/Entity: 데이터 접근

## 코드 품질

### 필수 준수
- 모든 DTO에 class-validator 적용
- 모든 엔티티에 명시적 타입 지정
- 에러 처리는 NestJS 표준 예외 사용
- 환경 변수는 @nestjs/config를 통해서만 접근

### 절대 금지
- Service에서 직접 Request 객체 사용
- DTO 없이 컨트롤러 파라미터 받기
- 하드코딩된 환경 변수 (process.env 직접 접근)

## 테스팅
- 모든 Service는 단위 테스트 작성
- E2E 테스트는 주요 API 엔드포인트에 대해 작성
- 테스트 파일은 `*.spec.ts` (단위), `*.e2e-spec.ts` (E2E)
```

---

## 설정 요구사항

### 버전 고정
- `.nvmrc` 또는 `.node-version` 파일 생성
- `package.json`에 `engines` 필드 명시

### 필수 스크립트
```json
{
  "scripts": {
    "start:dev": "nest start --watch",
    "start:prod": "node dist/main",
    "build": "nest build",
    "lint": "코드 린팅",
    "format": "코드 포맷팅",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  }
}
```

---

## 참고사항

이 skill은 **계획 및 설계 단계**에서 활성화됩니다. 실제 프로젝트 초기화 실행이 필요하면 자연스럽게 대화를 이어가세요.
