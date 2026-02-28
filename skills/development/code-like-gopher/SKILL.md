---
name: code-like-gopher
description: Provides Go programming expertise, including language syntax, idiomatic patterns, concurrency, and standard library usage. Use when generating, analyzing, refactoring, or reviewing Go code.
---

## General Coding Approach

All the naming, commenting must be in **English**

When writing Go code, always check the Go version specified in the `go.mod`
file and use language features that are available for that version.

For example, starting with **Go 1.23**, range-over-integers (iterator-style
ranges) were introduced, making code like the following valid:

```go
for i := range 10 {
    fmt.Println(i)
}
```

Additionally, the long-standing **loop variable capture issue** in for loops has
been fixed in modern Go versions. Each iteration now gets its own copy of the
loop variable, so this pattern is safe and works as expected:

```go
for i := range 10 {
    go func() {
        fmt.Println(i)
    }()
}
```

## Formatting and Linting the Code

**Golangci-lint** is the defacto tool for go source code formatting, linting
and checking. First check if the tool is available:

```bash
command -v golangci-lint
```

if the `golangci-lint` doesn’t exist, offer to install:

```bash
brew install golangci-lint      # for macos + brew
```

Check if golangci-lint config file exists in the project, possible file names
are:

- `.golangci.yml`
- `.golangci.yaml`
- `.golangci.toml`
- `.golangci.json`

If the version of the config is not `"2"`, try to migrate configuration,
`golangci-lint` offers migrations via `golangci-lint migrate -h`

If config not available, here is your minimal `.golangci.yml` config example:

```yaml
version: "2"

run:
  timeout: 5m
  tests: false

linters:
  enable:
    - errcheck
    - govet
    - ineffassign
    - staticcheck
    - unused
    - misspell
    - unconvert
    - unparam
    - gosec
    - prealloc
    - revive
    - wrapcheck
  settings:
    govet:
      enable:
        - assign
        - appends
        - bools
        - defers
        - shadow
        - unmarshal
        - waitgroup
        - lostcancel
        - slog
        - unreachable
    errcheck:
      check-type-assertions: true
      exclude-functions:
        - fmt.Fprintln
        - fmt.Fprintf
    wrapcheck:
      ignore-package-globs:
        - encoding/*
        - github.com/pkg/*
    revive:
      enable-all-rules: true
      rules:
        - name: package-comments
          disabled: true
        - name: cognitive-complexity
          disabled: true
        - name: cyclomatic
          disabled: true
        - name: function-length
          disabled: true
        - name: use-waitgroup-go
          disabled: true
        - name: line-length-limit
          arguments: [120]
        - name: enforce-switch-style
          arguments: ["allowNoDefault"]
        - name: add-constant
          arguments:
            - max-lit-count: "3"
              allow-strs: '""'
              allow-ints: "0,1,2,10,64"
              allow-floats: "0.0,0.,1.0,1.,2.0,2."
        - name: comment-spacings
        - name: confusing-naming
        - name: datarace
        - name: context-as-argument
        - name: context-keys-type
        - name: deep-exit
        - name: defer
          arguments:
            - ["call-chain", "loop"]
        - name: duplicated-imports
        - name: early-return
          arguments:
            - "preserve-scope"
            - "allow-jump"
        - name: empty-block
        - name: empty-lines
        - name: error-naming
        - name: error-return
        - name: error-strings
        - name: errorf
        - name: exported
        - name: enforce-map-style
          arguments:
            - "make"
        - name: forbidden-call-in-wg-go
        - name: get-return
        - name: identical-branches
        - name: identical-ifelseif-branches
        - name: identical-ifelseif-conditions
        - name: identical-switch-branches
        - name: identical-switch-conditions
        - name: if-return
        - name: import-shadowing
        - name: increment-decrement
        - name: inefficient-map-lookup
        - name: modifies-parameter
        - name: modifies-value-receiver
        - name: optimize-operands-order
        - name: receiver-naming
        - name: redefines-builtin-id
        - name: redundant-import-alias
        - name: string-of-int
        - name: string-format
        - name: superfluous-else
        - name: time-date
        - name: time-equal
        - name: time-naming
        - name: unchecked-type-assertion
          arguments:
            - accept-ignored-assertion-result: true
        - name: unconditional-recursion
        - name: unexported-naming
        - name: unexported-return
        - name: unhandled-error
        - name: unnecessary-format
        - name: unreachable-code
        - name: unused-parameter
          arguments:
            - allow-regex: "^_"
        - name: unused-receiver
          arguments:
            - allow-regex: "^_"
        - name: use-errors-new
        - name: use-any
        - name: useless-break
        - name: var-declaration
        - name: var-naming
          arguments:
            - ["ID"] # AllowList
            - ["VM"] # DenyList
            - - skip-initialism-name-checks: true
                upper-case-const: true
                skip-package-name-checks: true
                skip-package-name-collision-with-go-std: true
                extra-bad-package-names:
                  - helpers
                  - utils
                  - tools
                  - models

formatters:
  enable:
    - gofmt
    - gofumpt
    - goimports
    - golines
  settings:
    golines:
      max-len: 120
```

