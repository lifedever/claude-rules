# Kotlin Guidelines

## Core Principles

- Prefer `val` over `var`; default to immutability
- Prefer type inference; explicitly declare return types for public APIs
- No `Any` as a parameter or return type; create specific types
- No magic numbers; define named constants

## Naming

- Classes and interfaces: `PascalCase` (`UserRepository`, `PaymentService`)
- Functions and variables: `camelCase` (`getUserById`, `isValid`)
- Constants and enum values: `UPPER_SNAKE_CASE` (`MAX_RETRY_COUNT`)
- Package names: all lowercase, dot-separated (`com.example.userservice`)
- Booleans: `is`/`has`/`can`/`should` prefix
- No meaningless abbreviations; standard abbreviations are acceptable (API, URL, HTTP, JSON, id, ctx)

## Function Design

- A single function should not exceed 20 lines
- Function names start with a verb
- Use expression functions for single-line returns: `fun square(x: Int) = x * x`
- Use named arguments for readability: `createUser(name = "John", age = 25)`
- Use default parameters instead of function overloads
- Return early to avoid deep nesting

```kotlin
// 禁止
fun process(user: User?): String {
    if (user != null) {
        if (user.isActive) {
            if (user.hasPermission("admin")) {
                return "admin"
            }
        }
    }
    return "none"
}

// 正确
fun process(user: User?): String {
    user ?: return "none"
    if (!user.isActive) return "none"
    if (!user.hasPermission("admin")) return "none"
    return "admin"
}
```

## Class Design

- Use `data class` for data holders
- Use `sealed class` / `sealed interface` for finite states
- Use `object` for singletons and utility classes
- Use `value class` for type-safe primitive wrappers
- A single class should not exceed 200 lines or 10 public methods
- Composition over inheritance

```kotlin
// 用 sealed interface 表示状态，而非 enum + 额外字段
sealed interface Result<out T> {
    data class Success<T>(val data: T) : Result<T>
    data class Error(val message: String, val cause: Throwable? = null) : Result<Nothing>
    data object Loading : Result<Nothing>
}
```

## Null Safety

- Use `?.` for safe calls and `?:` for default values
- No `!!` (unless accompanied by a comment explaining why non-null is guaranteed)
- Do not nest `if (x != null)` checks; use `?.let` or `?:` chains instead

```kotlin
// 禁止
val name = user!!.name

// 正确
val name = user?.name ?: "Unknown"
```

## Coroutines

- No `GlobalScope`; always use structured concurrency (`viewModelScope`, `lifecycleScope`, `coroutineScope`)
- `suspend` functions for one-shot async operations; `Flow` for data streams
- Use `StateFlow` for UI state; use `SharedFlow` for events
- Use `async`/`await` for concurrent operations; do not await sequentially

```kotlin
// 禁止：顺序等待两个无关请求
val user = fetchUser(id)
val orders = fetchOrders(id)

// 正确：并发请求
coroutineScope {
    val user = async { fetchUser(id) }
    val orders = async { fetchOrders(id) }
    process(user.await(), orders.await())
}
```

## Collections

- Default to immutable collections: `listOf`, `setOf`, `mapOf`
- Use `asSequence()` for large data sets or chained operations
- Use `it` for simple lambdas; use named parameters for complex scenarios
- Leverage `takeIf`, `takeUnless`, and scope functions

## Visibility

- Principle of least visibility: use `internal` within a module, `private` within a class
- Do not make everything `public`
