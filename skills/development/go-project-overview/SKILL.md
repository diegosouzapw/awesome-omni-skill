---
name: go-project-overview
description: Overview of Go project structure, technology stack (Echo, Connect RPC, Driver pattern), and essential commands for building, testing, and development
---

# Go Project Overview

## Technology Stack

### HTTP Framework
- **Echo v4** (`github.com/labstack/echo/v4`)
- Middleware: Recover, CORS, Logger, RequestID
- Pattern: `func(c echo.Context) error`

### RPC Framework
- **Connect RPC + gRPC-Gateway**
- Tool: `buf` for proto management
- **Dual Protocol**: Single impl serves both

### Database
- **Driver Interface Pattern** (`store/driver.go`)
- Drivers: SQLite (`modernc.org/sqlite`), MySQL (`go-sql-driver/mysql`), PostgreSQL (`lib/pq`)
- Caching: TTL 10min, max 1000 items

### Authentication
- JWT access token (stateless, 15min)
- Refresh token (long-lived, db-validated)
- PAT (personal access token, SHA-256 hash)
- Header: `Authorization: Bearer <token>`

## Essential Commands

### Backend
```bash
# Run dev server on port 8081
go run ./cmd/server --port 8081

# All tests
go test ./...

# Single test
go test -run TestName ./path/...

# Test with coverage
go test -cover ./...

# Lint
golangci-lint run

# Format imports
goimports -w .
```

### Protocol Buffers
```bash
# Install dependencies
cd proto && buf dep update

# Generate code
cd proto && buf generate

# Lint proto files
cd proto && buf lint

# Breaking changes
cd proto && buf breaking --against .git#main
```

### Multi-Database Testing
```bash
# SQLite (default)
DRIVER=sqlite go test ./...

# MySQL
DRIVER=mysql DSN="user:pass@tcp(localhost:3306)/db" go test ./...

# PostgreSQL
DRIVER=postgres DSN="postgres://user:pass@localhost:5432/db?sslmode=disable" go test ./...
```

## Project Structure

```
cmd/              # Entry point (main.go with Cobra CLI)
server/           # HTTP/gRPC server
  auth/           # Authentication logic
  router/         # API routing
    api/v1/       # Connect + gRPC-Gateway handlers
    fileserver/   # Static file serving
    frontend/     # SPA serving
    rss/          # RSS feeds
  runner/         # Background tasks
store/            # Data layer
  db/{driver}/    # Database implementations
  cache/          # In-memory cache
  migration/      # Schema migrations
  seed/           # Demo data
  test/           # Test utilities
proto/            # Protocol buffers
  api/v1/         # Service definitions
  google/         # Google API protos
plugin/           # Extensible features
internal/         # Internal utilities
```

## Key Files

| File | Purpose |
|------|---------|
| `cmd/server/main.go` | CLI entry point |
| `internal/profile/profile.go` | Configuration struct |
| `server/server.go` | Server initialization, cmux |
| `server/router/api/v1/v1.go` | Service registration |
| `store/driver.go` | Database interface |
| `store/store.go` | Store wrapper with caching |
| `proto/buf.yaml` | Proto lint config |
| `proto/buf.gen.yaml` | Code generation config |

## Skill Map

| Task | Skill |
|------|-------|
| CLI, Config, Shutdown | [go-project-main](../go-project-main/) |
| Echo, Connect, cmux | [go-project-server](../go-project-server/) |
| Proto definitions | [go-project-proto](../go-project-proto/) |
| Database, Migrations | [go-project-store](../go-project-store/) |
| Code style | [go-project-conventions](../go-project-conventions/) |

## Workflows

### ⚠️ CRITICAL: Development Order

**You MUST create files in this exact order:**

```
1. proto/api/v1/*_service.proto    ← DEFINE API FIRST
2. proto/buf.yaml
3. proto/buf.gen.yaml
4. cd proto && buf generate        ← GENERATE CODE
5. server/router/api/v1/*_service.go  ← IMPLEMENT SERVICE
6. store/driver.go
7. store/{model}.go
8. server/router/api/v1/v1.go      ← REGISTER SERVICE
9. server/server.go
10. cmd/server/main.go
```

**If you skip step 1-4, the server code will not compile!**

### Add API Endpoint (Detailed Steps)

**Step 1: Create proto definition** (see go-project-proto)
```bash
mkdir -p proto/api/v1
```

Create `proto/api/v1/auth_service.proto`:
```protobuf
syntax = "proto3";

package myapp.api.v1;

import "google/api/annotations.proto";
import "google/api/client.proto";
import "google/api/field_behavior.proto";
import "google/protobuf/empty.proto";

option go_package = "gen/api/v1";

service AuthService {
  rpc SignIn(SignInRequest) returns (SignInResponse) {
    option (google.api.http) = {
      post: "/api/v1/auth/signin"
      body: "*"
    };
  }

  rpc SignUp(SignUpRequest) returns (SignUpResponse) {
    option (google.api.http) = {
      post: "/api/v1/auth/signup"
      body: "*"
    };
  }
}

message SignInRequest {
  string username = 1 [(google.api.field_behavior) = REQUIRED];
  string password = 2 [(google.api.field_behavior) = REQUIRED];
}

message SignInResponse {
  string access_token = 1;
  string refresh_token = 2;
}

message SignUpRequest {
  string username = 1 [(google.api.field_behavior) = REQUIRED];
  string email = 2 [(google.api.field_behavior) = REQUIRED];
  string password = 3 [(google.api.field_behavior) = REQUIRED];
}

message SignUpResponse {
  User user = 1;
}

message User {
  string name = 1 [(google.api.field_behavior) = IDENTIFIER];
  string username = 2 [(google.api.field_behavior) = REQUIRED];
  string email = 3 [(google.api.field_behavior) = REQUIRED];
}
```