All the details of the configuration file can be found here:
https://github.com/golangci/golangci-lint/blob/HEAD/.golangci.reference.yml

- `gofmt` is your friend, use for formatting the `*.go` files
- `goimports` helps you to sort and find required package imports

Help is available:

```bash
golangci-lint help formatters
golangci-lint help linters

golangci-lint fmt dir1 dir2/...
golangci-lint fmt file1.go
```

## Coding Style

### Naming Conventions

Variable names should describe **the value they hold**, not **the type of the variable**.
Incorrect (bad) examples include:

```go
// Bad
var userString string
var countInt int
var usersMap     map[string]*User
var companiesMap map[string]*Company
var productsMap  map[string]*Product
var usersList    []User

// Good
var username string
var count int

var users     map[string]*User
var users     []User

var companies map[string]*Company
var companies []Company
var products  []Product
```

Use predictable and easily understandable names:

- Use short variable names like `i`, `j`, `k` in `for` loops
- Use `n` when representing a counter, total, or quantity
- In maps, use `k` for keys and `v` for values
- Use `a`, `b` for variables of the same type (e.g. during comparisons); their
  positions may be interchangeable
- Use `x`, `y` as conventional names for locally scoped variables created for
  comparisons
- Use `s` as a common shorthand for string values
- Collections (`map`, `slice`, `array`) should always use **plural** names

Functions should be named according to the result they return.

- A function name must start with a letter; it cannot start with a number and
  must not contain spaces
- Exported functions must start with an uppercase letter and **must be
  documented with a comment**
- Function names are case-sensitive

Examples:

```go
func Add(a, b) int {}
// describes only the operation

// this is better
func Sum(a, b) int {}
// returned thing is a sum of a and b...
// this describes the result, not the operation...
```

Method names should be named to describe the **action they perform**. This is
the opposite of function naming:

```go
package main

import "fmt"

type user struct {
	email    string
	password string
	fullName string
}

// Email is a getter for user.email
func (u user) Email() string {
	return u.email
}

// SetEmail is a setter for user.email
func (u *user) SetEmail(email string) {
	u.email = email
}

// resetPassword resets user's password
func (u *user) resetPassword() error {
	fmt.Println("example reset password")
	u.password = "reset"
	return nil
}

func main() {
	u := &user{}
	u.SetEmail("vigo@me.com")
	u.resetPassword()

	fmt.Println("email", u.Email())
	fmt.Printf("%+v\n", u)
}
```

### Package Names

When a package is imported, the package name becomes an accessor for the
contents. By convention, packages are given lower case, single-word names;
there should be no need for underscores or mixedCaps.

Another convention is that the package name is the base name of its source
directory; the package in `src/encoding/base64` is imported as "encoding/base64"
but has name `base64`, not `encoding_base64` and not `encodingBase64`.

Don’t use the import . notation, which can simplify tests that must run
outside the package they are testing, but should otherwise be avoided.

