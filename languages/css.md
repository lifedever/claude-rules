# CSS 规范

> 使用 Tailwind CSS 的项目优先用 Tailwind 类名，参考 frameworks/tailwind.md。本规范用于原生 CSS / SCSS / CSS Modules 场景。

## 基本原则

- 用 CSS 自定义属性（`--var`）管理设计令牌（颜色、间距、字号、圆角）
- 禁止硬编码颜色值散落各处，统一定义在 `:root` 或主题变量中
- 移动优先：默认写移动端样式，用 `min-width` 媒体查询向上适配

```css
/* 正确：设计令牌集中管理 */
:root {
  --color-primary: #3b82f6;
  --color-text: #1f2937;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --radius-md: 0.5rem;
}

.button {
  background: var(--color-primary);
  padding: var(--spacing-sm) var(--spacing-md);
  border-radius: var(--radius-md);
}
```

## 布局

- 用 Flexbox 做一维布局（导航栏、卡片行）
- 用 Grid 做二维布局（页面骨架、网格列表）
- 禁止用 `float` 做布局（仅限文字环绕图片场景）
- 禁止用固定像素宽度做响应式（用 `%`、`vw`、`fr`、`min()`/`max()`/`clamp()`）

```css
/* 禁止 */
.container { width: 1200px; margin: 0 auto; }

/* 正确 */
.container { width: min(90%, 1200px); margin-inline: auto; }
```

## 命名（BEM 或语义化）

- 类名用小写连字符：`.user-card`、`.nav-item`
- 避免过于通用的类名：`.box`、`.wrapper`、`.container`（除非有明确作用域）
- 禁止用 ID 选择器做样式（`#header { }`），ID 只用于 JS 或锚点
- 嵌套选择器不超过 3 层

```css
/* 禁止：嵌套太深 */
.page .content .sidebar .menu .item .link { ... }

/* 正确：扁平化 */
.sidebar-menu-link { ... }
```

## 现代 CSS 特性

- 用 `gap` 替代 margin 做间距（Flex/Grid 子元素之间）
- 用 `aspect-ratio` 替代 padding-top hack
- 用 `container query`（`@container`）做组件级响应式（浏览器支持的场景）
- 用 `color-mix()` 做颜色变体
- 用 `prefers-color-scheme` 支持暗色模式

## 禁止的写法

- 禁止 `!important`（除非覆盖第三方库样式，且必须加注释说明原因）
- 禁止 `*` 通配选择器做样式（reset 除外）
- 禁止用 `px` 做字号（用 `rem`），间距可以用 `px` 或 `rem`
- 禁止行内样式（`style=""`），除非是动态计算值（如 JS 控制位移）

## 动画

- 简单过渡用 `transition`，复杂动画用 `@keyframes`
- 只动画 `transform` 和 `opacity`（GPU 加速），避免动画 `width`/`height`/`margin`
- 尊重 `prefers-reduced-motion`

```css
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```
