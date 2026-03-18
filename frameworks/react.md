# React 规范

## 组件基本规则

- 只用函数组件，禁止 class 组件
- 单个组件文件不超过 200 行
- 组件只负责 UI，业务逻辑提取到自定义 hook
- JSX 中禁止复杂表达式，提取为变量或函数
- 一个文件只导出一个组件（小型辅助组件除外）

## 状态管理

- 组件本地状态用 `useState`
- 复杂状态逻辑用 `useReducer`
- 状态放在最近的使用者，不要无谓上提
- 跨组件共享用 Context（少量全局状态）或 Zustand/Jotai（复杂场景）
- 禁止 prop drilling 超过 2 层

```tsx
// 禁止：prop drilling
<GrandParent user={user}>
  <Parent user={user}>
    <Child user={user} />  // 3 层了
  </Parent>
</GrandParent>

// 正确：Context
const UserContext = createContext<User | null>(null)
const useUser = () => {
  const user = useContext(UserContext)
  if (!user) throw new Error('useUser must be used within UserProvider')
  return user
}
```

## Hooks 规则

- 自定义 hook 文件名 `use` 前缀：`useAuth.ts`
- 一个 hook 只做一件事
- `useEffect` 必须有正确的依赖数组，禁止空注释 `// eslint-disable-next-line`
- `useEffect` 有副作用必须返回清理函数
- 禁止在 `useEffect` 里直接写 async 函数

```typescript
// 禁止
useEffect(async () => {
  const data = await fetchData()
  setData(data)
}, [])

// 正确
useEffect(() => {
  const controller = new AbortController()
  const load = async () => {
    try {
      const data = await fetchData({ signal: controller.signal })
      setData(data)
    } catch (error) {
      if (!controller.signal.aborted) setError(error)
    }
  }
  load()
  return () => controller.abort()
}, [])
```

## 性能

- `React.memo` 只用在确实有性能问题的组件，不要预防性使用
- `useMemo` / `useCallback` 只在以下场景使用：
  - 计算量大的派生值
  - 作为其他 hook 的依赖
  - 传给 `React.memo` 包裹的子组件
- 列表必须有稳定唯一的 `key`，禁止用 index
- 大列表用虚拟化（`react-virtual` / `react-window`）

## Props

- 用 TypeScript interface 定义，命名 `XxxProps`
- Props 能用基本类型就不用对象
- 回调 props 用 `onXxx` 命名：`onClick`、`onSubmit`

```typescript
interface UserCardProps {
  name: string
  email: string
  onEdit: (id: string) => void
}
```

## 错误处理

- 页面级组件必须有 Error Boundary
- 异步操作必须处理 loading / error / empty 三种状态
- 错误信息对用户友好，原始错误打到 console

## 样式

- 优先 Tailwind CSS
- 需要动态样式时用 CSS Modules 或 `clsx`/`cn` 拼接类名
- 禁止内联 style 对象（除非真正动态计算的值）
- 禁止 `!important`