For instance, the buffered reader type in the `bufio` package is called `Reader`,
not `BufReader`, because users see it as `bufio.Reader`, which is a clear, concise
name. 

Moreover, because imported entities are always addressed with their package
name, `bufio.Reader` does not conflict with `io.Reader`. Similarly, the function
to make new instances of ring.Ring—which is the definition of a constructor in
Go—would normally be called NewRing, but since `Ring` is the only type **exported**
by the package, and since the package is called `ring`, it's called just **New**,
which clients of the package see as `ring.New`. Use the package structure to
help you choose good names.

Go doesn’t provide automatic support for **getters** and **setters**. There’s
nothing wrong with providing getters and setters yourself, and it’s often
appropriate to do so, but it's neither idiomatic nor necessary to put **Get** into
the getter's name. 

If you have a field called `owner` (lower case, unexported), the getter method
should be called `Owner` (upper case, exported), **not GetOwner**. The use of
upper-case names for export provides the hook to discriminate the field from
the method. A setter function, if needed, will likely be called `SetOwner`. Both
names read well in practice:

```go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

Abbreviate judiciously. Package names may be abbreviated when the abbreviation
is familiar to the programmer. Widely-used packages often have compressed
names:

    strconv (string conversion)
    syscall (system call)
    fmt (formatted I/O)

**Don’t steal good names from the user**. Avoid giving a package a name that is
commonly used in client code. For example, the buffered I/O package is called
**bufio**, not **buf**, since buf is a good variable name for a buffer.

**Avoid repetition**. Since client code uses the package name as a prefix when
referring to the package contents, the names for those contents need not
repeat the package name. The HTTP server provided by the `http` package is
called `Server`, not **HTTPServer**. Client code refers to this type as
`http.Server`, so there is no ambiguity.

Simplify function names. When a function in package `pkg` returns a value of
`type pkg.Pkg` (or `*pkg.Pkg`), the function name can often omit the type name
without confusion:

```go
start := time.Now()                                  // start is a time.Time
t, err := time.Parse(time.Kitchen, "6:06PM")         // t is a time.Time
ctx = context.WithTimeout(ctx, 10*time.Millisecond)  // ctx is a context.Context
ip, ok := userip.FromContext(ctx)                    // ip is a net.IP
```

A function named `New` in package `pkg` returns a value of type pkg.Pkg. This is a
standard entry point for client code using that type:

```go
q := list.New()  // q is a *list.List
```

Write code that uses your package as a client would, and restructure your
packages if the result seems poor. This approach will yield packages that are
easier for clients to understand and for the package developers to maintain.

Avoid meaningless package names. Packages named `util`, `common`, or `misc`
provide clients with no sense of what the package contains. This makes it
harder for clients to use the package and makes it harder for maintainers to
keep the package focused.

Over time, they accumulate dependencies that can make compilation
significantly and unnecessarily slower, especially in large programs. And
since such package names are generic, they are more likely to collide with
other packages imported by client code, forcing clients to invent names to
distinguish them.

Bad package name example:

```go
package util
func NewStringSet(...string) map[string]bool {...}
func SortStringSet(map[string]bool) []string {...}

set := util.NewStringSet("c", "a", "b")
fmt.Println(util.SortStringSet(set))
```

Good package name example:

```go
package stringset
func New(...string) map[string]bool {...}
func Sort(map[string]bool) []string {...}

set := stringset.New("c", "a", "b")
fmt.Println(stringset.Sort(set))

// Once you’ve made this change, it’s easier to see how to improve the new package:

package stringset
type Set map[string]bool
func New(...string) Set {...}
func (s Set) Sort() []string {...}

set := stringset.New("c", "a", "b")
fmt.Println(set.Sort())
```

Don’t use a single package for all your APIs. Many well-intentioned
programmers put all the interfaces exposed by their program into a single
package named `api`, `types`, or `interfaces`, thinking it makes it easier to find
the entry points to their code base. This is a mistake. Such packages suffer
from the same problems as those named util or common, growing without bound,
providing no guidance to users, accumulating dependencies, and colliding with
other imports. Break them up, perhaps using directories to separate public
packages from implementation.

### Interface names

By convention, one-method interfaces are named by the method name plus an **-er**
suffix or similar modification to construct an agent noun: `Reader`, `Writer`,
`Formatter`, `CloseNotifier` etc.

```go
type Stringer interface {
	String() string
}

