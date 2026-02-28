---
name: cs-guide-writer
description: CS 학습 문서를 작성합니다. "오늘의 CS", "CS 정리", "{주제} 정리해줘", "최근 이슈 CS" 요청 시 사용하세요.
---

# CS 문서 작성 도우미

CS-GUIDE.md 템플릿에 맞춰 학습 문서를 작성하는 Skill입니다.

## 트리거 키워드

- "오늘의 CS: {주제}" - 특정 주제로 문서 작성
- "오늘의 CS" - 주제 미지정 시 옵션 제안 (아래 참고)
- "CS 정리: {주제}"
- "{주제} 정리해줘" (CS 관련 주제인 경우)
- "최근 이슈 CS" - 최신 기술 트렌드 주제

## "오늘의 CS" 주제 미지정 시 처리

주제 없이 "오늘의 CS"만 요청한 경우, **AskUserQuestion 도구**를 사용하여 다음 옵션을 제안:

### 옵션 1: 최신 트렌드 (Recommended)
- 웹 검색으로 2025년 최신 기술 트렌드 조사
- 가장 관련성 높은 주제 1개 선택하여 문서 작성
- 문서 상단에 `Trend` 마크 추가

### 옵션 2: 카테고리 선택
아래 카테고리 중 선택하면 해당 분야의 핵심 주제 추천:

| 대분류 | 세부 분야 | 추천 주제 예시 |
|--------|----------|---------------|
| **Network** | 프로토콜 | HTTP/HTTPS, TCP/IP, UDP, WebSocket, gRPC |
| | 아키텍처 | REST API, GraphQL, CDN, 로드밸런싱, DNS |
| | 보안 | TLS/SSL, CORS, 네트워크 보안 |
| **OS** | 프로세스/스레드 | 프로세스 vs 스레드, 컨텍스트 스위칭, 스케줄링 |
| | 메모리 | 가상 메모리, 페이징, 메모리 단편화, GC |
| | 동기화 | 뮤텍스, 세마포어, 데드락, IPC |
| | 파일 시스템 | 파일 디스크립터, I/O 모델, 버퍼/캐시 |
| **DB** | 기초 | 정규화, 인덱스, 트랜잭션, ACID |
| | 성능 | 쿼리 최적화, 실행 계획, 커넥션 풀 |
| | 분산 | 샤딩, 복제, 파티셔닝, CAP 정리 |
| | NoSQL | Redis, MongoDB, Elasticsearch, 캐싱 전략 |
| **Algorithm** | 기초 | 시간/공간 복잡도, Big-O 표기법 |
| | 자료구조 | Array, LinkedList, Tree, Graph, Hash |
| | 알고리즘 | 정렬, 탐색, DP, 그리디, 백트래킹 |
| **System Design** | 확장성 | 수평/수직 확장, 로드밸런싱, Rate Limiting |
| | 분산 시스템 | 분산 락, 합의 알고리즘, 이벤트 소싱, CDC |
| | 아키텍처 | MSA, 이벤트 드리븐, CQRS, DDD |
| | 메시징 | Kafka, RabbitMQ, 메시지 큐 패턴 |
| **Programming** | 패러다임 | OOP, 함수형 프로그래밍, 리액티브 |
| | 설계 | SOLID, 디자인 패턴, 클린 아키텍처 |
| | 품질 | TDD, 리팩토링, 코드 리뷰, 테스트 전략 |
| | API | REST 설계, API 버저닝, 문서화 |
| **Security** | 인증/인가 | OAuth, JWT, Session, RBAC |
| | 암호화 | 대칭/비대칭 암호화, 해싱, Salt |
| | 취약점 | OWASP Top 10, SQL Injection, XSS, CSRF |
| **AI/Agent** | 기초 | LLM 동작 원리, 프롬프트 엔지니어링 |
| | 도구 | AI Agent, MCP, Function Calling |
| | 응용 | RAG, 벡터 DB, 임베딩 |
| **DevOps** | CI/CD | 파이프라인, 자동화, GitOps |
| | 컨테이너 | Docker, Kubernetes, 서비스 메시 |
| | 모니터링 | 로깅, 메트릭, 트레이싱, APM |
| **Language** | Java | JVM, GC, 동시성, Stream API |
| | Kotlin | 코루틴, Null Safety, 확장 함수 |
| | 일반 | 타입 시스템, 메모리 모델, 컴파일러 |

