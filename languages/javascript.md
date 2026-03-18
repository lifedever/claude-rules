# JavaScript 规范

> 如果项目支持 TypeScript，应优先使用 TypeScript，参考 typescript.md。本规范用于纯 JS 项目或 JS/TS 混合项目中的 JS 文件。

## 基本原则

- 使用 ES2022+ 语法，禁止 `var`（用 `const` 优先，必要时 `let`）
- 禁止 `==` 和 `!=`，只用 `===` 和 `!==`
- 用可选链 `?.` 和空值合并 `??` 替代手动判空
- 用 JSDoc 为公共函数添加类型注释

## 模块

- 只用 ES Module（`import`/`export`），禁止 `require`/`module.exports`
- 命名导出优先，默认导出仅限一个文件一个主导出

```javascript
// 禁止
const utils = require('./utils')
module.exports = { ... }

// 正确
import { formatDate } from './utils.js'
export function processData(data) { ... }
```

## 函数

- 优先箭头函数，需要 `this` 绑定或函数提升时才用 `function`
- 优先 `async/await`，禁止 `.then()` 链超过 2 层
- 用解构取参数和返回值
- 用默认参数替代内部判空

```javascript
// 禁止
function createUser(options) {
    const name = options.name || 'Unknown'
    const age = options.age || 0
    ...
}

// 正确
const createUser = ({ name = 'Unknown', age = 0 } = {}) => {
    ...
}
```

## 数组和对象

- 用展开运算符拷贝和合并：`[...arr]`、`{ ...obj }`
- 用解构赋值：`const { name, age } = user`
- 数组操作优先用 `map`、`filter`、`reduce`，不要用 `for` 循环加 `push`
- 禁止直接修改函数参数对象

## JSDoc 类型注释

纯 JS 项目必须用 JSDoc 标注公共 API：

```javascript
/**
 * @param {string} id - 用户 ID
 * @returns {Promise<User>} 用户信息
 * @throws {NotFoundError} 用户不存在时抛出
 */
export async function fetchUser(id) { ... }

/** @typedef {{ name: string, age: number }} UserProfile */
```

## 错误处理

- 异步操作必须 try-catch 或 `.catch()`
- 错误信息包含上下文
- 禁止空 catch 块
