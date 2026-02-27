---
name: go-expert
description: Expert guidance for Go (Golang) development following industry best practices. Use when writing Go code, reviewing PRs, bootstrapping new services, configuring linters, implementing observability, or troubleshooting Go applications. Covers SOLID principles, Gang of Four design patterns, domain-driven structure, error handling, context patterns, concurrency, testing, structured logging, health endpoints, and CI gates.
license: MIT
metadata:
  author: Tresic Engineering
  version: "1.0.0"
notes: Requires Go 1.21+.
---

# Go Expert Skill

## Overview

This skill provides expert guidance for Go development following industry best practices. It covers coding standards, project structure, observability requirements, and CI gates.

## When to Use This Skill

- Bootstrapping new Go microservices
- Writing or reviewing Go code
- Implementing health endpoints, logging, metrics, or tracing
- Configuring linters and CI pipelines
- Troubleshooting Go applications
- Setting up table-driven tests or race detection
- Implementing graceful shutdown patterns

---

## Project Layout

Follow a domain-driven folder structure, organizing code by business domain rather than technical layer:

```
project/
├── cmd/
│   └── myapp/
│       └── main.go           # Entry point (keep minimal, under 20 lines)
├── internal/
│   ├── domain/               # Business domains (the core)
│   │   ├── user/
│   │   │   ├── user.go       # Entity, value objects
│   │   │   ├── repository.go # Repository interface
│   │   │   ├── service.go    # Business logic
│   │   │   └── errors.go     # Domain-specific errors
│   │   ├── order/
│   │   │   ├── order.go
│   │   │   ├── repository.go
│   │   │   ├── service.go
│   │   │   └── errors.go
│   │   └── conversation/
│   │       ├── vcon.go
│   │       ├── repository.go
│   │       ├── service.go
│   │       └── errors.go
│   ├── infra/                # Infrastructure implementations
│   │   ├── postgres/         # Database adapters
│   │   ├── redis/            # Cache adapters
│   │   └── http/             # HTTP handlers
│   ├── app/                  # Application services (orchestration)
│   │   └── app.go
│   └── config/
│       └── config.go
├── pkg/                       # Public library code (if any)
├── api/                       # API definitions (OpenAPI, protobuf)
├── scripts/                   # Build/deployment scripts
├── testdata/                  # Test fixtures
├── go.mod
├── go.sum
└── Makefile
```

**Key Principles:**
- **Organize by domain, not by layer** — Group all user-related code together, not all handlers together
- **Domain independence** — Each domain package should be self-contained with its own entities, repository interfaces, and services
- **Dependency inversion** — Domain defines interfaces, infra implements them
- **Keep `cmd/` minimal** — Just wire dependencies and call `internal/`
- **Use `internal/`** — Enforces package boundaries at compile time
- **Infra adapters** — Database, cache, and HTTP handlers implement domain interfaces

---

## Design Principles

### SOLID Principles (Required)

All Go code MUST follow SOLID principles:

| Principle | Description | Go Example |
|-----------|-------------|------------|
| **S**ingle Responsibility | A struct/file should have one reason to change | `user/service.go` handles only user business logic |
| **O**pen/Closed | Open for extension, closed for modification | Use interfaces to extend behavior without modifying existing code |
| **L**iskov Substitution | Implementations must be substitutable for their interfaces | Any `Repository` implementation works with `Service` |
| **I**nterface Segregation | Many small interfaces > one large interface | `Reader`, `Writer` instead of `ReadWriter` |
| **D**ependency Inversion | Depend on abstractions, not concretions | Service depends on `Repository` interface, not `PostgresRepo` |

```go
// ✅ SOLID example

// Single Responsibility: UserService only handles user business logic
// Dependency Inversion: Depends on Repository interface, not concrete implementation
// Interface Segregation: Small, focused interface
type UserRepository interface {
    FindByID(ctx context.Context, id string) (*User, error)
    Save(ctx context.Context, user *User) error
}

type UserService struct {
    repo   UserRepository  // Depends on abstraction
    logger *slog.Logger
}

func NewUserService(repo UserRepository, logger *slog.Logger) *UserService {
    return &UserService{repo: repo, logger: logger}
}

// Open/Closed: New repository implementations don't require changing UserService
// Liskov Substitution: PostgresRepo, MockRepo, RedisRepo all work here
```

### Gang of Four Design Patterns (Required)

Apply Gang of Four patterns where appropriate:

**Creational Patterns:**
- **Factory** — Use constructor functions (`NewService`, `NewRepository`)
- **Builder** — For complex object construction with many optional parameters
- **Singleton** — Use sparingly; prefer dependency injection

