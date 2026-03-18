# TypeScript 规范

## 类型系统

- 禁止 `any`。不确定类型时用 `unknown`，然后通过类型守卫收窄
- 对象形状用 `interface`，联合类型 / 交叉类型 / 映射类型用 `type`
- 公共函数必须有显式返回类型，内部函数可以依赖推断
- 用 `readonly` 标记不会被修改的属性和参数
- 善用内置工具类型：`Partial<T>`、`Pick<T, K>`、`Omit<T, K>`、`Record<K, V>`
- 泛型变量名要有意义：`TItem` 而非裸 `T`（单泛型参数除外）

```typescript
// 禁止
function parse(data: any): any { ... }

// 正确
function parse(data: unknown): ParseResult { ... }
```

## 命名

- 类型和接口：`PascalCase`（`UserProfile`、`ApiResponse`）
- 变量和函数：`camelCase`（`getUserById`、`isValid`）
- 常量：`UPPER_CASE`（`MAX_RETRY_COUNT`）
- 枚举成员：`PascalCase`（`Status.Active`）
- 泛型参数：单字母大写或 `T` 前缀（`T`、`TKey`、`TValue`）

## 模块组织

- 一个文件一个主要导出（组件、类、或一组紧密相关的函数）
- 类型定义放在使用它的文件顶部；跨文件共享的类型放 `types/` 目录
- 不要用 `index.ts` 桶导出，它会引起循环依赖和 tree-shaking 问题，直接从源文件导入
- `import type` 和值导入分开写

```typescript
import type { UserProfile } from './types/user'
import { formatDate } from './utils/date'
```

## 函数

- 优先用箭头函数，需要 `this` 绑定时才用 `function`
- 优先 `async/await`，禁止 `.then()` 链超过 2 层
- 错误处理用具体类型，不要 `catch(e: any)`

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

## 禁止的写法

- 禁止 `// @ts-ignore` 和 `// @ts-expect-error`（除非有注释说明原因）
- 禁止 `as` 类型断言（除非从 `unknown` 收窄且有充分理由）
- 禁止 `!` 非空断言（用可选链 `?.` 或提前判空）
- 禁止 `enum`（用 `as const` 对象或联合类型替代，避免运行时开销）

```typescript
// 禁止
enum Status { Active, Inactive }

// 正确
const Status = { Active: 'active', Inactive: 'inactive' } as const
type Status = typeof Status[keyof typeof Status]
```