**Step 2: Create buf config**
```bash
# proto/buf.yaml
version: v2
deps:
  - buf.build/googleapis/googleapis
lint:
  use: [BASIC]
breaking:
  use: [FILE]

# proto/buf.gen.yaml
version: v2
plugins:
  - remote: buf.build/protocolbuffers/go
    out: gen
    opt: paths=source_relative
  - remote: buf.build/grpc/go
    out: gen
    opt: paths=source_relative
  - remote: buf.build/connectrpc/go
    out: gen
    opt: paths=source_relative
  - remote: buf.build/grpc-ecosystem/gateway
    out: gen
    opt: paths=source_relative
```

**Step 3: Generate code**
```bash
cd proto && buf generate
```

**Step 4: Implement service** (see go-project-server)
Create `server/router/api/v1/auth_service.go`:
```go
package v1

type AuthService struct {
    Store   *store.Store
    Profile *profile.Profile
}

func (s *AuthService) SignIn(ctx context.Context, req *connect.Request[v1pb.SignInRequest]) (*connect.Response[v1pb.SignInResponse], error) {
    // Implementation here
}
```

**Step 5: Register service** (see go-project-server)
Add to `server/router/api/v1/v1.go`:
```go
func (s *APIV1Service) RegisterGateway(ctx context.Context, echoServer *echo.Echo) error {
    // ... existing code
    v1pb.RegisterAuthServiceHandlerServer(ctx, gwMux, s)
    return nil
}
```

### Add Database Model (Detailed Steps)

**Step 1: Create migration** (see go-project-store)
```bash
mkdir -p store/migration/sqlite/0.1
```

Create `store/migration/sqlite/0.1/00__create_user.sql`:
```sql
CREATE TABLE user (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    created_ts BIGINT NOT NULL DEFAULT (strftime('%s', 'now')),
    updated_ts BIGINT NOT NULL DEFAULT (strftime('%s', 'now')),
    username TEXT NOT NULL UNIQUE,
    email TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL
);
```

**Step 2: Update LATEST.sql**

**Step 3: Update store/driver.go interface**

**Step 4: Implement CRUD in store/db/sqlite/*.go**

## AI Configuration Files

Create these files for AI assistant context:

### AGENTS.md (通用)
```markdown
# Project Guide for AI Agents

## Technology Stack
- **Echo v4** - HTTP framework
- **Connect RPC + gRPC-Gateway** - Dual protocol API
- **buf** - Proto management
- **Driver Pattern** - SQLite/MySQL/PostgreSQL

## Essential Commands

\`\`\`bash
# Dev server
go run ./cmd/server --port 8081

# Tests
go test ./...
go test -run TestName ./path/...

# Lint
golangci-lint run

# Proto
cd proto && buf generate && buf lint
\`\`\`

## Code Conventions

### Error Handling
- Use `errors.Wrap(err, "context")` from `github.com/pkg/errors`
- gRPC errors: `status.Errorf(codes.NotFound, "message")`
- Never use `fmt.Errorf`

### Imports
- Group: stdlib, third-party, local
- Sort alphabetically within groups
- Use `goimports -w .`
```

### .cursor/rules/go-project.rule (Cursor)
```markdown
# Go Project Rules

## Architecture
- Echo v4 for HTTP
- Connect RPC + gRPC-Gateway
- Driver interface pattern for databases

## Key Patterns
- Store wraps Driver, adds caching
- Service implementations serve both Connect and gRPC-Gateway
- Background runners with context cancellation

## Commands
- Dev: `go run ./cmd/server --port 8081`
- Test: `go test ./...`
- Lint: `golangci-lint run`
```

### .github/copilot-instructions.md (Copilot)
```markdown
# Go Project Guidelines

This project uses:
- Echo v4 web framework
- Connect RPC + gRPC-Gateway for APIs
- buf for protocol buffers
- Driver pattern for multi-database support (SQLite/MySQL/PostgreSQL)

When generating code:
- Use `errors.Wrap(err, "context")` for error handling
- Group imports: stdlib, third-party, local
- Implement services once, serve both Connect and gRPC-Gateway
```

## File Templates

### .gitignore
```
# Generated by go mod init
go.mod
go.sum

# Binary
cmd/server/server
cmd/server/server.exe

# IDE
.idea/
.vscode/
*.swp
*.swo

# Data
*.db
*.sqlite
data/

# Proto generated
proto/gen/
web/src/types/proto/
```

### .golangci.yaml
```yaml
run:
  timeout: 5m
  issues-exit-code: 1

linters:
  enable:
    - revive
    - govet
    - staticcheck
    - misspell
    - gocritic
    - sqlclosecheck
    - rowserrcheck
    - godot

issues:
  exclude-rules:
    - path: _test\.go
      linters:
        - dupl
```
