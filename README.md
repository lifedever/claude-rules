# Claude Rules

面向 Claude Code 的编码规范模板库。通过 **base + language + framework** 三层组合，为项目生成针对性的 CLAUDE.md，让 AI 写出符合规范的代码。

## 解决什么问题

AI 编码助手在面对旧项目时，倾向于模仿已有代码风格——包括坏习惯。本规范库的核心理念：

> **不要模仿旧代码，按规范重构。**

每条规则都是可执行的具体指令（而非"使用适当的 XX"这种空话），并附有"禁止/正确"对比示例。

## 目录结构

```
claude-rules/
├── base/                   # 所有项目通用（必选）
│   ├── core.md             # 核心原则：旧代码态度、质量硬指标、命名、架构
│   └── git.md              # Git 提交 Message 规范
│
├── languages/              # 按语言选择
│   ├── typescript.md       # 禁 any/enum/桶导出，as const，import type
│   ├── javascript.md       # ES2022+，JSDoc 类型注释，ESM only
│   ├── java.md             # Java 17+ record/sealed/pattern matching，Optional
│   ├── kotlin.md           # 空安全、协程结构化并发、sealed class
│   ├── swift.md            # guard let、async/await、actor、Protocol
│   ├── python.md           # ruff、类型标注、Protocol、uv/poetry
│   ├── html.md             # 语义化标签、可访问性、禁止 div 滥用
│   └── css.md              # 自定义属性、Flexbox/Grid、BEM、现代特性
│
└── frameworks/             # 按框架选择
    ├── vue.md              # script setup、ref vs reactive、composable 结构
    ├── react.md            # hooks 规则、useEffect 正确写法、状态分层
    ├── swiftui.md          # @Observable（非旧 ObservableObject）、SwiftData
    ├── springboot.md       # 分层架构、DTO、全局异常、构造器注入
    └── tauri.md            # Command 设计、service 封装、安全配置
```

## 三层架构

```
┌─────────────────────────────────────┐
│           base (必选)                │  core.md + git.md
│  旧代码态度 / 质量硬指标 / 命名 / 架构  │  所有项目都适用
├─────────────────────────────────────┤
│         language (按需)              │  typescript.md / java.md / ...
│  类型系统 / 命名约定 / 语言特性        │  根据项目语言选择
├─────────────────────────────────────┤
│        framework (按需)              │  vue.md / react.md / ...
│  组件规范 / 状态管理 / 架构模式        │  根据项目框架选择
└─────────────────────────────────────┘
```

**规则优先级**：framework > language > base（具体规则覆盖通用规则）

## 使用方式

将 base + 对应的 language + framework 文件内容拼接，写入项目根目录的 `CLAUDE.md` 即可。

**示例：Vue 3 + TypeScript 项目**

```bash
cat base/core.md base/git.md languages/typescript.md frameworks/vue.md > /path/to/your-project/CLAUDE.md
```

**示例：SwiftUI macOS 项目**

```bash
cat base/core.md base/git.md languages/swift.md frameworks/swiftui.md > /path/to/your-project/CLAUDE.md
```

**示例：Spring Boot + Kotlin 项目**

```bash
cat base/core.md base/git.md languages/kotlin.md frameworks/springboot.md > /path/to/your-project/CLAUDE.md
```

## 与 Cursor Rules 的区别

| 对比项 | Cursor Rules (.mdc) | Claude Rules (.md) |
|--------|---------------------|---------------------|
| 适用工具 | Cursor IDE | Claude Code CLI / IDE |
| 文件格式 | .mdc + frontmatter | 纯 Markdown |
| 加载方式 | IDE 自动按 glob 匹配 | 拼接为单个 CLAUDE.md |
| 核心理念 | 遵循项目既有风格 | **打破旧代码坏习惯** |
| 规则风格 | "使用适当的 XX" | 具体指令 + 禁止/正确示例 |

## 规范设计原则

1. **可执行**：每条规则 AI 能直接执行，不含模糊表述
2. **有示例**：关键规则都有"禁止"和"正确"的代码对比
3. **有量化**：函数 ≤30 行、文件 ≤300 行、嵌套 ≤3 层、参数 ≤4 个
4. **不过时**：使用各语言/框架的现代 API（@Observable、record、as const 等）
5. **不空洞**：不写"正确处理错误"，而是写具体怎么处理

## 贡献

欢迎提交新的 language 或 framework 规范。请遵循：

- 规则必须具体可执行，禁止"使用适当的/正确的"这类表述
- 关键规则附带代码示例（禁止写法 + 正确写法）
- 推荐该语言/框架的现代写法，不要写已过时的 API

## 参考

- [flyeric0212/cursor-rules](https://github.com/flyeric0212/cursor-rules) — Cursor IDE 规则模板库

## License

[MIT](LICENSE)