### 옵션 3: TODO 목록
여러 주제를 한 번에 요청할 때 사용. 작성해야 할 문서를 리스트로 관리:

**사용 예시:**
- "오늘의 CS: 트랜잭션, 인덱스, 정규화 정리해줘"
- "Network 카테고리 기초 문서들 정리해줘"
- "면접 준비용으로 OS 핵심 5개 정리해줘"

**처리 방식:**
1. 요청된 주제들을 TodoWrite 도구로 TODO 목록에 추가
2. 각 주제별로 순차적으로 문서 작성
3. 완료된 항목은 completed로 표시
4. 전체 진행 상황을 사용자에게 표시

### 질문 형식 예시

```
오늘의 CS 주제를 어떻게 선택할까요?

1. 최신 트렌드 (Recommended) - 2025년 기술 트렌드에서 주제 선정
2. 카테고리 선택 - 관심 분야에서 핵심 주제 추천
3. TODO 목록 - 여러 주제를 리스트로 작성 (예: "트랜잭션, 인덱스, 정규화")
```

## 카테고리 판단 기준

| 카테고리 | 세부 분류 | 주제 예시 |
|----------|----------|----------|
| `network/` | 프로토콜 | HTTP, HTTPS, TCP/IP, UDP, WebSocket, gRPC |
| | 아키텍처 | REST API, GraphQL, CDN, 로드밸런싱, DNS |
| | 보안 | TLS/SSL, CORS, 방화벽, VPN |
| `os/` | 프로세스/스레드 | 컨텍스트 스위칭, 스케줄링, PCB, TCB |
| | 메모리 | 가상 메모리, 페이징, 세그멘테이션, GC |
| | 동기화 | 뮤텍스, 세마포어, 데드락, 조건 변수 |
| | 파일 시스템 | 파일 디스크립터, I/O 모델, 버퍼/캐시 |
| | IPC | 파이프, 소켓, 공유 메모리, 메시지 큐 |
| `db/` | 기초 | 정규화, 인덱스, 트랜잭션, ACID |
| | 성능 | 쿼리 최적화, 실행 계획, 커넥션 풀, N+1 |
| | 분산 | 샤딩, 복제, 파티셔닝, CAP, PACELC |
| | NoSQL | Redis, MongoDB, Elasticsearch, 캐싱 전략 |
| `algorithm/` | 기초 | 시간/공간 복잡도, Big-O, 점근 표기법 |
| | 자료구조 | Array, LinkedList, Tree, Graph, Hash |
| | 알고리즘 | 정렬, 탐색, DP, 그리디, 백트래킹, 분할정복 |
| `data-structure/` | 선형 | Array, LinkedList, Stack, Queue, Deque |
| | 비선형 | Tree, Graph, Heap, Trie |
| | 해시 | HashMap, HashSet, 충돌 해결 |
| `system-design/` | 확장성 | 수평/수직 확장, 로드밸런싱, Rate Limiting |
| | 분산 시스템 | 분산 락, 합의 알고리즘, 이벤트 소싱, CDC |
| | 아키텍처 | MSA, 이벤트 드리븐, CQRS, DDD, 헥사고날 |
| | 메시징 | Kafka, RabbitMQ, 메시지 큐 패턴, Pub/Sub |
| `git/` | 기초 | 버전 관리, 커밋, 브랜치, 머지 |
| | 전략 | Git Flow, GitHub Flow, Trunk-Based |
| | 내부 | Git 객체, 참조, reflog |
| `language/` | Java | JVM, GC, 동시성, Stream, 메모리 모델 |
| | Kotlin | 코루틴, Null Safety, 확장 함수, DSL |
| | 일반 | 타입 시스템, 컴파일러, 인터프리터 |
| `programming/` | 패러다임 | OOP, 함수형 프로그래밍, 리액티브 |
| | 설계 | SOLID, 디자인 패턴, 클린 아키텍처 |
| | 품질 | TDD, 리팩토링, 코드 리뷰, 정적 분석 |
| | API | REST 설계, API 버저닝, OpenAPI, 문서화 |
| `security/` | 인증/인가 | OAuth 2.0, JWT, Session, RBAC, ABAC |
| | 암호화 | 대칭/비대칭, 해싱, Salt, HTTPS |
| | 취약점 | OWASP Top 10, SQL Injection, XSS, CSRF |
| | 보안 설계 | 제로 트러스트, 시크릿 관리, 보안 감사 |
| `ai-agent/` | 기초 | LLM 동작 원리, 토큰화, 프롬프트 엔지니어링 |
| | 도구 | AI Agent, MCP, Function Calling, Tool Use |
| | 응용 | RAG, 벡터 DB, 임베딩, Fine-tuning |
| `devops/` | CI/CD | 파이프라인, GitHub Actions, GitOps |
| | 컨테이너 | Docker, Kubernetes, Helm, 서비스 메시 |
| | 모니터링 | 로깅, 메트릭, 트레이싱, APM, SRE |
| | 인프라 | IaC, Terraform, 클라우드 서비스 |
| `trend/` | - | 최신 기술 트렌드, 연도별 이슈 |

