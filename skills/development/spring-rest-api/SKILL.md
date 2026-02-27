---
name: spring-rest-api
description: RESTful API design with Spring Boot including OpenAPI/Swagger documentation, content negotiation, CORS, pagination, HATEOAS, and API versioning patterns.
version: "1.0.0"
author: GazApps
tags: [rest, api, spring-boot, openapi, swagger, hateoas, pagination]
dependencies: [java-fundamentals, spring-boot-core]
compatibility: [antigravity, claude-code, gemini-cli]
---

# Spring REST API

Build production-ready RESTful APIs with Spring Boot following industry best practices for design, documentation, error handling, and security.

## When to Use

- Building HTTP APIs consumed by web/mobile frontends, third-party integrations, or microservices
- Exposing CRUD operations over domain entities with pagination, sorting, and filtering
- Creating public or internal APIs that require OpenAPI/Swagger documentation
- Projects needing standardized error responses (RFC 7807 Problem Detail)
- APIs requiring content negotiation (JSON, XML), CORS, or versioning

## When NOT to Use

- Real-time bidirectional communication -- use WebSockets or SSE instead
- File-heavy streaming endpoints -- consider Spring WebFlux or dedicated file services
- GraphQL APIs -- use `spring-boot-starter-graphql`
- gRPC services -- use `grpc-spring-boot-starter`
- Simple internal method calls between co-located services -- direct invocation is simpler

## Dependencies

### Maven

```xml
<dependencies>
    <!-- Core web starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- OpenAPI / Swagger UI -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.8.4</version>
    </dependency>

    <!-- HATEOAS (optional, for hypermedia-driven APIs) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-hateoas</artifactId>
    </dependency>

    <!-- Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- XML content negotiation (optional) -->
    <dependency>
        <groupId>com.fasterxml.jackson.dataformat</groupId>
        <artifactId>jackson-dataformat-xml</artifactId>
    </dependency>
</dependencies>
```

