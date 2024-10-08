
# 样式 (Styles)

组件的模板会被渲染到其 shadow root 中。添加到组件中的样式会自动作用于 shadow root 范围内，并且只影响组件 shadow root 中的元素。

Shadow DOM 提供了强大的样式封装功能。如果 Lit 不使用 Shadow DOM，你就需要非常小心地避免样式意外地作用到组件之外的元素，例如组件的父元素或子元素。这可能需要编写冗长且难以使用的类名。通过使用 Shadow DOM，Lit 可以确保你编写的任何选择器都只作用于组件的 shadow root 中的元素。

## 向组件添加样式

你可以在静态 `styles` 类字段中使用 `css` 函数定义作用域样式。以这种方式定义样式可以获得最佳的性能表现：

```javascript
import {LitElement, html, css} from 'lit';
import {customElement} from 'lit/decorators.js';
​
@customElement('my-element')
export class MyElement extends LitElement {
  static styles = css`
    p {
      color: green;
    }
  `;
  protected render() {
    return html`<p>我是绿色的！</p>`;
  }
}
```

添加到组件中的样式会通过 shadow DOM 进行作用域隔离。有关 shadow DOM 的快速概述，请参阅 [Shadow DOM](#)。

静态 `styles` 类字段的值可以是：

- 单个模板字符串。
  
  ```javascript
  static styles = css`...`;
  ```

- 一组模板字符串。

  ```javascript
  static styles = [ css`...`, css`...`];
  ```

静态 `styles` 类字段几乎总是向组件添加样式的最佳方式，但在某些情况下你无法以这种方式处理样式 —— 例如，需要对样式进行实例化定制。有关其他添加样式的方法，请参阅[在模板中定义作用域样式](#)。

### 在静态样式中使用表达式

静态样式适用于组件的所有实例。CSS 中的任何表达式都会被求值一次，并且会被所有实例复用。

对于基于树结构或每个实例的样式定制，可以使用 CSS 自定义属性来允许元素进行主题化。

为了防止 Lit 组件评估潜在的恶意代码，`css` 标签只允许嵌入的表达式是被 `css` 标签标记的字符串或数字。

```javascript
const mainColor = css`red`;
...
static styles = css`
  div { color: ${mainColor} }
`;
```

此限制旨在保护应用程序免受安全漏洞的影响，例如恶意样式甚至恶意代码可能会通过不受信任的来源（如 URL 参数或数据库值）被注入。

如果你必须在 `css` 字符串中使用一个非 `css` 字符串的表达式，并且你确信该表达式是由你自己代码中定义的可信源生成的常量，那么你可以使用 `unsafeCSS` 函数来包装该表达式：

```javascript
const mainColor = 'red';
...
static styles = css`
  div { color: ${unsafeCSS(mainColor)} }
`;
```

仅使用 `unsafeCSS` 来处理可信输入。注入未经处理的 CSS 是一种安全风险。例如，恶意 CSS 可以通过添加指向第三方服务器的图片 URL 来“回传”信息。

### 从超类继承样式

通过使用一组模板字符串，组件可以继承超类中的样式，并添加自己的样式：

```javascript
import {css} from 'lit';
import {customElement} from 'lit/decorators.js';
import {SuperElement} from './super-element.js';
​
@customElement('my-element')
export class MyElement extends SuperElement {
  static styles = [
    SuperElement.styles,
    css`div {
      color: red;
    }`
  ];
}
```

你也可以在 JavaScript 中使用 `super.styles` 引用超类的样式属性。如果你使用 TypeScript，我们建议避免使用 `super.styles`，因为编译器有时无法正确转换它。像示例中那样显式地引用超类可以避免此问题。

在 TypeScript 中编写打算被子类化的组件时，静态 `styles` 字段应明确地定义为 `CSSResultGroup` 类型，以便用户可以使用数组覆盖样式：

```javascript
// 防止 TypeScript 将 `styles` 的类型缩小为 `CSSResult`
// 以便子类可以分配诸如 `[SuperElement.styles, css`...`]`;
static styles: CSSResultGroup = css`...`;
```

## 共享样式

你可以通过创建一个导出标记样式的模块，在组件之间共享样式：

```javascript
export const buttonStyles = css`
  .blue-button {
    color: white;
    background-color: blue;
  }
  .blue-button:disabled {
    background-color: grey;
  }`;
```

然后，你的元素可以导入样式，并将其添加到静态 `styles` 类字段中：

```javascript
import { buttonStyles } from './button-styles.js';

class MyElement extends LitElement {
  static styles = [
    buttonStyles,
    css`
      :host { display: block;
        border: 1px solid black;
      }`
  ];
}
```

## 在样式中使用 Unicode 转义

CSS 的 Unicode 转义序列是一个反斜杠加上四个或六个十六进制数字，例如 `\2022` 表示一个项目符号字符。这与 JavaScript 的已弃用的八进制转义序列格式相似，因此在 `css` 标记的模板字符串中使用这些序列会导致错误。

有两种解决方法可以将 Unicode 转义添加到样式中：

1. 添加第二个反斜杠（例如 `\\2022`）。
2. 使用 JavaScript 的转义序列，从 `\u` 开始（例如 `\u2022`）。

```javascript
static styles = css`
  div::before {
    content: '\u2022';
  }
`;
```

...

（继续翻译剩余内容）

