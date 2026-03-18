# Tauri Guidelines

## Architecture

- The frontend handles UI; the Rust backend handles system operations (files, network, processes, etc.)
- Frontend and backend communicate through Tauri Commands; do not use `fetch` in the frontend to bypass Tauri
- Commands are the sole bridge between frontend and backend; keep Command interfaces thin and clear

## Command Design

- Name Commands with verbs: `get_user`, `save_config`, `open_file`
- Parameters and return values must implement `Serialize`/`Deserialize`
- Return errors uniformly as `Result<T, String>` or a custom error type
- Use the generic `invoke<T>()` on the frontend; do not use `any`

```rust
// Rust side
#[tauri::command]
async fn get_config(app: AppHandle) -> Result<AppConfig, String> {
    load_config(&app).map_err(|e| e.to_string())
}
```

```typescript
// Frontend side
import { invoke } from '@tauri-apps/api/core'

const config = await invoke<AppConfig>('get_config')
```

## Frontend (Vue/React Integration)

- Follow the corresponding vue.md or react.md for frontend framework rules
- Wrap Tauri API calls in a dedicated service layer; components should not call `invoke` directly
- Use `@tauri-apps/plugin-fs` for file operations; do not use the Web File API

```typescript
// Wrap Tauri calls
// services/config.ts
export async function getConfig(): Promise<AppConfig> {
  return invoke<AppConfig>('get_config')
}

// Use the service in components, not invoke directly
const config = await getConfig()
```

## Window Management

- Use `WebviewWindow` to create multiple windows
- Use Tauri Events (`emit` / `listen`) for inter-window communication
- Declare window configuration in `tauri.conf.json`; do not hardcode dimensions in code

## Security

- In `tauri.conf.json`, only enable required permissions in `allowlist`
- Enabling wildcard mode for `shell.open` is forbidden
- Validate Command parameters on the backend; do not trust paths sent from the frontend
