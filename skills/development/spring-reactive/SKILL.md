---
name: spring-reactive
description: Build reactive applications - WebFlux, Mono/Flux, R2DBC, backpressure, reactive streams
sasmp_version: "1.3.0"
bonded_agent: 07-spring-reactive
bond_type: PRIMARY_BOND
version: "2.0.0"
updated: "2024-12-30"
---

# Spring Reactive Skill

Master reactive programming with Spring WebFlux, Project Reactor, R2DBC, and reactive streams patterns.

## Overview

This skill covers building non-blocking, reactive applications with Spring WebFlux and Project Reactor.

## Parameters

| Name | Type | Required | Default | Validation |
|------|------|----------|---------|------------|
| `reactive_db` | enum | ✗ | r2dbc-postgresql | r2dbc-postgresql \| r2dbc-mysql \| mongodb |
| `streaming` | enum | ✗ | - | sse \| websocket \| rsocket |
| `backpressure` | enum | ✗ | buffer | buffer \| drop \| latest |

## Topics Covered

### Core (Must Know)
- **WebFlux**: `@RestController` with `Mono<T>` and `Flux<T>`
- **Project Reactor**: Core operators (map, flatMap, filter)
- **R2DBC**: Reactive database access

### Intermediate
- **Error Handling**: onErrorResume, onErrorReturn
- **Backpressure**: Handling fast producers
- **SSE**: Server-Sent Events

### Advanced
- **WebSocket**: Reactive WebSocket handlers
- **RSocket**: Bi-directional reactive streams
- **Context Propagation**: MDC in reactive chains

## Code Examples

### Reactive Controller
```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public Flux<UserResponse> findAll() {
        return userService.findAll().map(UserResponse::from);
    }

    @GetMapping("/{id}")
    public Mono<ResponseEntity<UserResponse>> findById(@PathVariable Long id) {
        return userService.findById(id)
            .map(UserResponse::from)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<UserResponse> create(@Valid @RequestBody Mono<CreateUserRequest> request) {
        return request.flatMap(userService::create).map(UserResponse::from);
    }
}
```

### Reactive Repository (R2DBC)
```java
public interface UserRepository extends ReactiveCrudRepository<User, Long> {

    Mono<User> findByEmail(String email);

    Flux<User> findByActiveTrue();

    @Query("SELECT * FROM users WHERE created_at > :since")
    Flux<User> findRecentUsers(@Param("since") LocalDateTime since);
}
```

### Reactive Service with Error Handling
```java
@Service
@RequiredArgsConstructor
@Transactional
public class UserService {

    private final UserRepository userRepository;

    public Mono<User> create(CreateUserRequest request) {
        return userRepository.findByEmail(request.email())
            .flatMap(existing -> Mono.<User>error(new DuplicateEmailException()))
            .switchIfEmpty(Mono.defer(() -> {
                User user = new User(request.email(), request.name());
                return userRepository.save(user);
            }));
    }

    public Flux<User> findAll() {
        return userRepository.findAll()
            .timeout(Duration.ofSeconds(5))
            .onErrorResume(TimeoutException.class, e -> Flux.empty());
    }
}
```

### Server-Sent Events
```java
@RestController
@RequestMapping("/api/events")
public class EventController {

    private final Sinks.Many<Event> eventSink = Sinks.many()
        .multicast().onBackpressureBuffer();

    @GetMapping(produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<Event>> stream() {
        return eventSink.asFlux()
            .map(e -> ServerSentEvent.<Event>builder()
                .id(e.id())
                .event(e.type())
                .data(e)
                .build());
    }

    @PostMapping
    public Mono<Void> publish(@RequestBody Event event) {
        return Mono.fromRunnable(() -> eventSink.tryEmitNext(event));
    }
}
```

## Operator Quick Reference

```
Transformation: map(), flatMap(), flatMapMany()
Filtering:      filter(), take(), skip(), distinct()
Combination:    merge(), concat(), zip()
Error:          onErrorResume(), onErrorReturn(), retry(), timeout()
Side Effects:   doOnNext(), doOnError(), doFinally(), log()
```

## Troubleshooting

### Failure Modes

| Issue | Diagnosis | Fix |
|-------|-----------|-----|
| Nothing happens | Not subscribed | Return Mono/Flux from controller |
| Blocking error | Blocking in reactive | Use `subscribeOn(Schedulers.boundedElastic())` |
| Memory issues | Unbounded buffer | Add backpressure strategy |

### Debug Checklist

```
□ Verify Mono/Flux is returned (not subscribed manually)
□ Check for blocking calls (JDBC, Thread.sleep)
□ Review backpressure strategy
□ Enable Reactor debug: Hooks.onOperatorDebug()
□ Use .log() operator for debugging
```

## Unit Test Template

```java
@WebFluxTest(UserController.class)
class UserControllerTest {

    @Autowired
    private WebTestClient webTestClient;

    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUsers() {
        when(userService.findAll()).thenReturn(Flux.just(
            new User(1L, "john@test.com", "John")));

        webTestClient.get().uri("/api/users")
            .exchange()
            .expectStatus().isOk()
            .expectBodyList(UserResponse.class)
            .hasSize(1);
    }

    @Test
    void shouldReturn404WhenNotFound() {
        when(userService.findById(1L)).thenReturn(Mono.empty());

        webTestClient.get().uri("/api/users/1")
            .exchange()
            .expectStatus().isNotFound();
    }
}
```

## Usage

```
Skill("spring-reactive")
```

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.0.0 | 2024-12-30 | R2DBC, SSE, WebTestClient patterns |
| 1.0.0 | 2024-01-01 | Initial release |
