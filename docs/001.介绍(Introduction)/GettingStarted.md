# 入门指南

有多种方式可以开始使用 Lit，包括使用 Playground、交互式教程，或将 Lit 安装到现有项目中。

## Lit Playground
通过交互式 Playground 和示例快速上手。从 "Hello World" 开始，然后进行定制或参考更多示例。

## 交互式教程
按照我们的分步教程，在几分钟内学习如何构建 Lit 组件。

## Lit 起始模板
我们提供 TypeScript 和 JavaScript 的起始模板用于创建独立的可复用组件。详见 [Starter Kits](https://lit.dev/docs/tools/starter-kits/)。

## 从 npm 本地安装
Lit 可以通过 npm 安装：

```bash
npm i lit
```

然后在 JavaScript 或 TypeScript 文件中引入：

```js
import {LitElement, html} from 'lit';
import {customElement, property} from 'lit/decorators.js';
```

## 使用 Bundles
Lit 也以单文件捆绑包的形式提供，适用于更灵活的开发流程（例如，不使用 npm 时）。捆绑包是无依赖的标准 JavaScript 模块，可以通过以下方式导入：

```html
<script type="module">
  import {LitElement, html} from 'https://cdn.jsdelivr.net/gh/lit/dist@3/core/lit-core.min.js';
</script>
```

## 将 Lit 添加到现有项目中
查看 [将 Lit 添加到现有项目](https://lit.dev/docs/tools/add-to-project/) 以获取详细的说明。

## Open WC 项目生成器
Open WC 项目生成器可以基于 Lit 生成应用项目的脚手架。详见 [Open WC 项目生成器](https://open-wc.org/docs/development/generator/)。