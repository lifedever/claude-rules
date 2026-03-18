# Tauri 规范

## 架构

- 前端负责 UI，Rust 后端负责系统操作（文件、网络、进程等）
- 前后端通过 Tauri Command 通信，不要在前端用 `fetch` 绕过 Tauri
- Command 是唯一的前后端桥梁，保持 Command 接口薄而清晰

## Command 设计

- Command 命名用动词：`get_user`、`save_config`、`open_file`
- 参数和返回值必须实现 `Serialize`/`Deserialize`
- 错误统一返回 `Result<T, String>` 或自定义错误类型
- 前端用 `invoke<T>()` 泛型调用，不要用 `any`

```rust
// Rust 侧
#[tauri::command]
async fn get_config(app: AppHandle) -> Result<AppConfig, String> {
    load_config(&app).map_err(|e| e.to_string())
}
```

```typescript
// 前端侧
import { invoke } from '@tauri-apps/api/core'

const config = await invoke<AppConfig>('get_config')
```

## 前端（适配 Vue/React）

- 前端框架规则参考对应的 vue.md 或 react.md
- Tauri API 调用封装到独立的 service 层，组件不直接调用 `invoke`
- 文件操作用 `@tauri-apps/plugin-fs`，不要用 Web File API

```typescript
// 封装 Tauri 调用
// services/config.ts
export async function getConfig(): Promise<AppConfig> {
  return invoke<AppConfig>('get_config')
}

// 组件里使用 service，不直接用 invoke
const config = await getConfig()
```

## 窗口管理

- 多窗口用 `WebviewWindow` 创建
- 窗口间通信用 Tauri Event（`emit` / `listen`）
- 窗口配置在 `tauri.conf.json` 里声明，不要在代码里硬编码尺寸

## 安全

- `tauri.conf.json` 里的 `allowlist` 只开启需要的权限
- 禁止开启 `shell.open` 的通配符模式
- Command 参数做基本校验，不要信任前端传来的路径