## 난이도 체계

난이도는 **3개로 고정하지 않고 유연하게 세분화**할 수 있습니다.

**핵심 원칙:**
1. **카테고리 정의가 최우선** - 각 카테고리는 "~란?" 정의 문서로 시작
2. **선수 지식 기반 순서** - 선수 지식이 없는 것 → 있는 것 순서

| 레벨 | 설명 | 표기 예시 |
|------|------|----------|
| `[1] 정의/기초` | 카테고리 정의, 선수 지식 없음 | `> \`[1] 정의\` · 선수 지식 없음` |
| `[2] 입문` | 기본 개념, 간단한 선수 지식 | `> \`[2] 입문\` · 선수 지식: 없음` |
| `[3] 중급` | 실무 핵심, 여러 기초 지식 | `> \`[3] 중급\` · 선수 지식: [문서명](./path.md)` |
| `[4] 심화` | 복잡한 개념, 다중 선수 지식 | `> \`[4] 심화\` · 선수 지식: [문서1](./path.md), [문서2](./path.md)` |
| `[5+]` | 필요 시 추가 가능 | - |

## 문서 작성 템플릿

```markdown
# {주제명}

> `[N] {레벨명}` · 선수 지식: [선수 문서](./path.md) 또는 "없음"

> `Trend` {연도}  ← 트렌드 주제인 경우에만 추가

> 한 줄 정의 (명확하고 간결하게)

`#핵심한글` `#CoreEnglish` `#약어` `#FullName` `#관련기술1` `#관련기술2` `#하위개념` `#사용사례` `#면접키워드` ... (최대 30개)

## 왜 알아야 하는가?

{이 개념을 왜 배워야 하는지}

- **실무**: {실무에서 어떻게 활용되는지}
- **면접**: {면접에서 왜 자주 나오는지}
- **기반 지식**: {이 개념이 다른 개념의 기반이 되는 이유}

## 핵심 개념

- {핵심 1}
- {핵심 2}
- {핵심 3}

## 쉽게 이해하기

{실생활 비유로 개념 설명}

## 상세 설명

### {소주제 1}

{내용}

**왜 이렇게 하는가?**
{이유 설명}

## 동작 원리 (해당 시)

{Mermaid 다이어그램으로 시각화 - mermaid-diagram 스킬 참조}

![{다이어그램 설명}](./images/{diagram-name}.svg)

## 예제 코드 (해당 시)

```java
// 예제 코드
```

## 코드 블록 언어 가이드

웹 뷰어(cs-web)의 Highlight.js 호환성을 위해 아래 언어를 사용합니다.

### 권장 언어 태그

| 언어 | 태그 | 비고 |
|------|------|------|
| Java | `java` | |
| Kotlin | `kotlin` | |
| JavaScript | `javascript` 또는 `js` | |
| TypeScript | `typescript` 또는 `ts` | |
| Python | `python` | |
| SQL | `sql` | |
| JSON | `json` | |
| YAML | `yaml` | |
| Markdown | `markdown` | |
| **쉘/터미널** | `shell` 또는 `bash` | ⚠️ `powershell` 사용 금지 |
| HTML | `html` | |
| CSS | `css` | |
| Go | `go` | |
| Rust | `rust` | |

### 주의사항

- **`powershell` 사용 금지**: Highlight.js 기본 빌드에 포함되지 않아 웹에서 하이라이팅 안됨
- Windows 명령어도 `shell` 또는 `bash`로 표기
- 언어 미지정 시 하이라이팅 없이 표시됨

## 트레이드오프 (해당 시)

| 장점 | 단점 |
|------|------|
| {장점} | {단점} |

## 트러블슈팅 (해당 시)

### 사례 1: {증상/문제 요약}

#### 증상
{에러 메시지, 로그, 현상}

