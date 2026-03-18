# Go Guidelines

## Core Principles

- Follow [Effective Go](https://go.dev/doc/effective_go) and [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments)
- No `init()` functions; use explicit initialization
- No global mutable state; pass dependencies explicitly
- No dot imports; no blank identifier `_` to ignore errors
- Prefer composition over embedding unless embedding adds method set value

## Naming

- Exported names: `MixedCaps` (`UserService`, `ParseConfig`)
- Unexported names: `mixedCaps` (`userRepo`, `parseLine`)
- Package names: lowercase single word, no underscores or camelCase (`http`, `user`, `config`)
- Short variable names in small scopes: `i`, `r`, `buf`; descriptive names in larger scopes
- Interfaces: single-method interfaces use `-er` suffix (`Reader`, `Stringer`, `Closer`)
- Acronyms are all caps: `HTTPClient`, `userID`, `xmlParser`
- No `Get` prefix for getters: `user.Name()` not `user.GetName()`

## Function Design

- A single function should not exceed 30 lines
- No naked returns; always name what you return explicitly at the `return` statement
- Receiver names: short (1-2 letters), consistent across methods, never `this` or `self`
- Return early to avoid deep nesting
- Accept interfaces, return structs

```go
// Bad: naked return, unclear intent
func findUser(id string) (u *User, err error) {
    u, err = db.Lookup(id)
    if err != nil {
        return
    }
    return
}

// Good: explicit return values
func findUser(id string) (*User, error) {
    u, err := db.Lookup(id)
    if err != nil {
        return nil, fmt.Errorf("find user %s: %w", id, err)
    }
    return u, nil
}
```

## Error Handling

- Always check errors; never assign to `_`
- Wrap errors with context using `fmt.Errorf("doing X: %w", err)`
- Use `errors.Is` and `errors.As` for error inspection, not type assertions
- Define sentinel errors as package-level `var ErrNotFound = errors.New("not found")`
- No `panic` in library code; reserve panic for truly unrecoverable programmer errors in `main`

```go
// Bad: ignored error, raw comparison
result, _ := doSomething()
if err == sql.ErrNoRows {
    return nil
}

// Good: checked error, errors.Is
result, err := doSomething()
if err != nil {
    return fmt.Errorf("do something: %w", err)
}
if errors.Is(err, sql.ErrNoRows) {
    return nil, nil
}
```

## Interface Design

- Keep interfaces small: 1-3 methods maximum
- Define interfaces at the consumer side, not the implementation side
- Accept interfaces, return concrete structs
- Do not export interfaces solely for mocking; use consumer-defined interfaces

```go
// Bad: large interface defined by the provider
type UserStore interface {
    Create(user *User) error
    Update(user *User) error
    Delete(id string) error
    FindByID(id string) (*User, error)
    FindByEmail(email string) (*User, error)
    List(filter Filter) ([]*User, error)
}

// Good: small interface defined by the consumer
type UserFinder interface {
    FindByID(id string) (*User, error)
}

func NewOrderService(users UserFinder) *OrderService {
    return &OrderService{users: users}
}
```

## Concurrency

- Always pass `context.Context` as the first parameter to long-running or cancellable functions
- Start goroutines with clear ownership and shutdown path
- Prefer channels for communication; use `sync.Mutex` only for protecting shared state
- Use `errgroup.Group` for coordinating concurrent work with error propagation
- No goroutine leaks: every goroutine must have a termination condition

```go
// Bad: no context, no shutdown, goroutine leak
func startWorker() {
    go func() {
        for {
            processItem()
        }
    }()
}

// Good: context-controlled goroutine with clean shutdown
func startWorker(ctx context.Context, items <-chan Item) error {
    g, ctx := errgroup.WithContext(ctx)
    g.Go(func() error {
        for {
            select {
            case <-ctx.Done():
                return ctx.Err()
            case item, ok := <-items:
                if !ok {
                    return nil
                }
                if err := process(ctx, item); err != nil {
                    return fmt.Errorf("process item: %w", err)
                }
            }
        }
    })
    return g.Wait()
}
```

## Modern Go (1.18+)

- Use generics for type-safe utility functions and data structures; do not overuse for simple cases
- Use `slog` for structured logging instead of `log` or third-party loggers
- Use `any` instead of `interface{}`

```go
// Bad: untyped, requires casting
func Contains(slice []interface{}, target interface{}) bool {
    for _, v := range slice {
        if v == target {
            return true
        }
    }
    return false
}

// Good: generic, type-safe
func Contains[T comparable](slice []T, target T) bool {
    for _, v := range slice {
        if v == target {
            return true
        }
    }
    return false
}
```

## Structs and Types

- Use struct literals with field names; never rely on positional initialization
- Use `time.Duration` for durations, not `int` or `float64`
- Use `type X struct{}` for signal channels, not `chan bool`
- Prefer `[]byte` over `string` for mutable data

```go
// Bad: positional initialization, unclear fields
u := User{"Alice", 30, true}

// Good: named fields, self-documenting
u := User{
    Name:     "Alice",
    Age:      30,
    IsActive: true,
}
```

## Testing

- Table-driven tests with `t.Run` for subtests
- Use `t.Helper()` in test helper functions
- Use `t.Parallel()` for independent tests
- Test file in the same package for white-box tests; `_test` package for black-box tests
- No assertions libraries; use standard `if` checks with clear failure messages

```go
func TestParseSize(t *testing.T) {
    t.Parallel()
    tests := []struct {
        name    string
        input   string
        want    int64
        wantErr bool
    }{
        {name: "bytes", input: "100B", want: 100},
        {name: "kilobytes", input: "2KB", want: 2048},
        {name: "invalid", input: "abc", wantErr: true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel()
            got, err := ParseSize(tt.input)
            if (err != nil) != tt.wantErr {
                t.Fatalf("ParseSize(%q) error = %v, wantErr %v", tt.input, err, tt.wantErr)
            }
            if got != tt.want {
                t.Errorf("ParseSize(%q) = %d, want %d", tt.input, got, tt.want)
            }
        })
    }
}
```

## Forbidden

- `panic` in library code
- Global mutable state
- Dot imports (`import . "pkg"`)
- Ignoring errors with `_`
- `init()` functions
- Naked returns
- `interface{}` (use `any`)
