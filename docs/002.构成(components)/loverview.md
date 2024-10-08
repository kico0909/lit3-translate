
# 组件概览

Lit 组件是可复用的 UI 片段。你可以将 Lit 组件视为一个容器，它包含一些状态，并根据这些状态展示相应的 UI。它可以响应用户输入、触发事件——几乎任何 UI 组件应具备的功能。而且，Lit 组件本质上是一个 HTML 元素，因此具备所有标准元素的 API。

## 创建 Lit 组件涉及以下几个核心概念：

- **定义组件**：Lit 组件作为自定义元素注册到浏览器中。
- **渲染**：组件通过 `render` 方法渲染其内容，并在其中定义组件的模板。
- **响应式属性**：组件的属性用于存储状态。更改响应式属性会触发更新周期，重新渲染组件。
- **样式**：组件可以定义封装样式，控制自身的外观。
- **生命周期**：Lit 提供了生命周期回调，可以在组件被添加到页面或更新时执行相应代码。

## 示例组件

以下是一个简单的组件示例：

```typescript
import {LitElement, css, html} from 'lit';
import {customElement, property} from 'lit/decorators.js';

@customElement('simple-greeting')
export class SimpleGreeting extends LitElement {
  // 使用普通 CSS 定义组件的局部样式
  static styles = css`
    :host {
      color: blue;
    }
  `;

  // 声明响应式属性
  @property()
  name?: string = 'World';

  // 渲染 UI，根据组件状态动态更新
  render() {
    return html`<p>Hello, \${this.name}!</p>`;
  }
}
```
