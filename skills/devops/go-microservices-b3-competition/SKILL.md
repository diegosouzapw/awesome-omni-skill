---
name: go-microservices
description: Production-ready Go microservices patterns including Gin, Echo, gRPC, clean architecture, dependency injection, error handling, middleware, testing, Docker containerization, Kubernetes deployment, distributed tracing, observability with Prometheus, high-performance APIs, concurrent processing, database integration with GORM, Redis caching, message queues, and cloud-native best practices.
---

# Go Microservices Development Skill

## Purpose

Build high-performance, scalable microservices in Go following clean architecture, industry best practices, and cloud-native patterns for production AWS deployments.

## When to Use This Skill

Auto-activates when working with:
- Go microservice development
- gRPC service implementation
- RESTful APIs with Gin/Echo
- High-performance backend services
- Concurrent data processing
- Docker containerization
- Kubernetes deployments
- Distributed systems

## Core Principles

### 1. Clean Architecture (Hexagonal)
```
Handlers (HTTP/gRPC) → Use Cases (Business Logic) → Repositories (Data Access) → External Systems
```

### 2. Dependency Injection
- Wire dependencies at startup
- Use interfaces for testability
- Avoid global state

### 3. Concurrency
- Leverage goroutines and channels
- Use context for cancellation
- Implement worker pools

### 4. Error Handling
- Wrap errors with context
- Use custom error types
- Log with structured fields

## Quick Start Examples

### Clean Architecture Project Structure

```
service/
├── cmd/
│   └── api/
│       └── main.go              # Application entry point
├── internal/
│   ├── domain/                  # Business entities
│   │   ├── user.go
│   │   └── errors.go
│   ├── usecase/                 # Business logic
│   │   └── user_usecase.go
│   ├── repository/              # Data access
│   │   └── user_repository.go
│   ├── handler/                 # HTTP/gRPC handlers
│   │   ├── http/
│   │   │   └── user_handler.go
│   │   └── grpc/
│   │       └── user_service.go
│   ├── middleware/              # HTTP middleware
│   │   ├── auth.go
│   │   └── logging.go
│   └── config/                  # Configuration
│       └── config.go
├── pkg/                         # Public libraries
│   ├── logger/
│   └── database/
├── proto/                       # gRPC definitions
│   └── user.proto
├── Dockerfile
├── go.mod
└── go.sum
```

### HTTP API with Gin (Clean Architecture)

```go
package main

import (
    "context"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/gin-gonic/gin"
    "go.uber.org/zap"
)

// Domain Entity
type User struct {
    ID        string    `json:"id"`
    Email     string    `json:"email"`
    Name      string    `json:"name"`
    CreatedAt time.Time `json:"created_at"`
}

// Repository Interface
type UserRepository interface {
    Create(ctx context.Context, user *User) error
    FindByID(ctx context.Context, id string) (*User, error)
    FindByEmail(ctx context.Context, email string) (*User, error)
}

// Use Case
type UserUseCase struct {
    repo   UserRepository
    logger *zap.Logger
}

func NewUserUseCase(repo UserRepository, logger *zap.Logger) *UserUseCase {
    return &UserUseCase{repo: repo, logger: logger}
}

func (uc *UserUseCase) CreateUser(ctx context.Context, email, name string) (*User, error) {
    // Check if user exists
    existing, err := uc.repo.FindByEmail(ctx, email)
    if err != nil && err != ErrNotFound {
        uc.logger.Error("failed to check existing user", zap.Error(err))
        return nil, err
    }
    if existing != nil {
        return nil, ErrUserAlreadyExists
    }

    // Create new user
    user := &User{
        ID:        generateID(),
        Email:     email,
        Name:      name,
        CreatedAt: time.Now(),
    }

    if err := uc.repo.Create(ctx, user); err != nil {
        uc.logger.Error("failed to create user", zap.Error(err))
        return nil, err
    }

    uc.logger.Info("user created", zap.String("user_id", user.ID))
    return user, nil
}

// HTTP Handler
type UserHandler struct {
    useCase *UserUseCase
    logger  *zap.Logger
}

func NewUserHandler(useCase *UserUseCase, logger *zap.Logger) *UserHandler {
    return &UserHandler{useCase: useCase, logger: logger}
}

type CreateUserRequest struct {
    Email string `json:"email" binding:"required,email"`
    Name  string `json:"name" binding:"required,min=2"`
}

func (h *UserHandler) CreateUser(c *gin.Context) {
    var req CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    user, err := h.useCase.CreateUser(c.Request.Context(), req.Email, req.Name)
    if err != nil {
        if err == ErrUserAlreadyExists {
            c.JSON(http.StatusConflict, gin.H{"error": "user already exists"})
            return
        }
        h.logger.Error("failed to create user", zap.Error(err))
        c.JSON(http.StatusInternalServerError, gin.H{"error": "internal server error"})
        return
    }

    c.JSON(http.StatusCreated, user)
}

// Main Application
func main() {
    logger, _ := zap.NewProduction()
    defer logger.Sync()

    // Initialize dependencies
    db := initDatabase()
    repo := NewPostgresUserRepository(db)
    useCase := NewUserUseCase(repo, logger)
    handler := NewUserHandler(useCase, logger)

    // Setup router
    router := gin.Default()
    router.Use(LoggingMiddleware(logger))
    router.Use(RequestIDMiddleware())

    v1 := router.Group("/api/v1")
    {
        users := v1.Group("/users")
        {
            users.POST("", handler.CreateUser)
            users.GET("/:id", handler.GetUser)
        }
    }

    // Graceful shutdown
    srv := &http.Server{
        Addr:    ":8080",
        Handler: router,
    }

    go func() {
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            logger.Fatal("failed to start server", zap.Error(err))
        }
    }()

    // Wait for interrupt signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    logger.Info("shutting down server...")

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        logger.Fatal("server forced to shutdown", zap.Error(err))
    }

    logger.Info("server exited")
}
```

