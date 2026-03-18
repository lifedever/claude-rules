# Java Guidelines

## Core Principles

- Minimum Java 17; prefer newer language features (record, sealed class, pattern matching, text block)
- No bare `null`; wrap return values with `Optional<T>`, and never pass null as an argument
- Prefer immutability: use `final` for fields, `List.of()`, `Map.of()`, `Collections.unmodifiable*` for collections
- No magic numbers or magic strings; extract them into constants

## Naming

- Classes and interfaces: `PascalCase` (`UserService`, `Configurable`)
- Methods and variables: `camelCase` (`getUserById`, `isValid`)
- Constants: `UPPER_SNAKE_CASE` (`MAX_RETRY_COUNT`)
- Package names: all lowercase (`com.example.userservice`)
- Booleans: `is`/`has`/`can`/`should` prefix
- No `I` prefix for interfaces; implementation classes use an `Impl` suffix or a descriptive name (`JdbcUserRepository`)

## Modern Java Features (17+)

```java
// 禁止：传统 POJO
public class UserDto {
    private String name;
    private int age;
    // getter, setter, equals, hashCode, toString...
}

// 正确：record（不可变数据载体）
public record UserDto(String name, int age) {}

// 禁止：instanceof + 强制转换
if (shape instanceof Circle) {
    Circle c = (Circle) shape;
    return c.radius();
}

// 正确：pattern matching
if (shape instanceof Circle c) {
    return c.radius();
}

// 正确：sealed class 表示有限类型
public sealed interface Shape permits Circle, Rectangle, Triangle {}
public record Circle(double radius) implements Shape {}
public record Rectangle(double width, double height) implements Shape {}
```

## Class Design

- A single class should not exceed 300 lines
- A single method should not exceed 30 lines
- No more than 4 method parameters; use a parameter object when exceeding this limit
- Composition over inheritance; inheritance depth should not exceed 3 levels
- Utility classes should be `final class` with a private constructor; do not use `abstract class`

## Optional

```java
// 禁止
User user = userRepository.findById(id);
if (user != null) { ... }

// 正确
userRepository.findById(id)
    .map(User::getName)
    .orElseThrow(() -> new UserNotFoundException(id));

// 禁止：Optional 做参数
void process(Optional<String> name) { ... }

// 正确：重载或默认值
void process(String name) { ... }
void process() { process("default"); }
```

## Collections and Streams

- Create immutable collections: `List.of()`, `Set.of()`, `Map.of()`
- Stream chains should not exceed 5 steps; extract intermediate variables or methods when they do
- Do not force Streams for simple loops; use `for-each` when it is clearer
- No side effects inside Streams (do not modify external state)

```java
// 禁止：Stream 里有副作用
List<String> results = new ArrayList<>();
users.stream().forEach(u -> results.add(u.getName()));

// 正确
List<String> results = users.stream()
    .map(User::getName)
    .toList();
```

## Exception Handling

- Catch specific exceptions; no `catch (Exception e)`
- Business exceptions should extend a custom base class and carry an error code
- No swallowing exceptions (empty catch blocks)
- Use `try-with-resources` to manage resources

```java
// 禁止
try {
    readFile(path);
} catch (Exception e) {
    // 吞掉了
}

// 正确
try (var reader = Files.newBufferedReader(path)) {
    return reader.lines().toList();
} catch (NoSuchFileException e) {
    throw new ConfigNotFoundException("配置文件不存在: " + path, e);
}
```

## Concurrency

- Prefer `ExecutorService` / `CompletableFuture`; do not use bare `new Thread()`
- On Java 21+, prefer virtual threads (`Thread.ofVirtual()`)
- Use thread-safe classes for shared mutable state: `ConcurrentHashMap`, `AtomicReference`, etc.
- Do not `synchronized` an entire method; minimize the synchronized scope
