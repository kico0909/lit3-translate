
# 渲染

在组件中添加模板以定义它的渲染内容。模板可以包含表达式，用作动态内容的占位符。

要为 Lit 组件定义模板，可以添加 `render()` 方法：

```typescript
import {LitElement, html} from 'lit';
import {customElement} from 'lit/decorators.js';
​
@customElement('my-element')
class MyElement extends LitElement {
​
  render(){
    return html\`<p>Hello from my template.</p>\`;
  }
}
```

将你的模板写在 JavaScript 标签模板字符串中，并使用 Lit 的 `html` 标签函数。

Lit 模板可以包含 JavaScript 表达式。你可以使用表达式来设置文本内容、属性、属性值和事件监听器。`render()` 方法中还可以包含其他 JavaScript 代码，例如可以创建局部变量用于表达式中。

## 可渲染的值

通常，组件的 `render()` 方法返回一个 `TemplateResult` 对象（即 `html` 函数返回的类型）。然而，它也可以返回任何可以作为 HTML 元素子节点渲染的内容：

- 基本值，如字符串、数字或布尔值。
- 由 `html` 函数创建的 `TemplateResult` 对象。
- DOM 节点。
- 特殊值 `nothing` 和 `noChange`。
- 由上述类型组成的数组或可迭代对象。

这几乎与可以渲染到 Lit 子表达式的值集相同。唯一的区别是，子表达式还可以渲染 `SVGTemplateResult`，由 `svg` 函数返回的类型。此类模板结果只能渲染为 `<svg>` 元素的子节点。

## 编写良好的 `render()` 方法

为了充分利用 Lit 的函数式渲染模型，`render()` 方法应遵循以下准则：

- 避免改变组件的状态。
- 避免产生任何副作用。
- 仅使用组件的属性作为输入。
- 在相同属性值下返回相同的结果。

遵循这些准则可以保持模板的确定性，并使代码更易于理解。

大多数情况下，应避免在 `render()` 之外进行 DOM 更新。相反，应将组件的模板表示为其状态的函数，并将状态捕获在属性中。

例如，如果组件需要在接收到事件时更新 UI，建议在事件监听器中设置响应式属性，该属性用于 `render()`，而不是直接操作 DOM。

有关更多信息，请参阅[响应式属性](https://lit.dev/docs/components/properties/).

## 组合模板

你可以将 Lit 模板组合到其他模板中。以下示例将一个名为 `<my-page>` 的组件的模板由页面的 `header`、`footer` 和 `main content` 等较小模板组成：

```typescript
import {LitElement, html} from 'lit';
import {customElement, property} from 'lit/decorators.js';
​
@customElement('my-page')
class MyPage extends LitElement {
​
  @property({attribute: false})
  article = {
    title: 'My Nifty Article',
    text: 'Some witty text.',
  };
​
  headerTemplate() {
    return html\`<header>\${this.article.title}</header>\`;
  }
​
  articleTemplate() {
    return html\`<article>\${this.article.text}</article>\`;
  }
​
  footerTemplate() {
    return html\`<footer>Your footer here.</footer>\`;
  }
​
  render() {
    return html\`
      \${this.headerTemplate()}
      \${this.articleTemplate()}
      \${this.footerTemplate()}
    \`;
  }
}
```

在该示例中，单个模板定义为实例方法，因此子类可以扩展此组件，并重写一个或多个模板。

你还可以通过导入其他元素并在模板中使用它们来组合模板：

```typescript
import {LitElement, html} from 'lit';
import {customElement} from 'lit/decorators.js';
​
import './my-header.js';
import './my-article.js';
import './my-footer.js';
​
@customElement('my-page')
class MyPage extends LitElement {
  render() {
    return html\`
      <my-header></my-header>
      <my-article></my-article>
      <my-footer></my-footer>
    \`;
  }
}
```

## 何时渲染模板

Lit 组件在初次添加到页面的 DOM 中时会渲染其模板。在初次渲染之后，组件的响应式属性发生任何变化都会触发更新周期，重新渲染组件。

Lit 批量处理更新以最大化性能和效率。一次性设置多个属性仅会触发一次更新，并且会在微任务中异步执行。

在更新过程中，仅更改的 DOM 部分会被重新渲染。虽然 Lit 模板看起来像是字符串插值，但 Lit 仅会解析并创建一次静态 HTML，并在后续更新时仅更新表达式中的变化值，从而提高更新效率。

有关更新周期的更多信息，请参阅[属性变化时会发生什么](https://lit.dev/docs/components/properties/#what-happens-when-properties-change).

## DOM 封装

Lit 使用 Shadow DOM 来封装组件渲染的 DOM。Shadow DOM 允许元素创建一个独立于主文档树的隔离 DOM 树。它是 Web 组件规范的核心特性之一，可以实现互操作性、样式封装以及其他好处。

有关 Shadow DOM 的更多信息，请参阅 [Shadow DOM v1: Self-Contained Web Components](https://developers.google.com/web/fundamentals/web-components/shadowdom) 。

有关如何在组件中使用 Shadow DOM 的更多信息，请参阅[使用 Shadow DOM](https://lit.dev/docs/components/shadow-dom/).