**Structural Patterns:**
- **Adapter** — Wrap external APIs to match your domain interfaces
- **Decorator** — Add behavior (logging, metrics) without modifying core logic
- **Facade** — Simplify complex subsystems with a unified interface

**Behavioral Patterns:**
- **Strategy** — Inject different algorithms via interfaces
- **Observer** — Event-driven communication between components
- **Repository** — Abstract data access behind interfaces

```go
// Factory pattern
func NewUserService(repo UserRepository, logger *slog.Logger) *UserService {
    return &UserService{repo: repo, logger: logger}
}

// Options/Builder pattern for complex configuration
type ServiceOption func(*Service)

func WithTimeout(d time.Duration) ServiceOption {
    return func(s *Service) { s.timeout = d }
}

func WithRetries(n int) ServiceOption {
    return func(s *Service) { s.retries = n }
}

func NewService(repo Repository, opts ...ServiceOption) *Service {
    s := &Service{repo: repo, timeout: 30 * time.Second, retries: 3}
    for _, opt := range opts {
        opt(s)
    }
    return s
}

// Decorator pattern for adding logging
type LoggingRepository struct {
    next   UserRepository
    logger *slog.Logger
}

func (r *LoggingRepository) FindByID(ctx context.Context, id string) (*User, error) {
    r.logger.Info("finding user", "id", id)
    user, err := r.next.FindByID(ctx, id)
    if err != nil {
        r.logger.Error("failed to find user", "id", id, "error", err)
    }
    return user, err
}

// Strategy pattern via interfaces
type NotificationStrategy interface {
    Send(ctx context.Context, user *User, message string) error
}

type EmailNotifier struct{}
type SMSNotifier struct{}
type SlackNotifier struct{}

// All implement NotificationStrategy, injectable at runtime
```

---

## File Organization

**Guidelines:**
- **Functions under 50 lines** — If longer, extract helper functions
- **Files under 300 lines (soft limit)** — If exceeded, check for multiple responsibilities
- **One domain concept per file** — `user.go`, `repository.go`, `service.go`, `errors.go`
- **Don't artificially split cohesive code** — Keeping related code together trumps line limits

```
# Good: Each file has single responsibility
internal/domain/user/
├── user.go           # Entity and value objects (~50-100 lines)
├── repository.go     # Repository interface (~20-30 lines)
├── service.go        # Business logic (~100-200 lines)
├── errors.go         # Domain errors (~30-50 lines)
└── user_test.go      # Tests

# Bad: One massive file with everything
internal/domain/user/
└── user.go           # 800 lines with entity, repo, service, errors
```

**When to split a file:**
- Multiple structs with their own methods
- Mixed concerns (e.g., entity + HTTP handler)
- File exceeds 300 lines AND has multiple responsibilities

**When NOT to split:**
- Code is cohesive and related
- Splitting would require excessive cross-file navigation
- The file is long but has single responsibility

---

## Application Bootstrap Pattern

### Main Function

The `main()` function MUST be under 20 lines. Extract initialization into an `Application` struct:

```go
func main() {
    app, err := NewApplication()
    if err != nil {
        slog.Error("failed to initialize application", "error", err)
        os.Exit(1)
    }
    if err := app.Run(); err != nil {
        slog.Error("application error", "error", err)
        os.Exit(1)
    }
}
```

### Application Struct

```go
type Application struct {
    config   *config.Config
    log      *slog.Logger
    db       *sql.DB
    server   *http.Server
    services *Services
    tracer   trace.Tracer
}

func NewApplication() (*Application, error) {
    // Build dependencies in order
}

func (a *Application) Run() error {
    // Start server + graceful shutdown
}

func (a *Application) Shutdown(ctx context.Context) error {
    // Cleanup in reverse order
}
```

---

## Tooling Gates

All Go code MUST pass these checks before merge:

| Tool | Command | Requirement |
|------|---------|-------------|
| Format | `gofmt -l .` | No output |
| Vet | `go vet ./...` | Zero warnings |
| Lint | `golangci-lint run` | Zero errors |
| Test | `go test ./...` | All passing |
| Race | `go test -race ./...` | No races |
| Vuln | `govulncheck ./...` | No vulnerabilities |

### golangci-lint Configuration

```yaml
# .golangci.yml
run:
  timeout: 5m

linters:
  enable:
    - errcheck
    - gosimple
    - govet
    - ineffassign
    - staticcheck
    - unused
    - gofmt
    - goimports
    - misspell
    - unconvert
    - unparam
    - gosec
    - bodyclose
    - noctx

linters-settings:
  errcheck:
    check-type-assertions: true
  govet:
    check-shadowing: true
```

### Linter Compliance

NEVER use `//nolint` comments to suppress complexity warnings. If `gocyclo` or `gocognit` complains, refactor the code.

