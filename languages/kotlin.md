# Kotlin 规范

## 基本原则

- 优先 `val` 而非 `var`，默认不可变
- 优先类型推断，公共 API 的返回类型显式声明
- 禁止 `Any` 做参数或返回类型，创建具体类型
- 禁止魔法数字，定义命名常量

## 命名

- 类和接口：`PascalCase`（`UserRepository`、`PaymentService`）
- 函数和变量：`camelCase`（`getUserById`、`isValid`）
- 常量和枚举值：`UPPER_SNAKE_CASE`（`MAX_RETRY_COUNT`）
- 包名：全小写点分隔（`com.example.userservice`）
- 布尔值：`is`/`has`/`can`/`should` 前缀
- 禁止无意义缩写，标准缩写除外（API、URL、HTTP、JSON、id、ctx）

## 函数设计

- 单个函数不超过 20 行
- 函数名以动词开头
- 单行返回用表达式函数：`fun square(x: Int) = x * x`
- 用命名参数提高可读性：`createUser(name = "John", age = 25)`
- 用默认参数替代函数重载
- 通过提前返回避免深层嵌套

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

## 类设计

- 数据承载用 `data class`
- 有限状态用 `sealed class` / `sealed interface`
- 单例和工具类用 `object`
- 类型安全的原始包装用 `value class`
- 单个类不超过 200 行、不超过 10 个公共方法
- 组合优于继承

```kotlin
// 用 sealed interface 表示状态，而非 enum + 额外字段
sealed interface Result<out T> {
    data class Success<T>(val data: T) : Result<T>
    data class Error(val message: String, val cause: Throwable? = null) : Result<Nothing>
    data object Loading : Result<Nothing>
}
```

## 空安全

- 用 `?.` 安全调用，用 `?:` 提供默认值
- 禁止 `!!`（除非有注释说明为什么保证非空）
- 不要用 `if (x != null)` 嵌套，用 `?.let` 或 `?:` 链

```kotlin
// 禁止
val name = user!!.name

// 正确
val name = user?.name ?: "Unknown"
```

## 协程

- 禁止 `GlobalScope`，始终使用结构化并发（`viewModelScope`、`lifecycleScope`、`coroutineScope`）
- `suspend` 函数处理单次异步操作，`Flow` 处理数据流
- UI 状态用 `StateFlow`，事件用 `SharedFlow`
- 并发操作用 `async`/`await`，不要顺序等待

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

## 集合

- 默认用不可变集合：`listOf`、`setOf`、`mapOf`
- 大数据集或链式操作用 `asSequence()`
- 简单 lambda 用 `it`，复杂场景用命名参数
- 善用 `takeIf`、`takeUnless`、作用域函数

## 可见性

- 最小可见性原则：模块内部用 `internal`，类内部用 `private`
- 不要把所有东西都 `public`
