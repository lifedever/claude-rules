# CSS Guidelines

> For projects using Tailwind CSS, prefer Tailwind utility classes -- see frameworks/tailwind.md. These guidelines apply to vanilla CSS / SCSS / CSS Modules.

## Core Principles

- Use CSS custom properties (`--var`) to manage design tokens (colors, spacing, font sizes, border radii)
- No hard-coded color values scattered throughout; define them centrally in `:root` or theme variables
- Mobile first: write mobile styles by default, then scale up with `min-width` media queries

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

## Layout

- Use Flexbox for one-dimensional layouts (nav bars, card rows)
- Use Grid for two-dimensional layouts (page skeletons, grid lists)
- No `float` for layout (only acceptable for text wrapping around images)
- No fixed pixel widths for responsive design (use `%`, `vw`, `fr`, `min()`/`max()`/`clamp()`)

```css
/* 禁止 */
.container { width: 1200px; margin: 0 auto; }

/* 正确 */
.container { width: min(90%, 1200px); margin-inline: auto; }
```

## Naming (BEM or Semantic)

- Class names use lowercase with hyphens: `.user-card`, `.nav-item`
- Avoid overly generic class names: `.box`, `.wrapper`, `.container` (unless clearly scoped)
- No ID selectors for styling (`#header { }`); IDs are reserved for JS or anchor links
- Selector nesting should not exceed 3 levels

```css
/* 禁止：嵌套太深 */
.page .content .sidebar .menu .item .link { ... }

/* 正确：扁平化 */
.sidebar-menu-link { ... }
```

## Modern CSS Features

- Use `gap` instead of margins for spacing between Flex/Grid children
- Use `aspect-ratio` instead of the padding-top hack
- Use `container query` (`@container`) for component-level responsiveness (where browser support allows)
- Use `color-mix()` for color variants
- Use `prefers-color-scheme` to support dark mode

## Prohibited Patterns

- No `!important` (unless overriding third-party library styles, and a comment must explain why)
- No `*` universal selector for styling (resets excepted)
- No `px` for font sizes (use `rem`); `px` or `rem` are fine for spacing
- No inline styles (`style=""`), unless the value is dynamically computed (e.g., JS-controlled positioning)

## Animations

- Use `transition` for simple transitions; use `@keyframes` for complex animations
- Only animate `transform` and `opacity` (GPU-accelerated); avoid animating `width`/`height`/`margin`
- Respect `prefers-reduced-motion`

```css
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```
