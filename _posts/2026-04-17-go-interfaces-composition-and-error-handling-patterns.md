---

layout: post
title: "Go Interfaces, Composition, and Error Handling Patterns"
date: 2026-04-17 07:34:24 +0000
tags: [go, interfaces, composition, error-handling, concurrency]
categories: go
---

# Go Interfaces, Composition, and Error Handling Patterns

This post collects several core Go design patterns that show up across the standard library and production services. Together they explain much of Go’s philosophy: small interfaces, explicit composition, straightforward dependency wiring, and practical concurrency.

---

## Small Interfaces + Implicit Interface Satisfaction

Go prefers tiny, behavior-focused interfaces.

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

Small interfaces improve:

* testability
* composability
* decoupling
* clarity at API boundaries

Instead of describing what a type *is*, Go interfaces describe what a type *can do*.

### Example

```go
type Logger interface {
    Log(msg string)
}

type ConsoleLogger struct{}

func (c ConsoleLogger) Log(msg string) {
    fmt.Println(msg)
}

func Process(l Logger) {
    l.Log("processing...")
}
```

The function depends only on the behavior it needs.

### Implicit Satisfaction

In Go, a type satisfies an interface automatically if it has the required methods.

```go
type FileLogger struct{}

func (f FileLogger) Log(msg string) {
    // write to file
}
```

`FileLogger` now satisfies `Logger` without any extra declaration.

### Key Insight

> Define interfaces at the point of use, not implementation.

### Common Trap: Typed Nil Interfaces

```go
var l *ConsoleLogger = nil
var logger Logger = l

fmt.Println(logger == nil) // false
```

An interface value stores both a type and a value. Here the type is non-nil, even though the underlying pointer is nil.

---

## Struct Embedding + Middleware Decoration

Go has no inheritance. Instead, it relies on composition.

### Struct Embedding

```go
type Logger struct{}

func (l Logger) Log(msg string) {
    fmt.Println(msg)
}

type Service struct {
    Logger
}

func (s Service) Do() {
    s.Log("doing work")
}
```

Embedding promotes reuse without creating inheritance hierarchies.

### Middleware as Decorator Pattern

A canonical Go example is `net/http` middleware:

```go
func Logging(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("before")
        next.ServeHTTP(w, r)
        fmt.Println("after")
    })
}
```

This is behavioral composition through wrapping.

### When to Use Each

Use embedding for:

* shared internal capabilities
* helper behavior
* package-local composition

Use middleware for:

* logging
* tracing
* retries
* authentication
* request-scoped behavior

### Key Insight

> Embedding is structural composition. Middleware is behavioral composition.

---

## Functional Options vs Builder Pattern

These patterns solve the same problem: constructing configurable objects cleanly.

### Functional Options

This is the idiomatic Go approach.

```go
type Server struct {
    port    int
    timeout time.Duration
    logger  Logger
}

type Option func(*Server)

func WithPort(p int) Option {
    return func(s *Server) {
        s.port = p
    }
}

func WithTimeout(t time.Duration) Option {
    return func(s *Server) {
        s.timeout = t
    }
}

func NewServer(opts ...Option) *Server {
    s := &Server{port: 8080}

    for _, opt := range opts {
        opt(s)
    }

    return s
}
```

### Builder Pattern

```go
type ServerBuilder struct {
    port    int
    timeout time.Duration
}

func (b *ServerBuilder) Port(p int) *ServerBuilder {
    b.port = p
    return b
}

func (b *ServerBuilder) Timeout(t time.Duration) *ServerBuilder {
    b.timeout = t
    return b
}

func (b *ServerBuilder) Build() *Server {
    return &Server{
        port:    b.port,
        timeout: b.timeout,
    }
}
```

### Tradeoff

Functional options are usually preferred in Go because they:

* keep constructors compact
* preserve defaults cleanly
* evolve APIs without breaking call sites
* avoid extra mutable builder structs

Builders still make sense when construction is staged, hierarchical, or requires cross-field validation.

### Key Insight

> Functional options are configuration middleware for constructors.

---

## Context Propagation + Cancellation

The `context` package carries request-scoped control information through call chains.

```go
func FetchUser(ctx context.Context, db *sql.DB, id string) (*User, error) {
    row := db.QueryRowContext(ctx, "SELECT name FROM users WHERE id=?", id)

    var name string
    if err := row.Scan(&name); err != nil {
        return nil, err
    }

    return &User{Name: name}, nil
}
```

### What Context Carries

* cancellation
* deadlines
* tracing metadata
* request-scoped control signals

### Cancellation Pattern

