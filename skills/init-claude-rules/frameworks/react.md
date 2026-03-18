# React Guidelines

## Basic Component Rules

- Use only function components; class components are forbidden
- A single component file must not exceed 200 lines
- Components are responsible only for UI; extract business logic into custom hooks
- Complex expressions in JSX are forbidden; extract them into variables or functions
- A file should export only one component (except small helper components)

## State Management

- Use `useState` for component-local state
- Use `useReducer` for complex state logic
- Keep state as close to where it is used as possible; do not lift state unnecessarily
- Use Context (for small amounts of global state) or Zustand/Jotai (for complex scenarios) for cross-component sharing
- Prop drilling beyond 2 levels is forbidden

```tsx
// Forbidden: prop drilling
<GrandParent user={user}>
  <Parent user={user}>
    <Child user={user} />  // 3 levels deep
  </Parent>
</GrandParent>

// Correct: Context
const UserContext = createContext<User | null>(null)
const useUser = () => {
  const user = useContext(UserContext)
  if (!user) throw new Error('useUser must be used within UserProvider')
  return user
}
```

## Hook Rules

- Custom hook file names must have the `use` prefix: `useAuth.ts`
- A hook should do one thing only
- `useEffect` must have a correct dependency array; suppressing with `// eslint-disable-next-line` is forbidden
- `useEffect` with side effects must return a cleanup function
- Passing an async function directly to `useEffect` is forbidden

```typescript
// Forbidden
useEffect(async () => {
  const data = await fetchData()
  setData(data)
}, [])

// Correct
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

## Performance

- Use `React.memo` only on components with actual performance issues; do not use it preemptively
- Use `useMemo` / `useCallback` only in the following scenarios:
  - Computationally expensive derived values
  - Dependencies of other hooks
  - Props passed to children wrapped with `React.memo`
- Lists must have stable, unique `key` values; using index is forbidden
- Use virtualization for large lists (`react-virtual` / `react-window`)

## Props

- Define with TypeScript interfaces, named `XxxProps`
- Prefer primitive types over objects for props
- Name callback props with `onXxx`: `onClick`, `onSubmit`

```typescript
interface UserCardProps {
  name: string
  email: string
  onEdit: (id: string) => void
}
```

## Error Handling

- Page-level components must have an Error Boundary
- Async operations must handle loading / error / empty states
- Error messages should be user-friendly; log raw errors to the console

## Styling

- Prefer Tailwind CSS
- Use CSS Modules or `clsx`/`cn` for class name concatenation when dynamic styles are needed
- Inline style objects are forbidden (unless the values are truly dynamically computed)
- `!important` is forbidden