### Gradle

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.4'
    implementation 'org.springframework.boot:spring-boot-starter-hateoas'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-xml' // optional
}
```

## REST Conventions

### URL Patterns

Follow resource-oriented URL design. URLs represent nouns (resources), not verbs (actions).

| Pattern | Purpose | Example |
|---|---|---|
| `GET /api/v1/users` | List collection | Fetch all users (paginated) |
| `GET /api/v1/users/{id}` | Get single resource | Fetch user by ID |
| `POST /api/v1/users` | Create resource | Create a new user |
| `PUT /api/v1/users/{id}` | Full replace | Replace entire user |
| `PATCH /api/v1/users/{id}` | Partial update | Update specific fields |
| `DELETE /api/v1/users/{id}` | Delete resource | Remove user |
| `GET /api/v1/users/{id}/orders` | Sub-resource collection | User's orders |
| `GET /api/v1/users/{id}/orders/{orderId}` | Sub-resource item | Specific order |

Rules:
- Use plural nouns: `/users` not `/user`
- Use kebab-case for multi-word resources: `/order-items` not `/orderItems`
- Nest sub-resources at most one level deep
- Use query parameters for filtering, sorting, and pagination
- Prefix all API routes with `/api/v1` (or versioned equivalent)

### HTTP Methods

| Method | Idempotent | Safe | Request Body | Typical Use |
|---|---|---|---|---|
| `GET` | Yes | Yes | No | Retrieve resource(s) |
| `POST` | No | No | Yes | Create resource |
| `PUT` | Yes | No | Yes | Full replacement |
| `PATCH` | No | No | Yes | Partial update |
| `DELETE` | Yes | No | No | Remove resource |
| `HEAD` | Yes | Yes | No | Check existence |
| `OPTIONS` | Yes | Yes | No | CORS preflight |

### HTTP Status Codes

| Code | When to Use |
|---|---|
| `200 OK` | Successful GET, PUT, PATCH, or DELETE returning body |
| `201 Created` | Successful POST creating a resource (include `Location` header) |
| `204 No Content` | Successful DELETE or PUT/PATCH with no response body |
| `400 Bad Request` | Malformed request syntax, invalid JSON |
| `401 Unauthorized` | Missing or invalid authentication |
| `403 Forbidden` | Authenticated but lacking permission |
| `404 Not Found` | Resource does not exist |
| `409 Conflict` | State conflict (duplicate email, concurrent edit) |
| `422 Unprocessable Entity` | Validation errors on well-formed request |
| `500 Internal Server Error` | Unhandled server exception |

See `references/http-status-codes.md` for complete reference with Spring examples.

## Controller Patterns

### Basic REST Controller

```java
@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "Users", description = "User management operations")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public ResponseEntity<PageResponse<UserResponse>> list(
            @ParameterObject Pageable pageable,
            @RequestParam(required = false) String search) {
        Page<UserResponse> page = userService.findAll(search, pageable);
        return ResponseEntity.ok(PageResponse.of(page));
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserResponse> getById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }

    @PostMapping
    public ResponseEntity<UserResponse> create(
            @Valid @RequestBody CreateUserRequest request) {
        UserResponse created = userService.create(request);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}").buildAndExpand(created.id()).toUri();
        return ResponseEntity.created(location).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<UserResponse> replace(
            @PathVariable Long id,
            @Valid @RequestBody UpdateUserRequest request) {
        return ResponseEntity.ok(userService.replace(id, request));
    }

    @PatchMapping("/{id}")
    public ResponseEntity<UserResponse> patch(
            @PathVariable Long id,
            @Valid @RequestBody PatchUserRequest request) {
        return ResponseEntity.ok(userService.patch(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Key Annotations

| Annotation | Purpose |
|---|---|
| `@RestController` | Combines `@Controller` + `@ResponseBody` |
| `@RequestMapping("/path")` | Base path for all endpoints in controller |
| `@GetMapping` | Handle GET requests |
| `@PostMapping` | Handle POST requests |
| `@PutMapping` | Handle PUT requests |
| `@PatchMapping` | Handle PATCH requests |
| `@DeleteMapping` | Handle DELETE requests |
| `@PathVariable` | Extract value from URL path segment |
| `@RequestParam` | Extract query parameter |
| `@RequestBody` | Deserialize request body |
| `@Valid` | Trigger Bean Validation on parameter |
| `@ResponseStatus` | Set default HTTP status for handler |
| `@ParameterObject` | Expand Pageable into individual query params in OpenAPI |

## Request/Response DTOs

Use Java records for immutable, concise DTOs. Separate request and response representations.

```java
// Request DTO -- what the client sends
public record CreateUserRequest(
        @NotBlank @Size(max = 100)
        @Schema(description = "User's full name", example = "Jane Doe")
        String name,

        @NotBlank @Email @Size(max = 255)
        @Schema(description = "Unique email address", example = "jane@example.com")
        String email,

        @NotBlank @Size(min = 8, max = 72)
        @Schema(description = "Password (8-72 characters)", example = "s3cur3P@ss!")
        String password
) {}

// Response DTO -- what the client receives (never expose passwords, internal IDs, etc.)
public record UserResponse(
        @Schema(description = "Unique identifier", example = "42")
        Long id,

        @Schema(description = "User's full name", example = "Jane Doe")
        String name,

        @Schema(description = "Email address", example = "jane@example.com")
        String email,

        @Schema(description = "Account status", example = "ACTIVE")
        String status,

        @Schema(description = "Account creation timestamp")
        Instant createdAt
) {}
```

Rules:
- Never reuse the same DTO for create, update, and response
- Never expose sensitive fields (passwords, internal flags) in responses
- Use `@Schema` on every field for OpenAPI documentation
- Apply Bean Validation annotations on request DTOs
- Use `Instant` for timestamps, let Jackson serialize to ISO-8601

## Pagination

### Using Spring's Pageable

Spring Data provides `Pageable` and `Page` out of the box. Clients pass `page`, `size`, and `sort` query parameters.

```
GET /api/v1/users?page=0&size=20&sort=name,asc&sort=createdAt,desc
```

### Custom PageResponse Wrapper

Wrap Spring's `Page` in a consistent, API-friendly format:

```java
public record PageResponse<T>(
        List<T> content,
        PageMetadata metadata
) {
    public record PageMetadata(
            int page,
            int size,
            long totalElements,
            int totalPages,
            boolean first,
            boolean last
    ) {}

    public static <T> PageResponse<T> of(Page<T> page) {
        return new PageResponse<>(
                page.getContent(),
                new PageMetadata(
                        page.getNumber(),
                        page.getSize(),
                        page.getTotalElements(),
                        page.getTotalPages(),
                        page.isFirst(),
                        page.isLast()
                )
        );
    }
}
```

### Pagination Configuration

```yaml
spring:
  data:
    web:
      pageable:
        default-page-size: 20
        max-page-size: 100
        one-indexed-parameters: false
```

## Sorting

Accept `sort` as a query parameter. Spring Data parses `sort=field,direction` automatically.

```java
@GetMapping
public ResponseEntity<PageResponse<UserResponse>> list(
        @ParameterObject Pageable pageable) {
    // pageable.getSort() contains parsed Sort object
    Page<UserResponse> page = userService.findAll(pageable);
    return ResponseEntity.ok(PageResponse.of(page));
}
```

To restrict sortable fields, validate in the service layer:

```java
private static final Set<String> ALLOWED_SORT_FIELDS = Set.of("name", "email", "createdAt");

private Pageable validateSort(Pageable pageable) {
    for (Sort.Order order : pageable.getSort()) {
        if (!ALLOWED_SORT_FIELDS.contains(order.getProperty())) {
            throw new InvalidSortFieldException(order.getProperty(), ALLOWED_SORT_FIELDS);
        }
    }
    return pageable;
}
```

## Filtering

### Simple Query Parameters

```java
@GetMapping
public ResponseEntity<PageResponse<UserResponse>> list(
        @RequestParam(required = false) String search,
        @RequestParam(required = false) UserStatus status,
        @RequestParam(required = false) @DateTimeFormat(iso = ISO.DATE) LocalDate createdAfter,
        @ParameterObject Pageable pageable) {
    Page<UserResponse> page = userService.findAll(search, status, createdAfter, pageable);
    return ResponseEntity.ok(PageResponse.of(page));
}
```

### JPA Specification Pattern

For complex filtering, use Spring Data JPA Specifications:

```java
public class UserSpecifications {

    public static Specification<User> hasStatus(UserStatus status) {
        return (root, query, cb) ->
                status == null ? null : cb.equal(root.get("status"), status);
    }

    public static Specification<User> nameLike(String search) {
        return (root, query, cb) ->
                search == null ? null : cb.like(cb.lower(root.get("name")),
                        "%" + search.toLowerCase() + "%");
    }

    public static Specification<User> createdAfter(LocalDate date) {
        return (root, query, cb) ->
                date == null ? null : cb.greaterThanOrEqualTo(
                        root.get("createdAt"), date.atStartOfDay().toInstant(ZoneOffset.UTC));
    }
}
```

Combine in service:

```java
public Page<UserResponse> findAll(String search, UserStatus status,
                                   LocalDate createdAfter, Pageable pageable) {
    Specification<User> spec = Specification
            .where(UserSpecifications.nameLike(search))
            .and(UserSpecifications.hasStatus(status))
            .and(UserSpecifications.createdAfter(createdAfter));
    return userRepository.findAll(spec, pageable).map(userMapper::toResponse);
}
```

## CORS Configuration

### Profile-Based Configuration

```java
@Configuration
public class CorsConfig {

    @Bean
    @Profile("dev")
    public WebMvcConfigurer devCorsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                        .allowedOriginPatterns("*")
                        .allowedMethods("*")
                        .allowedHeaders("*")
                        .allowCredentials(true)
                        .maxAge(3600);
            }
        };
    }

    @Bean
    @Profile("prod")
    public WebMvcConfigurer prodCorsConfigurer(
            @Value("${app.cors.allowed-origins}") List<String> allowedOrigins) {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                        .allowedOrigins(allowedOrigins.toArray(String[]::new))
                        .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
                        .allowedHeaders("Authorization", "Content-Type", "Accept")
                        .exposedHeaders("Location", "X-Total-Count")
                        .allowCredentials(true)
                        .maxAge(3600);
            }
        };
    }
}
```

## Content Negotiation

Spring Boot supports JSON by default. Add XML support with `jackson-dataformat-xml`.

```yaml
spring:
  mvc:
    contentnegotiation:
      favor-parameter: false
      favor-path-extension: false
```

Clients use the `Accept` header:

```
GET /api/v1/users
Accept: application/json    # JSON response (default)
Accept: application/xml     # XML response
```

To restrict to JSON only on a specific controller:

```java
@RestController
@RequestMapping(path = "/api/v1/users", produces = MediaType.APPLICATION_JSON_VALUE)
public class UserController { ... }
```

## API Versioning

### Strategy 1: URL Path (Recommended for most projects)

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 { ... }

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 { ... }
```

### Strategy 2: Custom Request Header

```java
@GetMapping(headers = "X-API-Version=1")
public ResponseEntity<UserResponseV1> getUserV1(@PathVariable Long id) { ... }

@GetMapping(headers = "X-API-Version=2")
public ResponseEntity<UserResponseV2> getUserV2(@PathVariable Long id) { ... }
```

### Strategy 3: Media Type (Accept Header)

```java
@GetMapping(produces = "application/vnd.myapp.v1+json")
public ResponseEntity<UserResponseV1> getUserV1(@PathVariable Long id) { ... }

@GetMapping(produces = "application/vnd.myapp.v2+json")
public ResponseEntity<UserResponseV2> getUserV2(@PathVariable Long id) { ... }
```

### Recommendation

Use URL path versioning (`/api/v1/`) for simplicity and discoverability. Reserve header/media-type versioning for APIs with strict backward-compatibility contracts.

## OpenAPI / Swagger

### Configuration

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("My Application API")
                        .version("1.0.0")
                        .description("RESTful API documentation")
                        .contact(new Contact()
                                .name("API Support")
                                .email("support@example.com")))
                .addSecurityItem(new SecurityRequirement().addList("Bearer"))
                .components(new Components()
                        .addSecuritySchemes("Bearer",
                                new SecurityScheme()
                                        .type(SecurityScheme.Type.HTTP)
                                        .scheme("bearer")
                                        .bearerFormat("JWT")));
    }
}
```

### Key Annotations

```java
@Operation(summary = "Get user by ID", description = "Retrieves a single user by their unique identifier")
@ApiResponse(responseCode = "200", description = "User found")
@ApiResponse(responseCode = "404", description = "User not found",
        content = @Content(schema = @Schema(implementation = ProblemDetail.class)))
@GetMapping("/{id}")
public ResponseEntity<UserResponse> getById(@PathVariable Long id) { ... }
```

See `references/openapi-annotations.md` for complete annotation reference.

### Swagger UI Access

After starting the application:
- Swagger UI: `http://localhost:8080/swagger-ui.html`
- OpenAPI JSON: `http://localhost:8080/v3/api-docs`
- OpenAPI YAML: `http://localhost:8080/v3/api-docs.yaml`

## Error Responses -- ProblemDetail (RFC 7807)

Spring 6+ natively supports RFC 7807 `ProblemDetail` responses.

### Enable globally

```yaml
spring:
  mvc:
    problemdetails:
      enabled: true
```

### Custom Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ProblemDetail handleNotFound(ResourceNotFoundException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
                HttpStatus.NOT_FOUND, ex.getMessage());
        problem.setTitle("Resource Not Found");
        problem.setProperty("resource", ex.getResourceName());
        problem.setProperty("identifier", ex.getIdentifier());
        return problem;
    }
}
```

### Response Format

```json
{
    "type": "about:blank",
    "title": "Resource Not Found",
    "status": 404,
    "detail": "User with id 42 not found",
    "instance": "/api/v1/users/42",
    "resource": "User",
    "identifier": "42"
}
```

## HATEOAS

Add hypermedia links to responses for API discoverability.

```java
@GetMapping("/{id}")
public EntityModel<UserResponse> getById(@PathVariable Long id) {
    UserResponse user = userService.findById(id);
    return EntityModel.of(user,
            linkTo(methodOn(UserController.class).getById(id)).withSelfRel(),
            linkTo(methodOn(UserController.class).list(Pageable.unpaged(), null))
                    .withRel("users"),
            linkTo(methodOn(OrderController.class).listByUser(id, Pageable.unpaged()))
                    .withRel("orders"));
}
```

Response:

```json
{
    "id": 42,
    "name": "Jane Doe",
    "email": "jane@example.com",
    "_links": {
        "self": { "href": "/api/v1/users/42" },
        "users": { "href": "/api/v1/users" },
        "orders": { "href": "/api/v1/users/42/orders" }
    }
}
```

Use HATEOAS when:
- Building public APIs consumed by diverse, evolving clients
- API discoverability is a requirement
- You want clients to navigate the API without hardcoded URLs

Skip HATEOAS when:
- Internal APIs with known consumers (e.g., your own SPA)
- Simplicity is more important than discoverability

## Code Quality Checklist

Before submitting a REST API for review, verify:

- [ ] All endpoints return appropriate HTTP status codes (201 for POST, 204 for DELETE, etc.)
- [ ] POST endpoints return `Location` header pointing to the created resource
- [ ] Request DTOs have Bean Validation annotations (`@NotBlank`, `@Size`, `@Email`, etc.)
- [ ] Response DTOs never expose sensitive fields (passwords, tokens, internal flags)
- [ ] Separate DTOs for create, update, and response -- never reuse a single DTO
- [ ] Pagination is applied to all collection endpoints (never return unbounded lists)
- [ ] Sort fields are validated against an allow-list
- [ ] All endpoints have OpenAPI annotations (`@Operation`, `@ApiResponse`, `@Schema`)
- [ ] Error responses use ProblemDetail (RFC 7807) format
- [ ] CORS is configured per environment (permissive dev, restrictive prod)
- [ ] No business logic in controllers -- delegate to service layer
- [ ] `@Valid` is present on all `@RequestBody` parameters
- [ ] Path variables and query parameters have clear, documented names
- [ ] API version prefix is consistent across all controllers (`/api/v1/`)
- [ ] Integration tests cover happy path, validation errors, not-found, and conflict scenarios