### gRPC Service

```go
// proto/user.proto
syntax = "proto3";

package user;

option go_package = "github.com/yourorg/user-service/proto";

service UserService {
  rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
}

message CreateUserRequest {
  string email = 1;
  string name = 2;
}

message CreateUserResponse {
  string id = 1;
  string email = 2;
  string name = 3;
}

// Implementation
package grpc

import (
    "context"
    pb "github.com/yourorg/user-service/proto"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
)

type UserServiceServer struct {
    pb.UnimplementedUserServiceServer
    useCase *UserUseCase
    logger  *zap.Logger
}

func (s *UserServiceServer) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.CreateUserResponse, error) {
    user, err := s.useCase.CreateUser(ctx, req.Email, req.Name)
    if err != nil {
        if err == ErrUserAlreadyExists {
            return nil, status.Error(codes.AlreadyExists, "user already exists")
        }
        s.logger.Error("failed to create user", zap.Error(err))
        return nil, status.Error(codes.Internal, "internal error")
    }

    return &pb.CreateUserResponse{
        Id:    user.ID,
        Email: user.Email,
        Name:  user.Name,
    }, nil
}
```

### Worker Pool Pattern

```go
package worker

import (
    "context"
    "sync"
)

type Job struct {
    ID   string
    Data interface{}
}

type Result struct {
    JobID string
    Data  interface{}
    Error error
}

type WorkerPool struct {
    workerCount int
    jobs        chan Job
    results     chan Result
    wg          sync.WaitGroup
}

func NewWorkerPool(workerCount int) *WorkerPool {
    return &WorkerPool{
        workerCount: workerCount,
        jobs:        make(chan Job, workerCount*2),
        results:     make(chan Result, workerCount*2),
    }
}

func (wp *WorkerPool) Start(ctx context.Context, processor func(Job) (interface{}, error)) {
    for i := 0; i < wp.workerCount; i++ {
        wp.wg.Add(1)
        go wp.worker(ctx, processor)
    }
}

func (wp *WorkerPool) worker(ctx context.Context, processor func(Job) (interface{}, error)) {
    defer wp.wg.Done()

    for {
        select {
        case job, ok := <-wp.jobs:
            if !ok {
                return
            }
            data, err := processor(job)
            wp.results <- Result{
                JobID: job.ID,
                Data:  data,
                Error: err,
            }
        case <-ctx.Done():
            return
        }
    }
}

func (wp *WorkerPool) Submit(job Job) {
    wp.jobs <- job
}

func (wp *WorkerPool) Results() <-chan Result {
    return wp.results
}

func (wp *WorkerPool) Stop() {
    close(wp.jobs)
    wp.wg.Wait()
    close(wp.results)
}
```

## Resource Files

### [resources/clean-architecture.md](resources/clean-architecture.md)
- Layered architecture implementation
- Dependency injection patterns
- Interface design
- Testability strategies

### [resources/grpc-services.md](resources/grpc-services.md)
- Proto definition best practices
- Interceptors and middleware
- Error handling
- Streaming patterns

### [resources/concurrency-patterns.md](resources/concurrency-patterns.md)
- Goroutines and channels
- Worker pools
- Context usage
- Race condition prevention

### [resources/database-integration.md](resources/database-integration.md)
- GORM patterns
- Connection pooling
- Transaction management
- Migration strategies

### [resources/testing-strategies.md](resources/testing-strategies.md)
- Unit testing with mocks
- Integration testing
- Table-driven tests
- Benchmarking

### [resources/middleware-auth.md](resources/middleware-auth.md)
- Authentication middleware
- JWT validation
- Rate limiting
- CORS configuration

### [resources/observability.md](resources/observability.md)
- Structured logging (zap)
- Prometheus metrics
- Distributed tracing
- Health checks

### [resources/docker-kubernetes.md](resources/docker-kubernetes.md)
- Multi-stage Dockerfiles
- Kubernetes manifests
- Helm charts
- Health probes

## Best Practices

- Use context for cancellation and timeouts
- Handle errors explicitly, don't ignore
- Use interfaces for dependency injection
- Leverage Go's concurrency primitives
- Write table-driven tests
- Use structured logging
- Implement health and readiness endpoints
- Use migrations for database schema
- Version your APIs
- Document with godoc comments
- Use linters (golangci-lint)
- Follow effective Go guidelines

## Tools & Libraries

- **Web Frameworks**: Gin, Echo, Fiber
- **gRPC**: google.golang.org/grpc
- **Database**: GORM, sqlx
- **Logging**: zap, logrus
- **Config**: viper
- **Testing**: testify, gomock
- **Metrics**: prometheus/client_golang
- **Tracing**: OpenTelemetry

## Common Patterns

### Error Wrapping
```go
import "github.com/pkg/errors"

func (r *Repository) GetUser(id string) (*User, error) {
    user, err := r.db.FindByID(id)
    if err != nil {
        return nil, errors.Wrap(err, "failed to get user from database")
    }
    return user, nil
}
```

### Context Timeout
```go
func (uc *UseCase) Process(ctx context.Context) error {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()

    return uc.repo.SlowOperation(ctx)
}
```

---

**Status**: Production-Ready
**Last Updated**: 2025-11-04
**Performance**: Optimized for high-throughput, low-latency microservices