#### 원인 분석
{왜 발생했는지 - 기술적 근거 포함}

#### 해결 방법
{구체적인 명령어/코드와 함께 해결법 제시}

#### 예방 조치
{모니터링, 설정 변경 등 재발 방지 방법}

## 면접 예상 질문

### Q: {질문}

A: {답변 - "왜?"에 대한 설명 포함}

## 연관 문서

| 문서 | 연관성 | 난이도 |
|------|--------|--------|
| [선수 문서](./path.md) | 선수 지식 | Beginner |
| [관련 문서](./path.md) | 관련 개념 | Intermediate |
| [후속 문서](./path.md) | 심화 학습 | Advanced |

## 참고 자료

- {출처}
```

## 키워드 작성 가이드

문서 상단에 검색용 키워드를 작성합니다. 나중에 검색 시 해당 문서를 쉽게 찾을 수 있도록 합니다.

### 키워드 작성 원칙

1. **최대 30개**까지 작성 가능 (내용에 맞게 충분히 작성)
2. **한글/영문** 모두 포함 (검색 편의)
3. **약어와 전체 명칭** 모두 포함
4. **관련 기술/도구/서비스/프레임워크** 포함
5. **동의어/유사어/대체 용어** 포함
6. **사용 사례/시나리오** 관련 키워드 포함
7. **면접 키워드** 포함 (자주 나오는 질문 관련)

### 키워드 분류 체계

키워드는 다음 카테고리별로 작성합니다:

| 분류 | 설명 | 예시 |
|------|------|------|
| **핵심 개념** | 주제의 핵심 용어 (한글/영문) | `#트랜잭션` `#Transaction` |
| **약어/전체명** | 약어와 풀네임 | `#MSA` `#마이크로서비스` `#Microservices` |
| **관련 기술** | 함께 사용되는 기술/도구 | `#Spring` `#JPA` `#Hibernate` |
| **하위 개념** | 세부 개념/종류 | `#ReadCommitted` `#Serializable` |
| **사용 사례** | 실제 적용 시나리오 | `#분산락` `#동시성제어` `#데드락` |
| **비교 대상** | 대안/경쟁 기술 | `#MySQL` `#PostgreSQL` `#Oracle` |
| **면접 키워드** | 면접 빈출 주제 | `#ACID` `#격리수준` `#락` |

### 카테고리별 키워드 예시 (확장)

| 카테고리 | 주제 | 키워드 (최대 30개) |
|----------|------|-------------------|
| **AI/Agent** | AI Agent | `#AI에이전트` `#AIAgent` `#LLM` `#대규모언어모델` `#Claude` `#GPT` `#Gemini` `#ChatGPT` `#Anthropic` `#OpenAI` `#Google` `#자율AI` `#Agentic` `#ToolUse` `#FunctionCalling` `#ReAct` `#CoT` `#ChainOfThought` `#프롬프트` `#RAG` `#벡터DB` `#임베딩` `#파인튜닝` `#토큰` `#컨텍스트윈도우` |
| **Network** | WebSocket | `#웹소켓` `#WebSocket` `#WS` `#WSS` `#실시간통신` `#양방향통신` `#FullDuplex` `#Socket.io` `#STOMP` `#SockJS` `#채팅` `#푸시알림` `#게임서버` `#Polling` `#LongPolling` `#SSE` `#ServerSentEvents` `#HTTP업그레이드` `#핸드셰이크` `#연결유지` `#Stateful` `#Spring` `#Netty` |
| **DB** | 트랜잭션 | `#트랜잭션` `#Transaction` `#ACID` `#원자성` `#Atomicity` `#일관성` `#Consistency` `#격리성` `#Isolation` `#지속성` `#Durability` `#커밋` `#Commit` `#롤백` `#Rollback` `#격리수준` `#IsolationLevel` `#ReadUncommitted` `#ReadCommitted` `#RepeatableRead` `#Serializable` `#락` `#Lock` `#데드락` `#Deadlock` `#동시성` `#MVCC` |
| **OS** | 프로세스 | `#프로세스` `#Process` `#스레드` `#Thread` `#PCB` `#ProcessControlBlock` `#컨텍스트스위칭` `#ContextSwitch` `#스케줄링` `#Scheduling` `#fork` `#exec` `#IPC` `#프로세스간통신` `#공유메모리` `#파이프` `#소켓` `#메시지큐` `#좀비프로세스` `#고아프로세스` `#멀티프로세스` `#멀티스레드` |
| **System Design** | MSA | `#MSA` `#마이크로서비스` `#Microservices` `#분산시스템` `#DistributedSystem` `#API게이트웨이` `#APIGateway` `#서비스메시` `#ServiceMesh` `#Istio` `#Envoy` `#서비스디스커버리` `#로드밸런싱` `#서킷브레이커` `#Resilience4j` `#Saga패턴` `#CQRS` `#이벤트소싱` `#EventDriven` `#Kafka` `#RabbitMQ` `#Docker` `#Kubernetes` `#컨테이너` `#오케스트레이션` |
| **Programming** | OOP | `#OOP` `#객체지향` `#ObjectOriented` `#캡슐화` `#Encapsulation` `#상속` `#Inheritance` `#다형성` `#Polymorphism` `#추상화` `#Abstraction` `#SOLID` `#SRP` `#OCP` `#LSP` `#ISP` `#DIP` `#클래스` `#Class` `#인터페이스` `#Interface` `#추상클래스` `#오버라이딩` `#오버로딩` `#디자인패턴` `#의존성주입` `#DI` |
| **Security** | JWT | `#JWT` `#JsonWebToken` `#토큰` `#Token` `#인증` `#Authentication` `#인가` `#Authorization` `#AccessToken` `#RefreshToken` `#Bearer` `#Claim` `#Payload` `#Header` `#Signature` `#HS256` `#RS256` `#만료시간` `#토큰탈취` `#토큰무효화` `#블랙리스트` `#OAuth` `#OIDC` `#세션` `#Stateless` |

