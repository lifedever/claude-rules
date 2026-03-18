# Java 规范

## 基本原则

- 最低 Java 17，优先使用新版本特性（record、sealed class、pattern matching、text block）
- 禁止裸用 `null`，返回值用 `Optional<T>` 包装，参数禁止传 null
- 优先不可变：字段用 `final`，集合用 `List.of()`、`Map.of()`、`Collections.unmodifiable*`
- 禁止魔法数字和魔法字符串，提取为常量

## 命名

- 类和接口：`PascalCase`（`UserService`、`Configurable`）
- 方法和变量：`camelCase`（`getUserById`、`isValid`）
- 常量：`UPPER_SNAKE_CASE`（`MAX_RETRY_COUNT`）
- 包名：全小写（`com.example.userservice`）
- 布尔值：`is`/`has`/`can`/`should` 前缀
- 接口不加 `I` 前缀，实现类用 `Impl` 后缀或具体描述（`JdbcUserRepository`）

## 现代 Java 特性（17+）

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

## 类设计

- 单个类不超过 300 行
- 单个方法不超过 30 行
- 方法参数不超过 4 个，超过用参数对象
- 组合优于继承，继承层级不超过 3 层
- 工具类用 `final class` + 私有构造器，不要用 `abstract class`

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

## 集合和 Stream

- 创建不可变集合：`List.of()`、`Set.of()`、`Map.of()`
- Stream 链不超过 5 步，超过提取中间变量或方法
- 简单循环不要强行用 Stream，`for-each` 更清晰时就用 `for-each`
- 禁止在 Stream 里修改外部状态（副作用）

```java
// 禁止：Stream 里有副作用
List<String> results = new ArrayList<>();
users.stream().forEach(u -> results.add(u.getName()));

// 正确
List<String> results = users.stream()
    .map(User::getName)
    .toList();
```

## 异常处理

- 捕获具体异常，禁止 `catch (Exception e)`
- 业务异常继承自定义基类，携带错误码
- 禁止吞异常（空 catch 块）
- 用 `try-with-resources` 管理资源

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

## 并发

- 优先用 `ExecutorService` / `CompletableFuture`，不要裸 `new Thread()`
- Java 21+ 优先用虚拟线程（`Thread.ofVirtual()`）
- 共享可变状态用 `ConcurrentHashMap`、`AtomicReference` 等线程安全类
- 禁止 `synchronized` 整个方法，缩小同步范围
