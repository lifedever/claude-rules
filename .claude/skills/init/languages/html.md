# HTML Guidelines

## Semantic Markup

- Use semantic tags to convey structure; no wrapping everything in `<div>` inside `<div>`
- Page skeleton: `<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`
- Content sections: `<section>` (grouped content with a heading), `<article>` (self-contained content)
- Interactive elements: use `<button>` for buttons, not `<div onclick>`; use `<a>` for links

```html
<!-- 禁止 -->
<div class="header">
  <div class="nav">
    <div class="nav-item" onclick="goHome()">首页</div>
  </div>
</div>

<!-- 正确 -->
<header>
  <nav>
    <a href="/">首页</a>
  </nav>
</header>
```

## Accessibility (a11y)

- Images must have an `alt` attribute; decorative images use `alt=""`
- Form controls must be associated with a `<label>` (via `for` attribute or nesting)
- Interactive elements must be keyboard-operable (`tabindex`, `role`)
- Use `aria-label` to describe buttons that have no visible text

```html
<!-- 禁止 -->
<img src="avatar.jpg">
<input type="text" placeholder="用户名">

<!-- 正确 -->
<img src="avatar.jpg" alt="用户头像">
<label for="username">用户名</label>
<input id="username" type="text" placeholder="请输入用户名">
```

## Structural Rules

- Only one `<h1>` per page; heading levels must not skip (h1 -> h2 -> h3)
- Use `<ul>`/`<ol>` + `<li>` for lists; do not simulate lists with divs
- Use `<table>` + `<thead>`/`<tbody>` for tabular data; do not simulate tables with div grids
- No trailing slash on void elements: `<img>` `<input>` `<br>` (HTML5 style)

## Attribute Order (Recommended)

```html
<element
  id=""
  class=""
  data-*=""
  src="" href="" for="" type=""
  aria-*="" role=""
  other-attributes
>
```

## Prohibited Patterns

- No inline `style` attributes (unless the value is dynamically computed)
- No inline event handlers like `onclick`/`onchange` (bind events in JS)
- No `<br>` for spacing (use CSS margin/padding)
- No `<table>` for page layout
