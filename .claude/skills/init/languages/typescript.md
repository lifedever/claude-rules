# TypeScript Guidelines

## Type System

- No `any`. Use `unknown` when the type is uncertain, then narrow with type guards
- Use `interface` for object shapes; use `type` for unions / intersections / mapped types
- Public functions must have explicit return types; internal functions may rely on inference
- Mark properties and parameters that won't be mutated with `readonly`
- Leverage built-in utility types: `Partial<T>`, `Pick<T, K>`, `Omit<T, K>`, `Record<K, V>`
- Generic parameter names should be meaningful: `TItem` rather than bare `T` (single generic parameter excepted)

```typescript
// 禁止
function parse(data: any): any { ... }

// 正确
function parse(data: unknown): ParseResult { ... }
```

## Naming

- Types and interfaces: `PascalCase` (`UserProfile`, `ApiResponse`)
- Variables and functions: `camelCase` (`getUserById`, `isValid`)
- Constants: `UPPER_CASE` (`MAX_RETRY_COUNT`)
- Enum members: `PascalCase` (`Status.Active`)
- Generic parameters: single uppercase letter or `T` prefix (`T`, `TKey`, `TValue`)

## Module Organization

- One primary export per file (a component, a class, or a group of closely related functions)
- Place type definitions at the top of the file that uses them; shared cross-file types go in a `types/` directory
- Do not use `index.ts` barrel exports -- they cause circular dependencies and tree-shaking issues; import directly from source files
- Separate `import type` from value imports

```typescript
import type { UserProfile } from './types/user'
import { formatDate } from './utils/date'
```

## Functions

- Prefer arrow functions; use `function` only when `this` binding is needed
- Prefer `async/await`; do not chain more than 2 levels of `.then()`
- Handle errors with specific types; do not `catch(e: any)`

```typescript
// 禁止
fetchData().then(res => process(res)).then(data => save(data)).catch(e => console.log(e))

// 正确
try {
  const res = await fetchData()
  const data = process(res)
  await save(data)
} catch (error) {
  if (error instanceof NetworkError) {
    showNetworkError(error.message)
  }
  throw error
}
```

## Prohibited Patterns

- No `// @ts-ignore` or `// @ts-expect-error` (unless accompanied by a comment explaining why)
- No `as` type assertions (unless narrowing from `unknown` with good reason)
- No `!` non-null assertions (use optional chaining `?.` or early null checks instead)
- No `enum` (use `as const` objects or union types instead to avoid runtime overhead)

```typescript
// 禁止
enum Status { Active, Inactive }

// 正确
const Status = { Active: 'active', Inactive: 'inactive' } as const
type Status = typeof Status[keyof typeof Status]
```
