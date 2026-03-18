# Swift 规范

## 基本原则

- 优先 `let` 而非 `var`，默认不可变
- 利用 Swift 的类型推断，公共 API 显式标注类型
- 用 `guard` 提前退出，避免嵌套
- 用值类型（`struct`、`enum`）优于引用类型（`class`），除非需要引用语义或继承

## 命名

- 遵循 Swift API Design Guidelines
- 类型和协议：`PascalCase`（`UserProfile`、`Configurable`）
- 函数和变量：`camelCase`（`fetchUser`、`isValid`）
- 函数名读起来像英语短语：`remove(at: index)` 而非 `remove(index: index)`
- 布尔属性用 `is`/`has`/`should` 前缀
- 工厂方法用 `make` 前缀：`makeIterator()`

## 错误处理

- 可恢复错误用 `throws`，不可恢复用 `fatalError`（仅限编程错误）
- 自定义错误类型实现 `LocalizedError`，提供有意义的 `errorDescription`
- 禁止 `try!`（除非 100% 确定不会失败，如加载 bundle 资源）
- 用 `try?` 时必须处理 nil 情况

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

## 可选值

- 优先 `guard let` 解包并提前退出
- 用 `??` 提供默认值
- 禁止 `!` 强制解包（除非在 `IBOutlet` 或有注释说明的极端场景）
- 可选链 `?.` 串联调用

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

## 并发（Swift Concurrency）

- 用 `async/await` 替代 completion handler
- 用 `actor` 保护共享可变状态
- 用 `TaskGroup` 处理并发操作
- 标记 `@MainActor` 的代码仅限 UI 更新
- 用 `Task { }` 从同步上下文桥接到异步，不要用 `DispatchQueue`

```swift
// 禁止：老式回调
func fetchUser(id: String, completion: @escaping (Result<User, Error>) -> Void) { ... }

// 正确
func fetchUser(id: String) async throws -> User { ... }
```

## 协议

- 协议命名用形容词（`Configurable`、`Identifiable`）或名词（`DataSource`、`Delegate`）
- 用协议扩展提供默认实现
- 优先协议组合（`Codable & Identifiable`）而非继承链
- 用 `some` 和 `any` 区分不透明类型和存在类型

## 访问控制

- 最小可见性：默认 `private`，需要时才放宽
- 模块内部用 `internal`（默认），跨模块用 `public`
- `open` 仅在明确允许子类化时使用
- 用 `fileprivate` 替代 `private` 仅当同文件多个类型需要共享
