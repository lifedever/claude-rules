# SwiftUI 规范（Swift 5.9+ / macOS）

## 架构

- 用 MVVM：View → ViewModel → Model/Service
- View 只负责声明 UI，业务逻辑全部放在 ViewModel
- ViewModel 用 `@Observable`（Swift 5.9+），禁止旧的 `ObservableObject` + `@Published`
- 数据持久化用 SwiftData，禁止直接用 UserDefaults 存复杂数据

```swift
// 禁止：旧写法
class UserViewModel: ObservableObject {
    @Published var name = ""
    @Published var isLoading = false
}

// 正确：Swift 5.9+ 新写法
@Observable
class UserViewModel {
    var name = ""
    var isLoading = false

    func fetchUser() async { ... }
}
```

## View 规则

- 用 `struct` 创建 View，保持小巧（不超过 80 行 body）
- body 嵌套不超过 3 层，超过提取为子 View 或 ViewModifier
- 抽取可复用样式为自定义 `ViewModifier`，不要重复写 `.font().foregroundColor().padding()`
- 条件渲染用 `if let` / `switch`，不要用三元表达式拼 View

```swift
// 禁止：body 里堆逻辑
var body: some View {
    VStack {
        Text(name)
            .font(.title)
            .foregroundStyle(.primary)
            .padding()
        Text(email)
            .font(.title)
            .foregroundStyle(.primary)  // 重复样式
            .padding()
    }
}

// 正确：提取 ViewModifier
struct PrimaryTitle: ViewModifier {
    func body(content: Content) -> some View {
        content.font(.title).foregroundStyle(.primary).padding()
    }
}

var body: some View {
    VStack {
        Text(name).modifier(PrimaryTitle())
        Text(email).modifier(PrimaryTitle())
    }
}
```

## 状态管理

- View 内部临时状态用 `@State`（仅限简单值：Bool、String、Int）
- 共享数据通过 `@Environment` 注入，不要在 View 之间传递 ViewModel
- 子 View 需要修改父 View 状态时用 `@Binding`
- 全局共享状态通过 `.environment()` 注入到 View 树

```swift
// 正确的状态分层
@Observable class AppState {
    var currentUser: User?
    var theme: Theme = .system
}

// 根 View 注入
ContentView()
    .environment(appState)

// 子 View 使用
struct ProfileView: View {
    @Environment(AppState.self) private var appState
}
```

## SwiftData

- Model 用 `@Model` 宏标记
- 查询用 `@Query`，带排序和过滤
- 写操作通过 `modelContext` 在 ViewModel 里执行
- 复杂查询封装为 ViewModel 方法，不在 View 里写

## 导航

- macOS 用 `NavigationSplitView`（侧边栏 + 详情）
- 导航状态用枚举管理，不要用多个 Bool flag

```swift
// 禁止
@State private var showSettings = false
@State private var showProfile = false
@State private var showHelp = false

// 正确
enum Destination: Hashable {
    case settings
    case profile
    case help
}
@State private var selectedDestination: Destination?
```

## 异步

- 用 `.task { }` 修饰符在 View 出现时发起异步操作
- 用 `.task(id:)` 在参数变化时重新执行
- 禁止在 `onAppear` 里用 `Task { }`（`.task` 会自动取消）
- 加载状态和错误处理用明确的枚举

```swift
enum LoadState<T> {
    case idle
    case loading
    case loaded(T)
    case error(String)
}
```

## macOS 特有

- 菜单栏应用用 `MenuBarExtra`
- 多窗口用 `Window` 和 `WindowGroup`
- 键盘快捷键用 `.keyboardShortcut()`
- 支持暗色/亮色模式，用语义颜色（`.primary`、`.secondary`），不要硬编码颜色值