### 키워드 포맷

```markdown
`#핵심한글` `#CoreEnglish` `#약어` `#관련기술1` `#관련기술2` `#하위개념1` `#하위개념2` `#사용사례` `#비교대상` `#면접키워드`
```

### 키워드 개수 가이드

| 문서 복잡도 | 권장 키워드 수 |
|------------|---------------|
| 기초 개념 (정의) | 10~15개 |
| 중급 개념 | 15~20개 |
| 심화 개념 | 20~25개 |
| 복합 주제 | 25~30개 |

## 작업 절차

1. **카테고리 판단**: 주제에 맞는 카테고리 선택
2. **난이도 판단**: [1] 정의/기초 → [2] 입문 → [3] 중급 → [4] 심화 중 선택
   - 카테고리 정의 문서("~란?")는 반드시 [1]
   - 선수 지식이 없으면 낮은 레벨, 있으면 높은 레벨
3. **파일 생성**: `cs/{category}/{topic}.md` (kebab-case)
4. **내용 작성**: 위 템플릿 준수
   - **정의**: 한 문장으로 명확하게
   - **키워드**: 검색용 키워드 10~30개 (한글/영문, 약어, 관련 기술, 하위 개념, 면접 키워드)
   - **왜 알아야 하는가**: 실무/면접/기반지식 관점
   - **난이도 태그 + 연관 문서** 필수
5. **README 업데이트**: `cs/{category}/README.md`의 해당 레벨 섹션에 링크 추가
6. **git add**: 생성/수정된 파일 staging

### README 업데이트 시 확인사항

- 해당 레벨 섹션([1], [2], [3], [4])에 문서 추가
- 선수 지식이 있다면 명시
- 전체 목차에도 추가
- 정의 문서가 없으면 TODO로 추가

## 품질 기준

| 항목 | 기준 |
|------|------|
| 정확성 | 공식 문서, 신뢰할 수 있는 출처 기반 |
| 이해 용이성 | "쉽게 이해하기" 섹션 필수 포함 |
| 논리성 | 모든 설명에 "왜?"에 대한 답변 포함 |
| 실용성 | 면접 대비 + 실무 적용 가능한 내용 |
| 트러블슈팅 | 실무 문제 사례와 해결법 포함 (해당 시) |

### 트러블슈팅 품질 기준

| 조건 | 설명 |
|------|------|
| 재현 가능 | 같은 환경에서 같은 문제를 재현할 수 있어야 함 |
| 구체적 수치 | 에러 메시지, 로그, 메트릭 등 구체적 데이터 포함 |
| 원인 분석 | "왜 발생했는지" 기술적 분석 필수 |
| 다중 해결책 | 가능하다면 여러 해결 방법 제시 |
| 예방 조치 | 재발 방지를 위한 모니터링, 설정 가이드 |

## 최근 이슈 CS 처리

"최근 이슈 CS" 요청 시:

1. 웹 검색으로 최신 기술 트렌드 조사
2. 가장 관련성 높은 주제 1개 선택
3. 적절한 카테고리에 문서 생성 (주로 `ai-agent/` 또는 주제에 맞는 카테고리)
4. 문서 상단에 `Trend` 마크 추가
5. `cs/trend/README.md`에도 링크 추가 (트렌드 인덱스)

## 다이어그램 가이드

**ASCII 대신 SVG 다이어그램 사용**

시각적 설명이 필요한 경우 ASCII 다이어그램 대신 SVG를 생성합니다. 두 가지 방식을 상황에 맞게 선택합니다.

### 방식 선택 가이드

| 상황 | 권장 방식 | 이유 |
|------|----------|------|
| **플로우차트, 시퀀스** | Mermaid | 자동 레이아웃, 빠른 작성 |
| **ER/클래스 다이어그램** | Mermaid | UML 문법 지원 |
| **상태 다이어그램** | Mermaid | 상태 전이 표현 용이 |
| **마인드맵, Git 그래프** | Mermaid | 전용 문법 지원 |
| **계층 구조 (OSI 등)** | SVG 직접 | 정교한 테이블 레이아웃 |
| **네트워크 토폴로지** | SVG 직접 | 정확한 배치 제어 |
| **패킷/프레임 구조** | SVG 직접 | 데이터 구조 시각화 |
| **비교 다이어그램** | SVG 직접 | 좌우/전후 비교 레이아웃 |
| **정교한 레이아웃 필요** | SVG 직접 | 픽셀 단위 제어 가능 |

### 다이어그램 적용 섹션

| 섹션 | 다이어그램 유형 | 권장 방식 |
|------|----------------|----------|
| 핵심 개념 | Mindmap | Mermaid |
| 상세 설명 | Flowchart / 구조도 | Mermaid 또는 SVG |
| 동작 원리 | Sequence / 상태 전환 | Mermaid |
| 계층/구조 | 테이블 / 토폴로지 | SVG 직접 |

### 방식 1: Mermaid CLI (mermaid-diagram 스킬)

**적합한 경우:** 플로우차트, 시퀀스, 상태 다이어그램, 마인드맵 등

```bash
# 1. images 디렉토리 생성
mkdir -p cs/{category}/images

