# SwiftUI Guidelines (Swift 5.9+ / macOS)

## Architecture

- Use MVVM: View -> ViewModel -> Model/Service
- Views are responsible only for declaring UI; all business logic belongs in the ViewModel
- Use `@Observable` (Swift 5.9+) for ViewModels; the legacy `ObservableObject` + `@Published` pattern is forbidden
- Use SwiftData for data persistence; using UserDefaults directly for complex data is forbidden

```swift
// Forbidden: legacy approach
class UserViewModel: ObservableObject {
    @Published var name = ""
    @Published var isLoading = false
}

// Correct: Swift 5.9+ approach
@Observable
class UserViewModel {
    var name = ""
    var isLoading = false

    func fetchUser() async { ... }
}
```

## View Rules

- Create Views with `struct`, keep them small (body should not exceed 80 lines)
- Body nesting must not exceed 3 levels; extract into sub-Views or ViewModifiers if it does
- Extract reusable styles into custom `ViewModifier`s; do not repeat `.font().foregroundColor().padding()` chains
- Use `if let` / `switch` for conditional rendering; do not use ternary expressions to compose Views

```swift
// Forbidden: piling logic in body
var body: some View {
    VStack {
        Text(name)
            .font(.title)
            .foregroundStyle(.primary)
            .padding()
        Text(email)
            .font(.title)
            .foregroundStyle(.primary)  // Duplicated style
            .padding()
    }
}

// Correct: extract ViewModifier
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

## State Management

- Use `@State` for temporary state within a View (only for simple values: Bool, String, Int)
- Inject shared data via `@Environment`; do not pass ViewModels between Views
- Use `@Binding` when a child View needs to modify parent View state
- Inject globally shared state into the View tree via `.environment()`

```swift
// Correct state layering
@Observable class AppState {
    var currentUser: User?
    var theme: Theme = .system
}

// Inject at root View
ContentView()
    .environment(appState)

// Use in child View
struct ProfileView: View {
    @Environment(AppState.self) private var appState
}
```

## SwiftData

- Mark models with the `@Model` macro
- Use `@Query` for queries, with sorting and filtering
- Execute write operations through `modelContext` in the ViewModel
- Encapsulate complex queries as ViewModel methods; do not write them in Views

## Navigation

- Use `NavigationSplitView` on macOS (sidebar + detail)
- Manage navigation state with enums; do not use multiple Bool flags

```swift
// Forbidden
@State private var showSettings = false
@State private var showProfile = false
@State private var showHelp = false

// Correct
enum Destination: Hashable {
    case settings
    case profile
    case help
}
@State private var selectedDestination: Destination?
```

## Async

- Use the `.task { }` modifier to initiate async operations when a View appears
- Use `.task(id:)` to re-execute when a parameter changes
- Do not use `Task { }` inside `onAppear` (`.task` handles cancellation automatically)
- Use explicit enums for loading state and error handling

```swift
enum LoadState<T> {
    case idle
    case loading
    case loaded(T)
    case error(String)
}
```

## macOS-Specific

- Use `MenuBarExtra` for menu bar apps
- Use `Window` and `WindowGroup` for multi-window support
- Use `.keyboardShortcut()` for keyboard shortcuts
- Support dark/light mode; use semantic colors (`.primary`, `.secondary`); do not hardcode color values