```go
func worker(ctx context.Context, jobs <-chan int) {
    for {
        select {
        case job := <-jobs:
            fmt.Println("processing", job)
        case <-ctx.Done():
            fmt.Println("worker shutting down")
            return
        }
    }
}
```

### Rules of Thumb

* `context.Context` should be the first parameter
* always propagate it downward
* do not store dependencies in `ctx.Value`
* do not replace it with `context.Background()` inside request-scoped work

### Key Insight

> Context is not data. It is control information about the lifetime of work.

---

## Channel Pipelines + Backpressure

Go channels make it natural to model staged dataflow systems.

### Pipeline Example

```go
func generator(nums ...int) <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()

    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)

    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()

    return out
}
```

### Backpressure

Channels also act as flow-control mechanisms.

```go
jobs := make(chan Job, 100)
```

When consumers are slow and the buffer fills, producers block. That prevents runaway memory growth.

### Key Insight

> Channels are not just communication mechanisms. They are flow-control mechanisms.

---

## Error Wrapping + Sentinel Errors

Go error handling favors explicit control flow with rich context.

### Error Wrapping

```go
func loadUser(id string) error {
    err := queryDatabase(id)
    if err != nil {
        return fmt.Errorf("load user %s: %w", id, err)
    }
    return nil
}
```

Wrapping adds context while preserving the original cause.

Use `errors.Is` and `errors.As` to inspect wrapped chains.

### Sentinel Errors

```go
var ErrUserNotFound = errors.New("user not found")
```

Sentinel errors represent stable semantic conditions that callers can branch on.

```go
if errors.Is(err, ErrUserNotFound) {
    fmt.Println("handle missing user")
}
```

### Key Insight

> Error wrapping tells the story of failure. Sentinel errors define categories of failure.

---

## Custom Error Types + Error Classification

When plain errors are not enough, Go often uses typed errors.

### Custom Error Type

```go
type ValidationError struct {
    Field string
    Msg   string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed for %s: %s", e.Field, e.Msg)
}
```

This adds structured metadata to the error.

### Classification

```go
var netErr net.Error
if errors.As(err, &netErr) && netErr.Timeout() {
    fmt.Println("retry request")
}
```

This lets callers react to classes of failure without fragile string matching.

### Key Insight

> Good errors do not just explain failure. They guide recovery.

---

## Interface Adapters + Dependency Injection

Go favors explicit construction and interface boundaries.

### Interface Adapter

```go
type UserRepository interface {
    GetUser(ctx context.Context, id string) (*User, error)
}

type PostgresRepo struct {
    db *sql.DB
}

func (p *PostgresRepo) GetUser(ctx context.Context, id string) (*User, error) {
    row := p.db.QueryRowContext(ctx, "SELECT name FROM users WHERE id=$1", id)

    var name string
    if err := row.Scan(&name); err != nil {
        return nil, err
    }

    return &User{Name: name}, nil
}
```

### Dependency Injection

```go
type UserService struct {
    repo   UserRepository
    logger Logger
}

func NewUserService(repo UserRepository, logger Logger) *UserService {
    return &UserService{
        repo:   repo,
        logger: logger,
    }
}
```

Go does not usually need DI frameworks. Constructor injection is enough.

### Key Insight

> In Go, dependency injection means explicit construction plus interfaces at boundaries.

---

## Stream Interfaces + Adapter Composition

A huge portion of the Go standard library is built on tiny stream interfaces.

### `io.Reader`

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

This one interface can represent:

* files
* sockets
* HTTP request bodies
* compressed streams
* in-memory buffers

### Adapter Composition

```go
file, _ := os.Open("data.gz")
gzipReader, _ := gzip.NewReader(file)
bufReader := bufio.NewReader(gzipReader)
```

Pipeline:

```text
file → gzip → buffer → application
```

Each layer wraps the previous one while still implementing `io.Reader`.

### Key Insight

> Interfaces plus adapters turn Go programs into composable stream-processing graphs.

---

## Final Takeaways

Several themes show up again and again across Go:

* prefer small interfaces over large ones
* compose behavior instead of building hierarchies
* wire dependencies explicitly
* propagate context through the full call chain
* treat channels as both concurrency and flow-control tools
* make errors rich enough for both debugging and recovery

Go’s design is not about clever abstraction. It is about clear seams, explicit behavior, and systems that remain understandable under load.

<!-- gh-pages-taxonomy-links:start -->
Categories: [go](/categories/go/)
Tags: [go](/tags/go/), [interfaces](/tags/interfaces/), [composition](/tags/composition/), [error-handling](/tags/error-handling/), [concurrency](/tags/concurrency/)
<!-- gh-pages-taxonomy-links:end -->