# 2. .mmd 파일 작성

# 3. SVG 생성
npx -p @mermaid-js/mermaid-cli mmdc \
  -i cs/{category}/images/{name}.mmd \
  -o cs/{category}/images/{name}.svg \
  -b transparent

# 4. git add
git add cs/{category}/images/
```

**지원 다이어그램:** Flowchart, Sequence, Class, State, ER, Gantt, Mindmap, GitGraph, Timeline, C4 등

자세한 문법과 템플릿은 `mermaid-diagram` 스킬 참조.

### 방식 2: SVG 직접 생성 (svg-diagram 스킬)

**적합한 경우:** 계층 구조, 비교표, 정교한 레이아웃, 데이터 구조

```bash
# 1. images 디렉토리 생성
mkdir -p cs/{category}/images

# 2. SVG 파일 직접 작성
# cs/{category}/images/{name}.svg

# 3. git add
git add cs/{category}/images/
```

**색상 팔레트 (필수 준수):**

| 용도 | 시작색 | 끝색 |
|------|--------|------|
| 파랑 (기본) | #3498DB | #2980B9 |
| 초록 (성공) | #2ECC71 | #27AE60 |
| 빨강 (경고) | #E74C3C | #C0392B |
| 주황 | #E67E22 | #D35400 |
| 보라 | #9B59B6 | #8E44AD |

자세한 템플릿과 스타일 가이드는 `svg-diagram` 스킬 참조.

### 마크다운 삽입

```markdown
![{설명}](./images/{name}.svg)
```

### 결정 플로우

1. **복잡한 논리 흐름, 분기가 많음** → Mermaid Flowchart
2. **API/시스템 간 통신** → Mermaid Sequence
3. **DB 스키마, 클래스 관계** → Mermaid ER/Class
4. **개념 정리, 브레인스토밍** → Mermaid Mindmap
5. **OSI 7계층 같은 테이블 형태** → SVG 직접
6. **네트워크 토폴로지, 아키텍처** → SVG 직접 (정교) 또는 Mermaid (빠름)
7. **정확한 위치/크기 제어 필요** → SVG 직접