```go
// ❌ Never do this
//nolint:gocyclo
func main() { /* 200 lines */ }

// ✅ Refactor instead
func main() {
    app, err := NewApplication()
    // ...
}
```

---

## Health Endpoints

Every service MUST expose Kubernetes-compatible health checks:

- `GET /healthz` — Liveness probe (is the process alive?)
- `GET /readyz` — Readiness probe (can it accept traffic?)

```go
func (s *Server) registerHealthRoutes(mux *http.ServeMux) {
    mux.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        w.Write([]byte(`{"status":"healthy"}`))
    })

    mux.HandleFunc("/readyz", func(w http.ResponseWriter, r *http.Request) {
        if err := s.db.Ping(r.Context()); err != nil {
            w.WriteHeader(http.StatusServiceUnavailable)
            w.Write([]byte(`{"status":"not ready"}`))
            return
        }
        w.WriteHeader(http.StatusOK)
        w.Write([]byte(`{"status":"ready"}`))
    })
}
```

---

## Structured Logging

Use structured JSON logging for production services. A well-designed log entry includes:

```json
{
    "timestamp": "2025-01-15T10:30:45.123Z",
    "level": "INFO",
    "service": {
        "name": "my-service",
        "version": "1.2.3",
        "environment": "prod"
    },
    "message": "Request processed",
    "traceId": "abc123def456789012345678901234",
    "requestId": "req-789"
}
```

**Key Fields:**
- `timestamp`: ISO 8601, UTC
- `level`: Uppercase (TRACE, DEBUG, INFO, WARN, ERROR, FATAL)
- `service`: Name, version, environment
- `message`: Human-readable, NO PII
- `traceId`: For distributed tracing (32-char hex)

### PII Redaction (Required)

NEVER log PII. Implement redaction before logging:

```go
// ❌ Never log raw PII
logger.Info("user registered", "email", user.Email)

// ✅ Redact PII
logger.Info("user registered",
    "userId", user.ID,
    "email", redactEmail(user.Email),
    "clientIp", redactIP(request.RemoteAddr),
)

func redactEmail(email string) string {
    parts := strings.Split(email, "@")
    if len(parts) == 2 {
        return "[REDACTED]@" + parts[1]
    }
    return "[EMAIL_REDACTED]"
}
```

**Never log:** passwords, API keys, tokens, credit card numbers, SSN, phone numbers, full addresses, dates of birth.

---

## Metrics

Every service MUST expose Prometheus metrics at `/metrics`:

```go
var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total HTTP requests",
        },
        []string{"method", "path", "status"},
    )
    httpRequestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request latency",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "path"},
    )
)
```

---

## Tracing

OpenTelemetry with W3C Trace Context propagation:

```go
func (s *Service) Process(ctx context.Context, id string) error {
    tracer := otel.Tracer("my-service")
    ctx, span := tracer.Start(ctx, "Process")
    defer span.End()

    span.SetAttributes(attribute.String("entity.id", id))
    return s.repo.Get(ctx, id)
}
```

---

## Error Handling

### Error Wrapping

Always wrap errors with context using `%w`:

```go
if err := database.Connect(cfg); err != nil {
    return fmt.Errorf("connecting to database: %w", err)
}
```

All initialization errors bubble up to `main()` — don't `os.Exit()` deep in the call stack.

### Sentinel Errors

Define sentinel errors for expected conditions:

```go
var (
    ErrNotFound     = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrRateLimited  = errors.New("rate limited")
)
```

### Error Checking

Use `errors.Is()` and `errors.As()`:

```go
if errors.Is(err, ErrNotFound) {
    // Handle not found
}

var validationErr *ValidationError
if errors.As(err, &validationErr) {
    log.Printf("Invalid field: %s", validationErr.Field)
}
```

### Return Early

Use guard clauses to avoid deep nesting:

```go
// ❌ Deep nesting
func process(data []byte) error {
    if data != nil {
        if len(data) > 0 {
            if isValid(data) {
                // work
            }
        }
    }
    return nil
}

// ✅ Return early
func process(data []byte) error {
    if data == nil {
        return ErrNilData
    }
    if len(data) == 0 {
        return ErrEmptyData
    }
    if !isValid(data) {
        return ErrInvalidData
    }
    // work
    return nil
}
```

---

## Context & Concurrency

### Context as First Parameter

```go
func (s *Service) GetUser(ctx context.Context, id string) (*User, error) {
    select {
    case <-ctx.Done():
        return nil, ctx.Err()
    default:
    }
    return s.repo.FindByID(ctx, id)
}
```

### Always Defer Cancel

```go
func processWithTimeout(parent context.Context) error {
    ctx, cancel := context.WithTimeout(parent, 30*time.Second)
    defer cancel()
    return doWork(ctx)
}
```

