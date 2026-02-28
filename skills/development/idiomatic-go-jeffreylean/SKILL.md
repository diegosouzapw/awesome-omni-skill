---
name: idiomatic-go
description: Write production-ready Go backends, CLIs, and APIs following modern best practices from top tier tech companies. Use this skill when creating or reviewing Go code for (1) backend services and APIs, (2) command-line tools, (3) code requiring proper error handling, concurrency, or testing patterns, (4) any Go development requiring adherence to established style guidelines. Includes comprehensive linting configuration and detailed style guide.
version: 1.0.1
---

# Go Development

Write Go code that is readable, maintainable, and production-ready using battle-tested patterns from major production codebases.

## Quick Decision Trees

### MCP

Always use Context7 MCP to fetch the latest documentation.

### Libraries

- Prefer to use libraries that are well-maintained and have a large community.
- Prefer zero-dependency libraries.
- Prefer libraries that are present in the awesome-go list.
- For HTTP services, use [Chi](https://github.com/go-chi/chi) for routing.
- For logging, use slog.Logger for logging.
- For configuration use flags or environment variables.

### Linters

- golangci-lint is the best linter for Go. It is a comprehensive linter that checks for many issues in the code.
- It is a good idea to run golangci-lint on every commit.
- It is a good idea to run golangci-lint on every pull request.
- It is a good idea to run golangci-lint on every code review.
- It is a good idea to run golangci-lint on every code review.

### Formatting

- goimports is the best formatter for Go. It is a simple formatter that formats the code according to the Go language specification.
- It is a good idea to run goimports on every commit.
- It is a good idea to run goimports on every pull request.
- It is a good idea to run goimports on every code review.
- It is a good idea to run goimports on every code review.

### Testing

- Table-driven tests should follow the pattern of:

```go
func TestProcess(t *testing.T) {
    type testCase struct {
        // Fields for the test case.
    }

    tests := map[string]testCase{
        "name": {
            // Test case fields.
        },
        // More test cases.
    }

    for name, tc := range tests {
        t.Run(name, func(t *testing.T) {
            // Test code.
        })
    }
}
```

- Integration tests should be skipped if the environment variables are not set.

```go
func TestIntegration(t *testing.T) {
    if os.Getenv("INTEGRATION_TESTS") == "" {
        t.Skip("skipping integration tests")
    }
    // Test code.
}
```

- Test helpers should call `t.Helper()` so failure line numbers point to the actual test.

```go
func TestHelper(t *testing.T) {
    t.Helper()
    // Test code.
}
```

### When to use interfaces?

**Define interfaces at consumption site, not implementation:**

```go
// GOOD: Consumer defines what it needs
package storage

type Store interface {
    Get(key string) ([]byte, error)
}

// BAD: Implementation forces interface on consumers
package postgres

type PostgresStore interface { ... }
```

**Interface size:**

- 1 method: Perfect (Reader, Writer, Stringer)
- 2-3 methods: Good if cohesive
- 4+ methods: Consider splitting or using concrete types
- It is ok to have big interfaces for saas products or enterprice software products. For libraries, it is better to have small interfaces.

**Accept interfaces, return concrete types:**

```go
// GOOD
func Process(r io.Reader) (*Result, error)

// BAD: Forces caller to deal with interface
func Process(r io.Reader) (io.Reader, error)
```

### How to handle errors?

**Decision tree:**

1. Can I handle this error completely here? → Log and continue
2. Does caller need programmatic access? → Use `%w` wrapping
3. Should I hide implementation details? → Use `%v` wrapping
4. Is this a library? → Never log, always return

```go
// Handle completely
if err != nil {
    log.Printf("retrying with defaults: %v", err)
    return useDefaults(), nil
}

// Caller needs access (use %w)
if err != nil {
    return fmt.Errorf("connect to database: %w", err)
}

// Hide details (use %v)
if err != nil {
    return fmt.Errorf("service unavailable: %v", err)
}
```

**Error string format:**

- Lowercase, no punctuation
- Avoid "failed to" or "error" prefix
- Add context: `"operation: %w"`

### When to use concurrency?

**Leave concurrency to the caller unless:**

- You're building a server/daemon that must handle concurrent requests
- You're implementing a worker pool pattern
- You're managing background operations (cleanup, metrics)

```go
// GOOD: Synchronous by default
func Fetch(url string) (*Response, error)

// Caller decides concurrency
go fetch(url)

// BAD: Forces async on everyone
func FetchAsync(url string) <-chan *Response
```

**Before launching a goroutine, know when it will stop:**

```go
// GOOD: Clear lifecycle
ctx, cancel := context.WithTimeout(ctx, 30*time.Second)
defer cancel()

go func() {
    for {
        select {
        case <-ctx.Done():
            return // goroutine stops here
        case work := <-ch:
            process(work)
        }
    }
}()
```

### Context as first parameter?

**Always use context when:**

- Making external calls (HTTP, DB, RPC)
- Operations may be cancelled
- Deadlines matter
- Need to pass request-scoped values

```go
// GOOD
func Query(ctx context.Context, sql string) (*Rows, error)

// BAD: Can't be cancelled
func Query(sql string) (*Rows, error)
```

## Common Workflows

### Creating a new HTTP service

**1. Project structure:**

```text
myservice/
├── cmd/
│   └── server/
│       └── main.go          # Binary entrypoint
├── internal/
│   ├── handler/             # HTTP handlers
│   │   ├── handler.go
│   │   └── handler_test.go
│   ├── service/             # Business logic
│   │   ├── service.go
│   │   └── service_test.go
│   └── storage/             # Data layer
│       ├── postgres.go
│       └── postgres_test.go
├── go.mod
├── go.sum
├── Makefile
└── .golangci.yml
```

**2. Initialize project:**

```bash
mkdir -p myservice/{cmd/server,internal/{handler,service,storage}}
cd myservice
go mod init github.com/yourorg/myservice

# Setup linting
/path/to/scripts/setup_golangci_lint.sh .
```

**3. Main.go pattern:**

```go
package main

import (
    "context"
    "flag"
    "log"
    "net/http"
    "os"
    "os/signal"
    "time"
)

func main() {
    // Flags only in main
    addr := flag.String("addr", ":8080", "listen address")
    timeout := flag.Duration("timeout", 30*time.Second, "request timeout")
    flag.Parse()

    // Initialize dependencies
    srv := &http.Server{
        Addr:         *addr,
        Handler:      setupRoutes(),
        ReadTimeout:  *timeout,
        WriteTimeout: *timeout,
    }

    // Graceful shutdown
    go func() {
        sigint := make(chan os.Signal, 1)
        signal.Notify(sigint, os.Interrupt)
        <-sigint

        ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
        defer cancel()

        if err := srv.Shutdown(ctx); err != nil {
            log.Printf("shutdown error: %v", err)
        }
    }()

    log.Printf("listening on %s", *addr)
    if err := srv.ListenAndServe(); err != http.ErrServerClosed {
        log.Fatalf("server error: %v", err)
    }
}
```

**4. Handler pattern:**

```go
package handler

import (
    "encoding/json"
    "net/http"
)

type Handler struct {
    service Service
}

func New(svc Service) *Handler {
    return &Handler{service: svc}
}

func (h *Handler) HandleGet(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    
    // Extract params
    id := r.URL.Query().Get("id")
    if id == "" {
        http.Error(w, "missing id", http.StatusBadRequest)
        return
    }

    // Call service
    result, err := h.service.Get(ctx, id)
    if err != nil {
        // Log and return appropriate status
        http.Error(w, "internal error", http.StatusInternalServerError)
        return
    }

    // Respond
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(result)
}
```

### Creating a CLI tool

**1. Structure:**

```text
mycli/
├── main.go              # Flag parsing and dispatch
├── internal/
│   └── command/
│       ├── run.go       # Command implementations
│       └── run_test.go
├── go.mod
└── .golangci.yml
```

**2. Main.go with subcommands:**

```go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    if len(os.Args) < 2 {
        fmt.Fprintf(os.Stderr, "usage: %s <command> [flags]\n", os.Args[0])
        os.Exit(1)
    }

    switch os.Args[1] {
    case "process":
        processCmd := flag.NewFlagSet("process", flag.ExitOnError)
        input := processCmd.String("input", "", "input file")
        processCmd.Parse(os.Args[2:])
        
        if err := runProcess(*input); err != nil {
            fmt.Fprintf(os.Stderr, "error: %v\n", err)
            os.Exit(1)
        }

    default:
        fmt.Fprintf(os.Stderr, "unknown command: %s\n", os.Args[1])
        os.Exit(1)
    }
}
```

### Adding comprehensive tests

**1. Table-driven test pattern:**

```go
func TestProcess(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    string
        wantErr bool
    }{
        {
            name:    "valid input",
            input:   "hello",
            want:    "HELLO",
            wantErr: false,
        },
        {
            name:    "empty input",
            input:   "",
            want:    "",
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Process(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("Process() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if got != tt.want {
                t.Errorf("Process() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

**2. Test helper pattern:**

```go
func TestHandler(t *testing.T) {
    h := setupHandler(t)  // Helper creates handler
    
    req := newRequest(t, "GET", "/api/test")  // Helper creates request
    rr := httptest.NewRecorder()
    
    h.ServeHTTP(rr, req)
    
    assertStatus(t, rr.Code, http.StatusOK)  // Helper asserts
    assertBody(t, rr.Body.String(), "expected")
}

func setupHandler(t *testing.T) http.Handler {
    t.Helper()  // Marks this as test helper
    // Setup code
}
```

## Detailed Reference

For comprehensive coverage of all Go idioms, patterns, and best practices:

**Read `references/go-styleguide.md` for:**

- Complete naming conventions (packages, variables, interfaces, constants)
- Code organization principles (when to create packages, file structure)
- Error handling patterns (wrapping, checking, panics)
- Concurrency patterns (goroutines, channels, context, waitgroups)
- Interface design (when/where to define, sizes, embedded interfaces)
- Testing patterns (table-driven, subtests, helpers, mocks)
- Performance considerations (allocations, profiling, benchmarks)
- Critical pitfalls to avoid (loop variables, nil interfaces, defer in loops)

## Linting Setup

**Run the setup script:**

```bash
scripts/setup_golangci_lint.sh /path/to/your/project
```

This configures comprehensive linting including:

- Error checking (errcheck, errorlint)
- Security analysis (gosec)
- Style enforcement (revive, gocritic)
- Performance checks (prealloc, perfsprint)
- Code quality (gocyclo, gocognit, staticcheck)

**Common commands:**

```bash
# Run all linters
golangci-lint run

# Auto-fix issues
golangci-lint run --fix

# Lint specific paths
golangci-lint run ./internal/...
```

## Quick Reference Cheatsheet

**Naming:**

- Packages: lowercase, singular, no underscores
- Getters: `obj.Owner()` not `obj.GetOwner()`
- Acronyms: consistent case (`URL` or `url`, never `Url`)

**Error handling:**

- Check immediately after call
- Wrap with context: `fmt.Errorf("operation: %w", err)`
- Handle exactly once: log OR return, not both
- Never panic in libraries

**Concurrency:**

- Context as first parameter
- Know when every goroutine stops
- Use `sync.WaitGroup` for coordination
- Don't force concurrency on callers

**Structure:**

- Return early with guard clauses
- Keep success path left-aligned
- Import groups: stdlib, external, internal

**Testing:**

- Table-driven with named fields
- Use `t.Run()` for subtests
- Call `t.Helper()` in helpers
- Message format: `got X, want Y`

**Critical pitfalls:**

- Loop variable capture: pass to closure explicitly
- Nil interface check: interface with nil value ≠ nil
- Defer in loops: wrap in closure
- Map writes to nil: always `make()` first
