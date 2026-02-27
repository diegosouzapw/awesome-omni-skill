---
name: go-microservice
description: This skill should be used when the user asks to "создать микросервис", "добавить домен", "имплементировать фичу", "использовать Uber FX", "создать репозиторий", "добавить gRPC endpoint", "добавить HTTP endpoint", "создать воркер", "составь план создания микросервиса", "использовать внутренние пакеты", "настроить kafka/redis/postgres/кафку/редис/постгрес", mentions "service_template", "shared", or needs guidance on Go microservice architecture, domain-driven design patterns, or backend development.
---

# Go Microservice Development

This skill provides guidance for creating and extending Go microservices using the internal service template and backend packages.

## Overview

The microservice architecture follows Clean Architecture principles with Uber FX for dependency injection. All services share a common structure and use a set of internal infrastructure packages.

## Architecture Overview

```
cmd/app/main.go              # Entry point with signal handling
config/config.go             # Configuration with fx.Out pattern
internal/
├── app/app.go               # fx bootstrap (CreateApp)
├── domain/                  # Business logic (DDD)
│   ├── fx.go                # Domain modules aggregation
│   └── {name}/              # Domain module
│       ├── fx.go            # Module registration
│       ├── consts/          # Permission scopes
│       ├── entities/        # Domain entities
│       ├── dto/             # Request/Response objects
│       ├── deps/            # Interface definitions
│       ├── errors/          # Domain-specific errors
│       ├── repository/      # Data access (postgres, http_clients)
│       ├── usecase/         # Business logic
│       ├── delivery/        # HTTP, gRPC, Kafka, RabbitMQ handlers
│       └── workers/         # Background tasks
└── infrastructure/          # Servers, clients, interceptors
pkg/                         # Local utilities (errors, httputil, ctxutil)
```

## Domain Development Quick Guide

### Creating New Domain - Checklist

```
1. [ ] Create directory: internal/domain/{name}/
2. [ ] Create subdirectories: entities/, dto/, deps/, repository/postgres/,
       usecase/buissines/, delivery/http/, errors/
3. [ ] Define entities in entities/entities.go
4. [ ] Define DTOs in dto/dto.go
5. [ ] Define interfaces in deps/dep.go
6. [ ] Implement repository in repository/postgres/repo.go
7. [ ] Implement usecase in usecase/buissines/uc.go
8. [ ] Implement HTTP handlers in delivery/http/handlers.go
9. [ ] Implement router in delivery/http/router.go
10. [ ] Create fx.Module in fx.go
11. [ ] Register module in internal/domain/fx.go
12. [ ] Validate: go test -run Test__CreateApp ./internal/app
```

For detailed file templates, consult `examples/new-domain-checklist.md`.

## Layer Rules Summary

| Layer | Key Rules | Example Tags/Patterns |
|-------|-----------|----------------------|
| **Entities** | Use db/json tags, typed constants for enums | `db:"column" json:"field"` |
| **DTO** | Separate Request/Response, use validate tags | `validate:"required"` |
| **Deps** | Interface-only, context.Context first | `Create(ctx, entity) error` |
| **Repository** | Return interface, use NamedExec/Get/SelectContext | `func New(db) deps.Repo` |
| **Usecase** | Inject via interfaces, return DTOs, validate | Inject deps, return dto |
| **Delivery HTTP** | fasthttp, httputil.WriteResponse | `mapper.MapErrorToHttp(err)` |
| **Delivery gRPC** | OnStart/OnStop lifecycle | `lc.Append(fx.Hook{...})` |
| **Workers** | fx.Lifecycle, graceful shutdown via channels | `ticker + done channel` |

For detailed patterns and code examples, consult `references/layer-patterns.md`.

## Internal Packages Quick Reference

### Infrastructure Connectors

| Package | Purpose | FX Module | Key Interface |
|---------|---------|-----------|---------------|
| `pgconnector` | PostgreSQL | `pgconnectorfx.PGConnectorFx` | `IDB` |
| `redisconnector` | Redis/Sentinel | `redisfx.RedisFx` | `IRedis` |
| `kafkaconnector` | Kafka producer/consumer | Manual | `IProducer`, `IConsumer` |
| `rabbitconnector` | RabbitMQ | Manual | `IProducer`, `IConsumer` |
| `vaultconnector` | HashiCorp Vault | `VaultFx` | `Connector` |
| `s3` | S3 storage | `S3Fx` | `IS3` |
| `clickhouseconnector` | ClickHouse | `clickhouseconnectorfx.ClickHouseConnectorFx` | `ICH` |

### Observability

| Package | Purpose | FX Module | Key Interface |
|---------|---------|-----------|---------------|
| `logger` | Structured logging (zap) | `loggerfx.LoggerFx` | `ILogger` |
| `tracer` | OpenTelemetry tracing | `tracerfx.TracerFx` | `ITracer` |
| `meter` | Prometheus metrics | `meterFx` | `IMeter` |
| `healthcheck` | /healthz, /readyz probes | `healthfx.HealthCheckFx` | - |

### Utilities

| Package | Purpose |
|---------|---------|
| `configurator` | YAML/ENV configuration with validation |
| `events` | Event types for inter-service communication |
| `outbox` | Transactional Outbox pattern |
| `memcache` | In-memory cache (Cache, SnapshotCache, DeadlineCache) |
| `pagination` | Pagination primitives |
| `access_level_guard` | gRPC access control via protobuf annotations |
| `tlser` | TLS certificates for gRPC |
| `wallet_streamer` | Wallet subscription streaming |

For detailed usage with code examples, consult `references/internal-packages.md`.

## FX Module Essentials

### Module Registration Pattern

