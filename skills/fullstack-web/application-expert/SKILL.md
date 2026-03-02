---
name: application-expert
description: UseCase 설계, Transaction 경계 관리, CQRS 적용. @Transactional 내 외부 API 호출 금지. /kb-application 명령 시 자동 활성화.
triggers: [/kb-application, /go, /red, /green, /refactor, /tidy, usecase, command, query, transaction, port]
---

# Application Layer 전문가

## 핵심 원칙 (Zero-Tolerance)

### ✅ Mandatory
1. **@Transactional 필수** - Command UseCase에 필수
2. **Port 인터페이스 의존** - SaveOrderPort, LoadOrderPort
3. **Command/Query 분리** - CQRS 패턴
4. **Inner Record DTO** - UseCase 내부에 Command, Response
5. **Domain 로직 위임** - 비즈니스 로직은 Domain에
6. **@Component 필수** - UseCase 구현체는 Spring Bean
7. **Port In/Out 분리** - Input Port (UseCase), Output Port (Adapter)

### ❌ Prohibited
1. **Domain 로직 금지** - UseCase에 비즈니스 로직 진입 금지
2. **@Transactional 내 외부 API 금지** - RestTemplate, WebClient 호출 금지
3. **Entity 직접 반환 금지** - DTO 변환 필수
4. **Service 명명 금지** - XxxService → XxxUseCase
5. **Orchestration 로직 금지** - 복잡한 다중 UseCase 조합 금지
6. **Transaction 중첩 금지** - REQUIRES_NEW 사용 금지
7. **Public DTO 금지** - Command/Response는 UseCase 내부 Record

## 예시

### ✅ CORRECT: UseCase 패턴
```java
@Component
@Transactional
public class PlaceOrderService implements PlaceOrderUseCase {
    private final SaveOrderPort saveOrderPort;

    @Override
    public Response execute(Command command) {
        Order order = Order.create(command.customerId(), ...);
        order.place(); // Domain 비즈니스 메서드
        Order saved = saveOrderPort.save(order);
        return new Response(saved.getOrderIdValue(), ...);
    }

    public record Command(Long customerId, ...) {}
    public record Response(String orderId, ...) {}
}
```

### ❌ WRONG: @Transactional 내 외부 API
```java
@Transactional
public Response execute(Command command) {
    saveOrderPort.save(order);
    // ❌ 금지
    restTemplate.postForEntity("http://external-api/payment", ...);
}
```

## 참조

- **전체 가이드**: [Application Guide](../../../docs/coding_convention/03-application-layer/application-guide.md)
- **상세 규칙 + 템플릿**: [REFERENCE.md](./REFERENCE.md)
- **Transaction 관리**: docs/coding_convention/03-application-layer/transaction-management/

## 자동 활성화

`/kb-application /go|red|green|refactor|tidy` 실행 시 자동 활성화.