### Guard Shared State

```go
type Counter struct {
    mu    sync.Mutex
    value int64
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}
```

---

## Testing

### Table-Driven Tests

```go
func TestCalculateDiscount(t *testing.T) {
    tests := []struct {
        name       string
        price      float64
        percentage float64
        want       float64
    }{
        {"10% off $100", 100, 10, 90},
        {"no discount", 100, 0, 100},
        {"full discount", 100, 100, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := CalculateDiscount(tt.price, tt.percentage)
            if got != tt.want {
                t.Errorf("got %v, want %v", got, tt.want)
            }
        })
    }
}
```

### Race Detection

Always run tests with race detection:

```bash
go test -race ./...
```

### Integration Tests

Use build tags to separate integration tests:

```go
//go:build integration

package integration

func TestDatabaseConnection(t *testing.T) {
    // Requires real database
}
```

```bash
go test -tags=integration ./...
```

---

## Makefile Template

```makefile
.PHONY: lint test build check

lint:
	gofmt -l .
	go vet ./...
	golangci-lint run

test:
	go test -race -coverprofile=coverage.out ./...
	go tool cover -func=coverage.out

build:
	go build -o bin/myapp ./cmd/myapp

check: lint test
	govulncheck ./...
```

---

## New Service Checklist

- [ ] `main()` under 20 lines
- [ ] Project follows domain-driven folder structure
- [ ] SOLID principles applied throughout
- [ ] Gang of Four patterns used appropriately
- [ ] Functions under 50 lines
- [ ] Files under 300 lines (or single responsibility if longer)
- [ ] `Application` struct with `NewApplication()`, `Run()`, `Shutdown()`
- [ ] `/healthz` and `/readyz` endpoints
- [ ] Prometheus metrics at `/metrics`
- [ ] OpenTelemetry tracing configured
- [ ] TraceId propagation (W3C Trace Context, 32-char hex)
- [ ] JSON structured logging
- [ ] PII redaction implemented and tested
- [ ] No `//nolint` for complexity
- [ ] All errors wrapped with context
- [ ] Sentinel errors for expected conditions
- [ ] Graceful shutdown with configurable timeout
- [ ] Table-driven tests with `t.Run()` subtests
- [ ] Race detection passes (`go test -race`)
- [ ] `govulncheck` passes
- [ ] `golangci-lint` passes with standard config

---

## Anti-Patterns to Avoid

```
❌ Organizing by technical layer instead of domain
   internal/handlers/, internal/models/, internal/services/
   → internal/domain/user/, internal/domain/order/

❌ Violating Single Responsibility
   UserService that also sends emails and logs metrics
   → Separate EmailService, MetricsService

❌ Depending on concretions instead of abstractions
   func NewService(repo *PostgresRepo) → func NewService(repo Repository)

❌ Large interfaces (violates Interface Segregation)
   type Repository interface { 20 methods... }
   → Split into Reader, Writer, Deleter interfaces

❌ Functions over 50 lines
   Refactor into smaller, focused helper functions

❌ Files over 300 lines with multiple responsibilities
   Split by domain concept: entity, repository, service, errors

❌ Using //nolint to suppress complexity warnings
   //nolint:gocyclo → Refactor into smaller functions

❌ Long main() functions
   func main() { /* 200 lines */ } → Extract to Application struct

❌ Panicking in library code
   panic(err) → return fmt.Errorf("context: %w", err)

❌ Deep nesting instead of guard clauses
   if x { if y { if z { } } } → Return early with guard clauses

❌ Ignoring context cancellation
   func Process(id string) → func Process(ctx context.Context, id string)

❌ Wrong health endpoint paths
   /health, /ready → /healthz, /readyz (Kubernetes convention)

❌ Logging PII
   logger.Info("user", "email", user.Email) → Redact before logging

❌ Missing traceId in logs
   slog.Info("message") → Include traceId on every log entry

❌ Using offset pagination
   ?page=3&per_page=25 → Use cursor-based pagination

❌ os.Exit() deep in call stack
   os.Exit(1) in helper → Bubble errors up to main()

❌ Not deferring cancel()
   ctx, cancel := context.WithTimeout(...) → defer cancel() immediately

❌ Interface pollution
   Large interfaces → Small, focused interfaces (1-3 methods)

❌ Not running race detector
   go test ./... → go test -race ./...
```

---

## Related Resources

- [Go DDD Example](https://github.com/marcusolsson/goddd)
- [SOLID Go Design](https://dave.cheney.net/2016/08/20/solid-go-design)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [OpenTelemetry Go](https://opentelemetry.io/docs/languages/go/)
- [Prometheus Go Client](https://github.com/prometheus/client_golang)