```go
// internal/domain/{name}/fx.go
var Module = fx.Module(
    "{name}",
    fx.Provide(
        postgres.NewRepository,
        buissines.NewUseCase,
        http.NewHandlers,
        http.NewRouter,
    ),
)
```

### Domain Aggregation

```go
// internal/domain/fx.go
var Module = fx.Module(
    "domain",
    order.Module,
    user.Module,
    // Add new domain modules here
)
```

### App Bootstrap

```go
// internal/app/app.go
func CreateApp() fx.Option {
    return fx.Options(
        loggerfx.LoggerFx,
        healthfx.HealthCheckFx,
        pgconnectorfx.PGConnectorFx,
        redisfx.RedisFx,
        domain.Module,
        infrastructure.Module,
        fx.Provide(config.Out, context.Background),
        healthfx.ReadinessProbeFX,
    )
}
```

### Workers with Lifecycle

```go
// internal/domain/{name}/workers/fx.go
var Module = fx.Module(
    "{name}-workers",
    fx.Provide(NewWorker),
    fx.Invoke(registerLifecycle),
)

func registerLifecycle(lc fx.Lifecycle, w *Worker) {
    lc.Append(fx.Hook{
        OnStart: func(ctx context.Context) error { w.Start(ctx); return nil },
        OnStop:  func(ctx context.Context) error { w.Stop(); return nil },
    })
}
```

For advanced patterns (fx.As, fx.Annotate, fx.In/Out), consult `references/fx-patterns.md`.

## Common Integration Scenarios

### Adding HTTP Endpoint

1. Add handler method to `delivery/http/handlers.go`
2. Register route in `delivery/http/router.go`
3. Add usecase method if needed
4. Add repository method if needed
5. Validate: `go test -run Test__CreateApp ./internal/app`

### Adding gRPC Endpoint

1. Update proto definition
2. Regenerate proto: `protoc --go_out=. --go-grpc_out=. *.proto`
3. Implement handler in `delivery/grpc/handlers.go`
4. Add usecase method if needed
5. Validate fx graph

### External HTTP Client Integration

1. Create client: `repository/http_clients/{service}/client.go`
2. Define interface in `deps/dep.go`
3. Add config in `config/config.go`:
   ```go
   ServiceName containers.RepositoryConfig `envPrefix:"SERVICE_NAME_"`
   ```
4. Register in fx.Module

### Background Worker

1. Create worker: `workers/worker.go` with ticker + done channel
2. Create fx module: `workers/fx.go` with lifecycle hooks
3. Register in domain fx.Module

### New Configuration Field

1. Add field to `config/config.go` with env/yaml/validate tags
2. Add to Result struct with `fx.Out`
3. Return in `Out()` function

For detailed step-by-step scenarios, consult `references/template-structure.md`.

## Error Handling

### Error Types and HTTP Codes

| Type | HTTP Code | Constructor |
|------|-----------|-------------|
| `ValidationError` | 400 | `NewValidationError(msg)` |
| `UnauthorizedError` | 401 | `NewUnauthorizedError(msg)` |
| `PermissionError` | 403 | `NewPermissionError(msg)` |
| `NotFoundError` | 404 | `NewNotFoundError(msg)` |
| `ConflictError` | 409 | `NewConflictError(msg)` |

### Domain Errors Pattern

```go
// internal/domain/{name}/errors/errors.go
package errors

import pkgerrors "service/pkg/errors"

var (
    OrderNotFound      = pkgerrors.NewNotFoundError("order not found")
    OrderAlreadyPaid   = pkgerrors.NewConflictError("order already paid")
    InvalidOrderAmount = pkgerrors.NewValidationError("invalid amount")
)
```

### Handler Error Mapping

```go
resp, err := h.uc.CreateOrder(ctx, &req)
if err != nil {
    status, msg := h.mapper.MapErrorToHttp(err)
    httputil.WriteErrorResponse(ctx, msg, status, err)
    return
}
httputil.WriteResponse(ctx, resp)
```

## Local Utilities (pkg/)

| Package | Purpose | Key Functions |
|---------|---------|---------------|
| `pkg/errors` | Typed errors | `NewValidationError`, `NewNotFoundError`, `NewMapper` |
| `pkg/httputil` | HTTP helpers | `WriteResponse`, `WriteError`, `NewMiddlewareGroup` |
| `pkg/ctxutil` | Context helpers | `FromCtx[T](ctx, key)`, `HasInCtx(ctx, key)` |
| `pkg/generator` | Data generation | `GenerateApiKey(prefix, length)` |
| `pkg/hasher` | Password hashing | `HashPassword(pwd)`, `ValidatePassword(hash, pwd)` |
| `pkg/mapfn` | Slice transforms | `ConvertSlice[T,R](input, fn)` |
| `pkg/timetools` | Time formatting | `FrontendTime` (JSON-compatible) |

### Context Keys (infrastructure/constants)

```go
const (
    UserIDContextKey      = "user_id"
    MerchantIDContextKey  = "merchant_id"
    SessionInfoContextKey = "session_info"
    ApiKeyInfoContextKey  = "api_key_info"
)
```

## Commands

```powershell
# Run application
go run ./cmd/app

# Run all tests
go test ./...

# Validate fx dependency graph
go test -run Test__CreateApp ./internal/app

# Generate swagger docs
swag init -g ./cmd/app/main.go -o ./docs

# Download dependencies
go mod download
```

## Additional Resources

### Reference Files

For detailed patterns and techniques, consult:
- **`references/template-structure.md`** - Complete file/folder structure with examples
- **`references/internal-packages.md`** - Detailed package documentation
- **`references/fx-patterns.md`** - Advanced FX DI patterns
- **`references/layer-patterns.md`** - Layer rules with code examples

### Example Files

Working examples in `examples/`:
- **`examples/new-domain-checklist.md`** - Step-by-step domain creation with copy-paste code

