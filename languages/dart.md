# Dart Guidelines

## Core Principles

- Target Dart 3+ with sound null safety
- Use strong typing; avoid `dynamic` unless justified with a comment
- Prefer immutability: use `final` for local variables, `const` for compile-time constants
- No magic numbers; define named constants
- A single file should not exceed 300 lines; a single function should not exceed 25 lines

## Naming

- Classes, enums, type aliases, extensions: `UpperCamelCase` (`UserRepository`, `PaymentStatus`)
- Functions, variables, parameters, named constants: `lowerCamelCase` (`getUserById`, `isValid`, `maxRetryCount`)
- Libraries, packages, directories, source files: `lowercase_with_underscores` (`user_service.dart`, `payment_utils`)
- Private members: `_` prefix (`_cache`, `_processInternal`)
- Booleans: `is`/`has`/`can`/`should` prefix (`isActive`, `hasPermission`)
- No Hungarian notation or type-name prefixes

## Modern Dart 3+ Features

- Use records for lightweight multi-value returns
- Use pattern matching and destructuring in `switch` and `if-case`
- Use sealed classes for exhaustive type hierarchies
- Use class modifiers (`final`, `base`, `interface`, `mixin`) to control subtyping

```dart
// Bad: using a Map to return multiple values
Map<String, dynamic> getNameAndAge() => {'name': 'Alice', 'age': 30};

// Good: use a record
(String name, int age) getNameAndAge() => ('Alice', 30);

// Good: destructuring
final (name, age) = getNameAndAge();
```

## Sealed Classes and Pattern Matching

- Use `sealed` for types with a known, finite set of subtypes
- Use `switch` expressions for exhaustive handling — the compiler enforces completeness

```dart
// Good: sealed class with exhaustive switch
sealed class AuthState {}
class Authenticated extends AuthState {
  final User user;
  Authenticated(this.user);
}
class Unauthenticated extends AuthState {}
class Loading extends AuthState {}

String describeState(AuthState state) => switch (state) {
  Authenticated(user: final u) => 'Logged in as ${u.name}',
  Unauthenticated() => 'Please log in',
  Loading() => 'Loading...',
};
```

## Null Safety

- Use `?` only when a value is genuinely optional in the domain
- Prefer `late` over nullable when initialization is guaranteed before access
- Use `??` for defaults, `?.` for safe navigation
- Never use `!` without a comment explaining why null is impossible at that point

```dart
// Bad: nullable when value is always set before use
String? _name;
void init() { _name = 'Alice'; }
void greet() { print(_name!); }

// Good: late initialization
late String _name;
void init() { _name = 'Alice'; }
void greet() { print(_name); }
```

## Const and Immutability

- Use `const` constructors wherever possible — they enable compile-time constants and widget caching
- Prefer `final` fields in classes; use mutable fields only when mutation is essential
- Use `unmodifiable` wrappers for collections exposed via public APIs

```dart
// Bad
var items = [1, 2, 3]; // mutable and not const

// Good
const items = [1, 2, 3]; // compile-time constant
final dynamicItems = List<int>.unmodifiable(fetchItems());
```

## Function Design

- Return early to avoid deep nesting
- Use expression bodies (`=>`) for single-expression functions
- Use named parameters with `required` for functions with more than 2 parameters
- Use `typedef` for complex function signatures

```dart
// Bad: positional parameters are confusing
void createUser(String name, String email, int age, bool isAdmin) { ... }

// Good: named parameters
void createUser({
  required String name,
  required String email,
  required int age,
  bool isAdmin = false,
}) { ... }
```

## Async

- Use `async`/`await` over raw `Future.then()` chains
- Return `Future<void>` not `void` for async functions
- Use `Stream` for multiple async values; prefer `async*` generators
- Handle errors with `try`/`catch` on specific exception types

```dart
// Bad
Future<User> fetchUser(int id) {
  return http.get(uri).then((res) => parseUser(res.body)).catchError((e) => throw e);
}

// Good
Future<User> fetchUser(int id) async {
  try {
    final res = await http.get(uri);
    return parseUser(res.body);
  } on HttpException catch (e) {
    logger.warning('Failed to fetch user $id: $e');
    rethrow;
  }
}
```

## Error Handling

- Catch specific exception types; never catch bare `Exception` without rethrowing or logging
- Define custom exception classes for domain errors
- No empty catch blocks — at minimum log the error

## Prohibited Patterns

- No `print()` in production code; use a logger (`package:logging` or equivalent)
- No `dynamic` without a justifying comment
- No `as` type casts without a preceding type check
- No `runtimeType` checks in production; use `is` or sealed classes
- No mutable global state; use dependency injection or a service locator

```dart
// Bad
dynamic data = fetchData();
print('Got data: $data');

// Good
final UserDto data = await fetchData();
logger.info('Got data: ${data.id}');
```

## Visibility

- Use `_` prefix to make members library-private; Dart has no `protected`/`public` keywords
- Expose the minimum necessary API; keep implementation details private
- Use `@visibleForTesting` annotation for members exposed only for tests
