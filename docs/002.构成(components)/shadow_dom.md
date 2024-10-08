
# 使用 Shadow DOM (Working with Shadow DOM)

Lit 组件使用 Shadow DOM 来封装其 DOM。Shadow DOM 为元素添加了一个独立且封装的 DOM 树。DOM 封装是实现与页面上任何其他代码（包括其他 Web 组件或 Lit 组件）互操作的关键。

Shadow DOM 提供了三大优势：

1. **DOM 作用域**：像 `document.querySelector` 这样的 DOM API 无法找到组件 Shadow DOM 中的元素，因此全局脚本更难意外破坏你的组件。
2. **样式作用域**：你可以为 Shadow DOM 编写封装的样式，而这些样式不会影响整个 DOM 树的其余部分。
3. **组合性**：组件的 shadow root 包含其内部 DOM，与组件的子元素分离。你可以选择如何在组件内部 DOM 中渲染子元素。

有关 Shadow DOM 的更多信息：

- [Shadow DOM v1: 独立的 Web 组件](https://developers.google.com/web/fundamentals/web-components/shadowdom)（Web Fundamentals 文档）。
- [在 MDN 上使用 Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM)。

### 访问 Shadow DOM 中的节点

Lit 将组件渲染到 `renderRoot` 中，该 `renderRoot` 默认是一个 `shadow root`。要查找内部元素，可以使用 DOM 查询 API，例如 `this.renderRoot.querySelector()`。

`renderRoot` 应始终是 `shadow root` 或元素，它们共享类似 `.querySelectorAll()` 和 `.children` 的 API。

可以在组件初始渲染后（例如在 `firstUpdated` 中）查询内部 DOM，或者使用 getter 模式：

```javascript
firstUpdated() {
  this.staticNode = this.renderRoot.querySelector('#static-node');
}

get _closeButton() {
  return this.renderRoot.querySelector('#close-button');
}
```

LitElement 提供了一组装饰器，可以简化定义这种 getter 的方式。

#### `@query`、`@queryAll` 和 `@queryAsync` 装饰器

`@query`、`@queryAll` 和 `@queryAsync` 装饰器都提供了一种方便的方法来访问组件内部 DOM 中的节点。

#### `@query`

将类属性修改为 getter，返回来自渲染根节点的某个节点。当第二个可选参数为 `true` 时，只执行一次 DOM 查询并缓存结果。在节点不会更改的情况下，这可以作为一种性能优化。

```javascript
import {LitElement, html} from 'lit';
import {query} from 'lit/decorators/query.js';

class MyElement extends LitElement {
  @query('#first')
  _first;

  render() {
    return html`
      <div id="first"></div>
      <div id="second"></div>
    `;
  }
}
```

此装饰器等价于：

```javascript
get _first() {
  return this.renderRoot?.querySelector('#first') ?? null;
}
```

#### `@queryAll`

与 `@query` 相同，但它返回所有匹配的节点，而不是单个节点。等同于调用 `querySelectorAll`。

```javascript
import {LitElement, html} from 'lit';
import {queryAll} from 'lit/decorators/queryAll.js';

class MyElement extends LitElement {
  @queryAll('div')
  _divs;

  render() {
    return html`
      <div id="first"></div>
      <div id="second"></div>
    `;
  }
}
```

在此示例中，`_divs` 将返回模板中的两个 `<div>` 元素。对于 TypeScript，`@queryAll` 属性的类型为 `NodeListOf<HTMLElement>`。如果你确切知道将检索哪种节点，可以将类型设置得更具体一些：

```typescript
@queryAll('button')
_buttons!: NodeListOf<HTMLButtonElement>;
```

#### `@queryAsync`

类似于 `@query`，但它返回一个 Promise，在任何挂起的元素渲染完成后解析为该节点。代码可以使用此功能而无需等待 `updateComplete` Promise。

如果返回的节点可能因其他属性的更改而更改，则 `@queryAsync` 非常有用。

### 使用 `slot` 渲染子元素

你的组件可能接受子元素（如 `<ul>` 元素可以拥有 `<li>` 子元素）。

```html
<my-element>
  <p>一个子元素</p>
</my-element>
```

默认情况下，如果元素具有 `shadow tree`，其子元素不会渲染。

要渲染子元素，你的模板需要包含一个或多个 `<slot>` 元素，作为子节点的占位符。

#### 使用 `slot` 元素

要渲染元素的子元素，请在元素的模板中为它们创建一个 `<slot>`。子元素不会在 DOM 树中移动，但它们会像是 `<slot>` 的子元素一样进行渲染。例如：

```html
<my-element>
  <p>在 slot 中渲染我</p>
</my-element>
```

#### 使用具名 `slot`

要将子元素分配到特定的 `slot`，确保子元素的 `slot` 属性与 `slot` 元素的 `name` 属性匹配：

命名 `slot` 只接受具有匹配 `slot` 属性的子元素。

例如，`<slot name="one"></slot>` 只接受 `slot="one"` 属性的子元素。

```html
<my-element>
  <p slot="two">包括我在 slot "two" 中。</p>
</my-element>
```

#### 指定 `slot` 回退内容

你可以为 `slot` 指定回退内容。当没有子节点分配给 `slot` 时，回退内容会显示。

```html
<slot>我是回退内容</slot>
```

```
如果有子节点分配给插槽（slot），插槽的后备内容将不会渲染。默认的无名称插槽可以接收任意子节点。即使分配的节点仅包含空白字符的文本节点（例如 <example-element> </example-element>），它也不会渲染后备内容。

当使用 Lit 表达式作为自定义元素的子节点时，请确保在适当情况下使用不渲染的值，以便渲染任何插槽的后备内容。有关更多信息，请参阅 **移除子节点内容**。
```





### 访问插槽中的子元素 (Accessing slotted children)

要访问分配给 shadow root 中插槽的子元素，可以使用标准的 `slot.assignedNodes` 或 `slot.assignedElements` 方法，并配合 `slotchange` 事件。

例如，你可以创建一个 getter 来访问某个特定插槽中分配的元素：

```javascript
get _slottedChildren() {
  const slot = this.shadowRoot.querySelector('slot');
  return slot.assignedElements({flatten: true});
}
```

你也可以使用 `slotchange` 事件在分配的节点发生更改时执行操作。以下示例提取了所有分配到插槽的子元素的文本内容：

```javascript
handleSlotchange(e) {
  const childNodes = e.target.assignedNodes({flatten: true});
  // ... 对 childNodes 执行某些操作 ...
  this.allText = childNodes.map((node) => {
    return node.textContent ? node.textContent : ''
  }).join('');
}

render() {
  return html`<slot @slotchange=${this.handleSlotchange}></slot>`;
}
```

有关更多信息，请参阅 MDN 上的 [HTMLSlotElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSlotElement)。

##### `@queryAssignedElements` 和 `@queryAssignedNodes` 装饰器

`@queryAssignedElements` 和 `@queryAssignedNodes` 将类属性转换为 getter，返回对组件 shadow tree 中给定插槽调用 `slot.assignedElements` 或 `slot.assignedNodes` 的结果。使用这些装饰器可以查询分配给特定插槽的元素或节点。

它们都接受一个包含以下属性的可选对象：

- **flatten**：布尔值，指定是否通过替换任何子 `<slot>` 元素及其分配的节点来展平分配的节点。
- **slot**：指定要查询的插槽名称。留空则选择默认插槽。
- **selector**（仅 `queryAssignedElements`）：如果指定，则只返回匹配此 CSS 选择器的分配元素。

根据你希望查询分配到插槽的文本节点还是仅元素节点，来决定使用哪个装饰器。这取决于你的具体用例。

装饰器是一个提议的 JavaScript 特性，因此你需要使用 Babel 或 TypeScript 之类的编译器来使用装饰器。有关详细信息，请参阅[使用装饰器](https://lit.dev/docs/tools/decorators/#using-decorators)。

```typescript
@queryAssignedElements({slot: 'list', selector: '.item'})
_listItems!: Array<HTMLElement>;

@queryAssignedNodes({slot: 'header', flatten: true})
_headerNodes!: Array<Node>;
```

上面的示例等价于以下代码：

```javascript
get _listItems() {
  const slot = this.shadowRoot.querySelector('slot[name=list]');
  return slot.assignedElements().filter((node) => node.matches('.item'));
}

get _headerNodes() {
  const slot = this.shadowRoot.querySelector('slot[name=header]');
  return slot.assignedNodes({flatten: true});
}
```

### 自定义渲染根 (Customizing the render root)

每个 Lit 组件都有一个渲染根（render root）——它是一个 DOM 节点，用作内部 DOM 的容器。

默认情况下，`LitElement` 创建一个开放的 `shadowRoot` 并在其中进行渲染，从而生成以下 DOM 结构：

```html
<my-element>
  #shadow-root
    <p>子元素 1</p>
    <p>子元素 2</p>
</my-element>
```

有两种方式可以自定义 `LitElement` 使用的渲染根：

1. 设置 `shadowRootOptions`。
2. 实现 `createRenderRoot` 方法。

##### 设置 `shadowRootOptions`

自定义渲染根的最简单方法是设置静态属性 `shadowRootOptions`。`createRenderRoot` 的默认实现会将 `shadowRootOptions` 作为选项参数传递给 `attachShadow` 方法，用于创建组件的 `shadow root`。你可以设置它以自定义 `ShadowRootInit` 字典中允许的任何选项，例如 `mode` 和 `delegatesFocus`。

```typescript
class DelegatesFocus extends LitElement {
  static shadowRootOptions = {...LitElement.shadowRootOptions, delegatesFocus: true};
}
```

有关更多信息，请参阅 MDN 上的 [Element.attachShadow()](https://developer.mozilla.org/en-US/docs/Web/API/Element/attachShadow)。

##### 实现 `createRenderRoot`

`createRenderRoot` 的默认实现会创建一个开放的 `shadow root`，并将 `static styles` 类字段中设置的样式添加到其中。有关样式的更多信息，请参阅[样式文档](#)。

要自定义组件的渲染根，可以实现 `createRenderRoot` 并返回你希望模板渲染到的节点。

例如，要将模板渲染到主 DOM 树中作为元素的子节点，请实现 `createRenderRoot` 并返回 `this`。

> **注意：** 渲染到子元素（而非 Shadow DOM）通常不推荐使用。这样你的元素将无法访问 DOM 或样式作用域，并且无法将元素组合到其内部 DOM 中。

```typescript
import {LitElement, html} from 'lit';
import {customElement} from 'lit/decorators.js';

@customElement('default-root')
export class DefaultRoot extends LitElement {
  protected render() {
    return html`
      <p>默认情况下，模板会渲染到 Shadow DOM 中。</p>
    `;
  }
}
```
