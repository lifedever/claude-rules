# Rust Guidelines

## Core Principles

- Follow the [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- Code must pass `cargo clippy` with no warnings
- Prefer borrowing over cloning; clone only when ownership transfer is truly needed
- Use `Result<T, E>` for all fallible operations; no `unwrap()` in production code
- Prefer zero-cost abstractions: iterators, traits, and generics over runtime polymorphism

## Naming

- Functions and variables: `snake_case` (`parse_config`, `user_count`)
- Types, traits, and enums: `PascalCase` (`UserService`, `FromStr`, `HttpError`)
- Constants and statics: `SCREAMING_SNAKE_CASE` (`MAX_RETRIES`, `DEFAULT_PORT`)
- Lifetimes: short lowercase (`'a`, `'de`); descriptive only when multiple lifetimes need distinction
- Crate names: `kebab-case` in Cargo.toml, `snake_case` when imported
- Type conversions follow standard naming: `as_`, `to_`, `into_` prefixes per Rust conventions

## Ownership and Borrowing

- Prefer references (`&T`, `&mut T`) over owned values in function parameters
- Use `&str` instead of `String` in function parameters when the function does not need ownership
- Clone only when the borrow checker requires it and restructuring is not feasible
- Use `Cow<'_, str>` when a function may or may not allocate

```rust
// Bad: takes ownership unnecessarily, forces caller to clone
fn greet(name: String) -> String {
    format!("Hello, {name}!")
}

// Good: borrows, no allocation required from caller
fn greet(name: &str) -> String {
    format!("Hello, {name}!")
}
```

## Error Handling

- Libraries: use `thiserror` for defining error types with `#[derive(Error)]`
- Applications: use `anyhow` for ad-hoc error context with `.context("msg")`
- Implement `From<SourceError>` for automatic error conversion with `?`
- No `unwrap()` or `expect()` in library code; acceptable in tests and one-off scripts
- Use `?` operator for propagation; avoid manual `match` on `Result` just to re-wrap

```rust
// Bad: panics on error, no context
fn read_config() -> Config {
    let content = std::fs::read_to_string("config.toml").unwrap();
    toml::from_str(&content).unwrap()
}

// Good: propagates with context (application code using anyhow)
fn read_config() -> anyhow::Result<Config> {
    let content = std::fs::read_to_string("config.toml")
        .context("failed to read config.toml")?;
    let config: Config = toml::from_str(&content)
        .context("failed to parse config.toml")?;
    Ok(config)
}
```

```rust
// Good: library error type with thiserror
#[derive(Debug, Error)]
pub enum StorageError {
    #[error("record not found: {id}")]
    NotFound { id: String },
    #[error("connection failed")]
    Connection(#[from] std::io::Error),
    #[error("serialization failed")]
    Serialization(#[from] serde_json::Error),
}
```

## Pattern Matching

- Use exhaustive matches; no wildcard `_` catch-all unless the remaining cases are truly irrelevant
- Use `if let` for single-variant checks instead of a full `match`
- Use `matches!` macro for boolean pattern checks
- Destructure in match arms to avoid redundant field access

```rust
// Bad: full match for a single variant
match result {
    Ok(value) => process(value),
    Err(_) => {}
}

// Good: if let for single variant
if let Ok(value) = result {
    process(value);
}

// Bad: verbose boolean check
match status {
    Status::Active | Status::Pending => true,
    _ => false,
}

// Good: matches! macro
matches!(status, Status::Active | Status::Pending)
```

## Iterators and Combinators

- Prefer iterator chains over manual `for` loops with mutable accumulators
- Use `collect()` with type annotation to drive inference
- Use `filter_map` instead of `filter` + `map` when both apply
- Prefer `iter()` over indexing with `[i]`

```rust
// Bad: manual accumulation
let mut names = Vec::new();
for user in &users {
    if user.is_active {
        names.push(user.name.clone());
    }
}

// Good: iterator chain
let names: Vec<String> = users
    .iter()
    .filter(|u| u.is_active)
    .map(|u| u.name.clone())
    .collect();
```

## Traits

- Derive common traits: `Debug`, `Clone`, `PartialEq` on all public types unless there is a reason not to
- Keep trait definitions small: 1-3 required methods, provide default implementations where sensible
- Use trait bounds with `impl Trait` in argument position for simple cases; use `where` clauses when bounds are long
- Prefer static dispatch (`impl Trait`, generics) over dynamic dispatch (`dyn Trait`) unless heterogeneous collections are needed

```rust
// Bad: no derived traits, consumers cannot debug-print or compare
pub struct Config {
    pub host: String,
    pub port: u16,
}

// Good: common traits derived
#[derive(Debug, Clone, PartialEq)]
pub struct Config {
    pub host: String,
    pub port: u16,
}
```

## Lifetimes

- Let the compiler infer lifetimes; add explicit annotations only when the compiler requires them
- When explicit lifetimes are needed, keep them minimal
- Use `'_` for anonymous lifetimes in impl blocks to reduce noise

```rust
// Bad: unnecessary explicit lifetime
fn first_word<'a>(s: &'a str) -> &'a str {
    s.split_whitespace().next().unwrap_or("")
}

// Good: compiler infers the lifetime
fn first_word(s: &str) -> &str {
    s.split_whitespace().next().unwrap_or("")
}
```

## Struct and Enum Design

- Use the builder pattern for structs with more than 3 optional fields
- Use `#[non_exhaustive]` on public enums and structs that may grow
- Prefer enums over boolean flags for states with distinct behavior
- Use `Option<T>` for optional values; never use sentinel values like `-1` or `""`

```rust
// Bad: boolean flags, unclear meaning
fn connect(host: &str, port: u16, secure: bool, verbose: bool) { }

// Good: builder pattern for complex configuration
#[derive(Debug, Clone)]
pub struct ConnectionConfig {
    host: String,
    port: u16,
    secure: bool,
    verbose: bool,
}

impl ConnectionConfig {
    pub fn new(host: impl Into<String>, port: u16) -> Self {
        Self { host: host.into(), port, secure: false, verbose: false }
    }
    pub fn secure(mut self, secure: bool) -> Self { self.secure = secure; self }
    pub fn verbose(mut self, verbose: bool) -> Self { self.verbose = verbose; self }
}
```

## Unsafe

- No `unsafe` without an accompanying `// SAFETY:` comment explaining the invariant
- Encapsulate `unsafe` in a safe public API; never expose raw unsafe operations
- Prefer safe alternatives: `get()` over unchecked indexing, `checked_add` over wrapping arithmetic

```rust
// Bad: unsafe with no justification
unsafe { *ptr.add(offset) }

// Good: justified, encapsulated
// SAFETY: `offset` is bounds-checked against `self.len` in the caller,
// and `ptr` is guaranteed valid for the lifetime of `Buffer`.
unsafe { *ptr.add(offset) }
```

## Testing

- Unit tests in a `#[cfg(test)] mod tests` block at the bottom of each file
- Use `#[test]` with descriptive names: `fn parses_valid_input()`, not `fn test1()`
- Use `assert_eq!` and `assert_ne!` over `assert!(a == b)` for better failure messages
- Use `proptest` or `quickcheck` for property-based testing on parsing and serialization logic
- `unwrap()` and `expect()` are acceptable in test code

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn parses_valid_duration() {
        assert_eq!(parse_duration("5s"), Ok(Duration::from_secs(5)));
        assert_eq!(parse_duration("100ms"), Ok(Duration::from_millis(100)));
    }

    #[test]
    fn rejects_invalid_duration() {
        assert!(parse_duration("abc").is_err());
        assert!(parse_duration("").is_err());
    }
}
```

## Forbidden

- `unsafe` without a `// SAFETY:` comment
- `unwrap()` or `expect()` in library/production code
- Wildcard imports (`use some_crate::*`)
- `clone()` as a shortcut to avoid understanding ownership
- `String` parameters where `&str` suffices
- Silently ignoring `Result` with `let _ =` (use `if let Err(e)` and log, or propagate)
