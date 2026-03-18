# JavaScript Guidelines

> If the project supports TypeScript, prefer TypeScript -- see typescript.md. These guidelines apply to pure JS projects or JS files within JS/TS hybrid projects.

## Core Principles

- Use ES2022+ syntax; no `var` (prefer `const`, use `let` only when necessary)
- No `==` or `!=`; always use `===` and `!==`
- Use optional chaining `?.` and nullish coalescing `??` instead of manual null checks
- Add type annotations via JSDoc for all public functions

## Modules

- Use ES Modules (`import`/`export`) only; no `require`/`module.exports`
- Prefer named exports; default exports are reserved for one main export per file

```javascript
// 禁止
const utils = require('./utils')
module.exports = { ... }

// 正确
import { formatDate } from './utils.js'
export function processData(data) { ... }
```

## Functions

- Prefer arrow functions; use `function` only when `this` binding or hoisting is needed
- Prefer `async/await`; do not chain more than 2 levels of `.then()`
- Use destructuring for parameters and return values
- Use default parameters instead of internal null checks

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

## Arrays and Objects

- Use the spread operator for copying and merging: `[...arr]`, `{ ...obj }`
- Use destructuring assignment: `const { name, age } = user`
- Prefer `map`, `filter`, `reduce` for array operations; avoid `for` loops with `push`
- Do not mutate function parameter objects directly

## JSDoc Type Annotations

Pure JS projects must annotate public APIs with JSDoc:

```javascript
/**
 * @param {string} id - 用户 ID
 * @returns {Promise<User>} 用户信息
 * @throws {NotFoundError} 用户不存在时抛出
 */
export async function fetchUser(id) { ... }

/** @typedef {{ name: string, age: number }} UserProfile */
```

## Error Handling

- Async operations must use try-catch or `.catch()`
- Error messages must include context
- No empty catch blocks
