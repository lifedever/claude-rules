# Swift Guidelines

## Core Principles

- Prefer `let` over `var`; default to immutability
- Leverage Swift's type inference; explicitly annotate types for public APIs
- Use `guard` for early exits to avoid nesting
- Prefer value types (`struct`, `enum`) over reference types (`class`), unless reference semantics or inheritance is needed

## Naming

- Follow the Swift API Design Guidelines
- Types and protocols: `PascalCase` (`UserProfile`, `Configurable`)
- Functions and variables: `camelCase` (`fetchUser`, `isValid`)
- Function names should read like English phrases: `remove(at: index)` rather than `remove(index: index)`
- Boolean properties use `is`/`has`/`should` prefix
- Factory methods use `make` prefix: `makeIterator()`

## Error Handling

- Use `throws` for recoverable errors; use `fatalError` only for programmer errors
- Custom error types should conform to `LocalizedError` and provide a meaningful `errorDescription`
- No `try!` (unless the call is 100% guaranteed to succeed, e.g., loading a bundle resource)
- When using `try?`, always handle the nil case

```swift
// 禁止
let data = try! JSONDecoder().decode(User.self, from: jsonData)

// 正确
do {
    let data = try JSONDecoder().decode(User.self, from: jsonData)
    handleUser(data)
} catch {
    logger.error("Failed to decode user: \(error.localizedDescription)")
    showError(.decodeFailed)
}
```

## Optionals

- Prefer `guard let` for unwrapping with early exit
- Use `??` to provide default values
- No `!` force unwrapping (except for `IBOutlet` or extreme cases with an explanatory comment)
- Chain calls with optional chaining `?.`

```swift
// 禁止
func process(user: User?) {
    if let user = user {
        if let email = user.email {
            sendEmail(to: email)
        }
    }
}

// 正确
func process(user: User?) {
    guard let user, let email = user.email else { return }
    sendEmail(to: email)
}
```

## Concurrency (Swift Concurrency)

- Use `async/await` instead of completion handlers
- Use `actor` to protect shared mutable state
- Use `TaskGroup` for concurrent operations
- Code marked `@MainActor` should be limited to UI updates
- Use `Task { }` to bridge from synchronous to asynchronous contexts; do not use `DispatchQueue`

```swift
// 禁止：老式回调
func fetchUser(id: String, completion: @escaping (Result<User, Error>) -> Void) { ... }

// 正确
func fetchUser(id: String) async throws -> User { ... }
```

## Protocols

- Name protocols with adjectives (`Configurable`, `Identifiable`) or nouns (`DataSource`, `Delegate`)
- Use protocol extensions to provide default implementations
- Prefer protocol composition (`Codable & Identifiable`) over inheritance chains
- Use `some` and `any` to distinguish opaque types from existential types

## Access Control

- Least visibility: default to `private`, widen only when needed
- Use `internal` (the default) within a module; use `public` across modules
- Use `open` only when subclassing is explicitly intended
- Use `fileprivate` instead of `private` only when multiple types in the same file need shared access
