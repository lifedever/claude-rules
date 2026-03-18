# HTML 规范

## 语义化

- 用语义标签表达结构，禁止全篇 `<div>` 套 `<div>`
- 页面骨架：`<header>`、`<nav>`、`<main>`、`<aside>`、`<footer>`
- 内容区块：`<section>`（有标题的分组）、`<article>`（独立内容）
- 交互元素：按钮用 `<button>`，不要用 `<div onclick>`；链接用 `<a>`

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

## 可访问性（a11y）

- 图片必须有 `alt` 属性，装饰性图片用 `alt=""`
- 表单控件必须关联 `<label>`（用 `for` 属性或嵌套）
- 交互元素必须可键盘操作（`tabindex`、`role`）
- 用 `aria-label` 为无文字的按钮提供说明

```html
<!-- 禁止 -->
<img src="avatar.jpg">
<input type="text" placeholder="用户名">

<!-- 正确 -->
<img src="avatar.jpg" alt="用户头像">
<label for="username">用户名</label>
<input id="username" type="text" placeholder="请输入用户名">
```

## 结构规则

- 一个页面只有一个 `<h1>`，标题层级不跳级（h1 → h2 → h3）
- 列表内容用 `<ul>`/`<ol>` + `<li>`，不要用 div 模拟
- 表格数据用 `<table>` + `<thead>`/`<tbody>`，不要用 div 网格模拟表格
- 自闭合标签不加斜杠：`<img>` `<input>` `<br>`（HTML5 风格）

## 属性顺序（推荐）

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

## 禁止的写法

- 禁止内联 `style` 属性（除非动态计算值）
- 禁止内联 `onclick`/`onchange` 等事件（用 JS 绑定）
- 禁止用 `<br>` 做间距（用 CSS margin/padding）
- 禁止用 `<table>` 做页面布局