type Reader interface {
	Read(p []byte) (n int, err error)
}

type Writer interface {
	Write(p []byte) (n int, err error)
}

// FooBarBazer :)
type ReadSeekCloser interface {
	Reader
	Seeker
	Closer
}
```

Finally, the convention in Go is to use `MixedCaps` or `mixedCaps` rather than
**underscores** to write multiword names.

### Syntactic Sugars and Techniques

- Try to avoid Naked Returns / Named Returns, instead of `func sum(a, b int) (result int)`
  use : `func sum(a, b int) (int)`
- Embrace early exit approach
- Instead of empty interface `interface{}` use `any`.
- Compile time proof and interface checks are important
- Make the zero value useful, stdlib’s `bytes.Buffer`, `sync.Mutex` approach
- Errors are values, use custom error types, `errors.As` and `errors.Is` your friend
- Every piece of code should be testable. Keep this in mind, use `interface`
  approach to mock/dependency inject your types, functions etc...
- Try not to use generics, generics should be your last option!
- When creating structs, keep **field set alignment** in your mind, order your
  fields properly, (golangci-linter has a linter - revive check field set alignment)

    type                                 size in bytes

    byte, uint8, int8                     1
    uint16, int16                         2
    uint32, int32, float32                4
    uint64, int64, float64, complex64     8
    complex128                           16

- Never hardcode the sql variables in a query, `database/sql` package provides lots
  of functions.
- Every type in go has a zero value never `var i int = 0` use `var i int`
- Keep pointer or value semantics in struct methods, if you don’t need to modify
  the data in struct, keep value semantics.
- Never put `context.Context` in struct field
- In Go, `context.Context` must always be passed as the **first argument** to 
  a function or method.

```go
// Bad
func FetchUser(id string, ctx context.Context) (*User, error) {
    // ...
}

// Good
func FetchUser(ctx context.Context, id string) (*User, error) {
    // ...
}
```

Don’t just check errors, handle them gracefully!

```go
// bad error handling
func AuthenticateRequest(r *Request) error {
	err := authenticate(r.User)
	if err != nil {
		return err // ??? who fired this error?
	}
	return nil
}

// good error handling
func AuthenticateRequest(r *Request) error {
	err := authenticate(r.User)
	if err != nil {
		return fmt.Errorf("authenticate failed: %v", err) // wrap errors with an extra message
	}
	return nil
}


// bad example
func Write(w io.Writer, buf []byte) error {
	_, err := w.Write(buf)
	if err != nil {
		// annotated error goes to log file
		log.Println("unable to write:", err)

		// unannotated error returned to caller
		return err
	}
	return nil
}

// good example
func Write(w io.Write, buf []byte) error {
	_, err := w.Write(buf)
	return errors.Wrap(err, "write failed") // <-- Wrap method from github.com/pkg/errors package
}
```

Use functional options pattern instead of config structs:

```go
// Option type definition
type Option func(*Server) error

// WithXxx functions - include validation
func WithLogger(l *slog.Logger) Option {
    return func(s *Server) error {
        if l == nil {
            return fmt.Errorf("[server.WithLogger] error: %w", ErrNilLogger)
        }
        s.Logger = l
        return nil
    }
}

func WithPort(port string) Option {
    return func(s *Server) error {
        if port == "" {
            return fmt.Errorf("[server.WithPort] error: %w", ErrEmptyPort)
        }
        s.Port = port
        return nil
    }
}

// Constructor - defaults + options + validation
func New(options ...Option) (*Server, error) {
    server := new(Server)

    // Set defaults
    server.Port = "8080"

    // Apply options
    for _, option := range options {
        if err := option(server); err != nil {
            return nil, err
        }
    }

    // Validate required fields
    if err := server.validate(); err != nil {
        return nil, err
    }

    return server, nil
}
```

### Test Conventions

Naming tests to self-document; instead of:

```go
func TestTitleIllegalChar(t *testing.T) {}
```

Use:

```go
func TestTitleEscape(t *testing.T) {}
```

With this rename, we also self-document how the illegal characters on the
title will be handled.

Parallelize your table-driven tests;

```go
func TestFoo(t *testing.T) {
	tc := []struct {
		dur time.Duration
	}{
		{time.Second},
		{2 * time.Second},
		{3 * time.Second},
		{4 * time.Second},
	}
	for _, tt := range tc {
		tt := tt
		t.Run("", func(st *testing.T) {
			st.Parallel()
			time.Sleep(tt.dur)
		})
	}
}
```

### Concurrency

**REMEMBER! THE GOLDEN RULE**

If you are the receiver (i.e. reading from a channel using `<-ch`), **never close the channel**.
Only the sender (the one writing to the channel using `ch <- value`) should close it.

The goroutine that closes a channel must be the one that **knows when the work is finished**. 
In this case, the sender function (e.g. `count()`) knows exactly when the loop ends — 
therefore, it is the one responsible for closing the channel.

The sender always knows how many goroutines are writing to the channel, but the receiver does not. 
For this reason, **a channel must always be closed by the sender, never by the receiver**.

Always keep this in mind, be concurrent safe, use `sync.Map` if you need concurrent
safe maps, way better that custom mutex guards.

When possible, prefer using a buffered channel with capacity `1` to avoid
unnecessary blocking and reduce tight synchronization between goroutines.

```go
// Unbuffered (can cause unnecessary blocking)
ch := make(chan int)

// Preferred when only a signal or single value is needed
ch := make(chan int, 1)

// use cases
// signal / done channel
// error propagation
// goroutine lifecycle coordination
```

---

## Pre-Commit Hooks

If pre-commit config file `.pre-commit-config.yaml` doesn’t exists, ask user to
install pre-commit:

```bash
brew install pre-commit
```

and install hooks:

```bash
pre-commit install
```

Use this minimum config:

```yaml
repos:
  - repo: https://github.com/TekWizely/pre-commit-golang
    rev: v1.0.0-rc.1
    hooks:
      - id: golangci-lint-mod
      - id: go-mod-tidy
      - id: go-test-mod
```

## Commit Message Guidelines

- Prefix: `[claude-opus]:` or `[claude-sonnet]:` based on model
- Short summary in lowercase (max 50 chars)
- Blank line, then bullet points with details
- Include Claude Code footer
- Use **present tense** in commit messages.
- Always start the message with a **lowercase letter**.
- Start with a **verb**, followed by a brief and clear description.

Examples:

- `fix login redirect issue`
- `implement user profile page`
- `remove unused dependencies`

**If related to a GitHub issue**:

- Add `Fixes #ISSUE-NUMBER` or `Closes #ISSUE-NUMBER` at the end of the commit
  message. This auto closes issue on GitHub.
- Include a direct link to the related GitHub issue.

Also this commit-template is helpful:

    #
    # 3456789x123456789x123456789x123456789x123456789x
    # Short description (subject) : 50 chars

    # 3456789x123456789x123456789x123456789x123456789x123456789x123456789x12
    # Long description : 72 chars
    #
    # - Why was this change necessary?
    # - How does it address the problem?
    # - Are there any side effects?
    #
    # Fixes #ticket
    # Closes #ticket, #ticket, #ticket
    #
    # Include a link to the ticket, if any.
    #

Example:

    [claude-opus]: add TXT and DOCX resume upload support
    
    - Convert TXT/DOCX files to PDF on upload using fpdf2 and python-docx
    - Skip text extraction for pre-extracted content
    - Add Unicode font support for Turkish characters
    
    🤖 Generated with [Claude Code](https://claude.com/claude-code)
    
    Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>

---

## Resources

- https://go.dev/ref/spec
- https://go.dev/doc/effective_go
- https://rakyll.org/style-packages/
- https://rakyll.org/typesystem/
- https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

