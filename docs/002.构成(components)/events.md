
# 事件 (Events)

事件是元素传递变化的标准方式。这些变化通常由于用户交互而发生。例如，当用户点击按钮时，按钮会触发 `click` 事件；当用户在输入框中输入值时，输入框会触发 `change` 事件。

除了这些自动触发的标准事件之外，Lit 元素还可以触发自定义事件。例如，菜单元素可能触发一个事件，以指示所选项已更改；弹出框元素可能在弹出框打开或关闭时触发一个事件。

任何 JavaScript 代码（包括 Lit 元素本身）都可以监听事件并根据事件采取相应的操作。例如，工具栏元素可能在菜单项被选中时过滤列表；登录元素可能在处理登录按钮的点击事件时执行登录操作。


## 监听事件 (Listening to events)

除了标准的 `addEventListener` API 之外，Lit 还引入了一种声明式方式来添加事件监听器。

### 在元素模板中添加事件监听器 (Adding event listeners in the element template)

你可以在模板中使用 `@` 表达式为组件模板中的元素添加事件监听器。声明式事件监听器在模板渲染时被添加。

```javascript
import {LitElement, html} from 'lit';

export class MyElement extends LitElement {
  static properties = {
    count: {type: Number},
  };

  constructor() {
    super();
    this.count = 0;
  }

  render() {
    return html`
      <p><button @click="${this._increment}">点击我！</button></p>
      <p>点击次数: ${this.count}</p>
    `;
  }

  _increment(e) {
    this.count++;
  }
}

customElements.define('my-element', MyElement);
```

### 自定义事件监听器选项 (Customizing event listener options)

如果你需要自定义声明式事件监听器的选项（例如 `passive` 或 `capture`），可以使用 `@eventOptions` 装饰器。传递给 `@eventOptions` 的对象会作为选项参数传递给 `addEventListener`。

```javascript
import {LitElement, html} from 'lit';
import {eventOptions} from 'lit/decorators.js';

@eventOptions({passive: true})
private _handleTouchStart(e) {
  console.log(e.type);
}
```

装饰器是一个提议的 JavaScript 特性，因此你需要使用 Babel 或 TypeScript 之类的编译器来使用装饰器。有关详细信息，请参阅[启用装饰器](https://lit.dev/docs/tools/decorators/#using-decorators)。

如果你不使用装饰器，可以通过将对象传递给事件监听器表达式来自定义事件监听器选项。该对象必须具有 `handleEvent()` 方法，并可以包含 `addEventListener()` 中 `options` 参数中通常出现的任何选项。

```javascript
render() {
  return html`<button @click=${{handleEvent: () => this.onClick(), once: true}}>点击</button>`;
}
```

### 为组件或其 shadow root 添加事件监听器 (Adding event listeners to the component or its shadow root)

要监听从组件插槽子元素触发的事件以及通过组件模板渲染到 shadow DOM 的子元素触发的事件，你可以使用标准的 `addEventListener` DOM 方法在组件本身上添加监听器。有关更多详细信息，请参阅 MDN 上的 [EventTarget.addEventListener()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)。

在组件的构造函数中添加事件监听器是一个不错的选择：

```javascript
constructor() {
  super();
  this.addEventListener('click', (e) => console.log(e.type, e.target.localName));
}
```

将事件监听器添加到组件本身是一种事件委托（event delegation）形式，可以用来减少代码量或提升性能。有关详细信息，请参阅[事件委托](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#event_delegation)。通常情况下，当这样做时，事件的 `target` 属性用于基于哪个元素触发事件来采取相应的操作。

然而，当事件监听器在组件上监听来自组件 shadow DOM 的事件时，这些事件会被重新定向（retargeted）。这意味着事件的 `target` 会是组件本身。有关更多信息，请参阅[在 Shadow DOM 中处理事件](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM#events_in_shadow_dom)。

重新定向可能会干扰事件委托。为了避免这种情况，可以将事件监听器添加到组件的 `shadow root` 中。由于 `shadowRoot` 在构造函数中不可用，因此可以在 `createRenderRoot` 方法中添加事件监听器，如下所示。请注意，在 `createRenderRoot` 方法中返回 `shadow root` 非常重要。

```javascript
import {LitElement, html} from 'lit';

class MyElement extends LitElement {
  static properties = {
    hostName: {},
    shadowName: {},
  };

  constructor() {
    super();
    this.addEventListener('click', (e) => (this.hostName = e.target.localName));
    this.hostName = '';
    this.shadowName = '';
  }

  createRenderRoot() {
    const root = super.createRenderRoot();
    root.addEventListener('click', (e) => (this.shadowName = e.target.localName));
    return root;
  }

  render() {
    return html`
      <p><button>点击我！</button></p>
      <p>组件目标: ${this.hostName}</p>
      <p>Shadow DOM 目标: ${this.shadowName}</p>
    `;
  }
}

customElements.define('my-element', MyElement);
```

### 为其他元素添加事件监听器 (Adding event listeners to other elements)

如果组件向自身或其模板化 DOM 之外的任何内容添加事件监听器（例如，向 `Window`、`Document` 或主 DOM 中的某个元素添加事件监听器），应在 `connectedCallback` 中添加监听器，并在 `disconnectedCallback` 中移除它。

在 `disconnectedCallback` 中移除事件监听器可以确保当组件被销毁或从页面断开时，组件分配的任何内存都能被释放。

在 `connectedCallback` 中添加事件监听器（而不是例如在构造函数或 `firstUpdated` 中）可以确保组件在断开并随后重新连接到 DOM 时重新创建其事件监听器。

```javascript
connectedCallback() {
  super.connectedCallback();
  window.addEventListener('resize', this._handleResize);
}

disconnectedCallback() {
  window.removeEventListener('resize', this._handleResize);
  super.disconnectedCallback();
}
```

有关 `connectedCallback` 和 `disconnectedCallback` 的更多信息，请参阅 MDN 上的[自定义元素生命周期回调](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements#using_the_lifecycle_callbacks)文档。


### 优化性能 (Optimizing for performance)

添加事件监听器的速度非常快，通常不会引起性能问题。但是，对于高频使用且需要大量事件监听器的组件，可以通过事件委托减少使用的监听器数量，并在渲染后异步添加监听器来优化首次渲染性能。

### 事件委托 (Event delegation)

使用事件委托可以减少使用的事件监听器数量，从而提升性能。有时将事件处理集中到一个地方还可以简化代码。事件委托只能用于处理冒泡事件。有关冒泡的详细信息，请参阅[触发事件](https://developer.mozilla.org/en-US/docs/Web/Events#event_bubbling)。

冒泡事件可以在 DOM 中的任何祖先元素上被监听。你可以利用这一点，在祖先组件上添加一个事件监听器来接收其后代元素触发的冒泡事件。使用事件的 `target` 属性来基于触发事件的元素执行特定操作。

```javascript
import {LitElement, html} from 'lit';

class MyElement extends LitElement {
  static properties = {
    clicked: {},
  };

  constructor() {
    super();
    this.clicked = '';
  }

  render() {
    return html`
      <div @click="${this._clickHandler}">
        <button>Item 1</button>
        <button>Item 2</button>
        <button>Item 3</button>
      </div>
      <p>点击的项: ${this.clicked}</p>
    `;
  }

  _clickHandler(e) {
    this.clicked = e.target === e.currentTarget ? 'container' : e.target.textContent;
  }
}

customElements.define('my-element', MyElement);
```

### 异步添加事件监听器 (Asynchronously adding event listeners)

要在渲染后添加事件监听器，可以使用 `firstUpdated` 方法。这是 Lit 的生命周期回调，在组件首次更新并渲染其模板化 DOM 后触发。

`firstUpdated` 回调在组件首次调用 `render` 方法更新并渲染后触发，但在浏览器有机会绘制之前执行。

有关更多信息，请参阅生命周期文档中的 [firstUpdated](https://lit.dev/docs/components/lifecycle/#firstupdated)。

为了确保监听器在用户可以看到组件后添加，可以等待在浏览器绘制后解析的 Promise。

```javascript
async firstUpdated() {
  // 让浏览器有机会绘制
  await new Promise((r) => setTimeout(r, 0));
  this.addEventListener('click', this._handleClick);
}
```

### 理解事件监听器中的 `this` (Understanding `this` in event listeners)

在模板中使用声明式 `@` 语法添加的事件监听器会自动绑定到组件实例。

因此，可以在任意声明式事件处理程序中使用 `this` 引用组件实例：

```javascript
class MyElement extends LitElement {
  render() {
    return html`<button @click="${this._handleClick}">点击</button>`;
  }
  _handleClick(e) {
    console.log(this.prop);
  }
}
```

当使用 `addEventListener` 添加事件监听器时，需要使用箭头函数，以便 `this` 引用组件实例：

```javascript
export class MyElement extends LitElement {
  private _handleResize = () => {
    // `this` 引用组件实例
    console.log(this.isConnected);
  }

  constructor() {
    window.addEventListener('resize', this._handleResize);
  }
}
```

有关 `this` 的更多信息，请参阅 [MDN 文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)。

### 监听从重复模板触发的事件 (Listening to events fired from repeated templates)

当监听重复项上的事件时，如果事件是冒泡的，通常使用事件委托更为方便。对于不冒泡的事件，可以在重复元素上添加监听器。以下是这两种方法的示例：

```javascript
import {LitElement, html} from 'lit';

class MyElement extends LitElement {
  static properties = {
    clicked: {},
    focused: {},
  };

  constructor() {
    super();
    this.clicked = '';
    this.focused = '';
  }

  data = [1, 2, 3];

  render() {
    return html`
      <div key="container" @click=${this._clickHandler}>
        ${this.data.map(
          (i) => html`
            <button key=${i} @focus=${this._focusHandler}>项 ${i}</button>
          `
        )}
      </div>
      <p>点击的项: ${this.clicked}</p>
      <p>聚焦的项: ${this.focused}</p>
    `;
  }

  _clickHandler(e) {
    this.clicked = e.target.getAttribute('key');
  }

  _focusHandler(e) {
    this.focused = e.target.textContent;
  }
}

customElements.define('my-element', MyElement);
```

### 移除事件监听器 (Removing event listeners)

传递 `null`、`undefined` 或不传递任何值给 `@` 表达式会移除任何现有的监听器。


## 触发事件 (Dispatching events)

所有 DOM 节点都可以使用 `dispatchEvent` 方法触发事件。首先，创建一个事件实例，指定事件类型和选项。然后将该事件传递给 `dispatchEvent`，如下所示：

```javascript
const event = new Event('my-event', {bubbles: true, composed: true});
myElement.dispatchEvent(event);
```

`bubbles` 选项允许事件在 DOM 树中向上传递到触发该事件元素的祖先元素。如果希望事件能够参与事件委托，设置此标志非常重要。

`composed` 选项允许事件在元素所在的 shadow DOM 树之外传递。有关更多信息，请参阅[在 Shadow DOM 中处理事件](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM#events_in_shadow_dom)。

有关触发事件的完整描述，请参阅 MDN 上的 [EventTarget.dispatchEvent()](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent)。

### 何时触发事件 (When to dispatch an event)

事件应在响应用户交互或组件状态的异步更改时触发。通常不应在组件的属性或属性 API 改变状态时触发事件。这通常是原生 Web 平台元素的工作方式。

例如，当用户在输入框中输入值时会触发 `change` 事件，但如果代码设置了输入框的 `value` 属性，则不会触发 `change` 事件。

类似地，当用户选择菜单项时，菜单组件应触发事件，但如果设置了菜单的 `selectedItem` 属性，则不应触发事件。

这通常意味着组件应在响应其监听的其他事件时触发事件。

```javascript
import {LitElement, html} from 'lit';

class MyDispatcher extends LitElement {
  get _input() {
    return (this.___input ??= this.renderRoot?.querySelector('input') ?? null);
  }

  render() {
    return html`
      <p>姓名: <input></p>
      <p><button @click=${this._dispatchLogin}>登录</button></p>
    `;
  }

  _dispatchLogin() {
    const name = this._input.value.trim();
    if (name) {
      const options = {
        detail: {name},
        bubbles: true,
        composed: true,
      };
      this.dispatchEvent(new CustomEvent('mylogin', options));
    }
  }
}

customElements.define('my-dispatcher', MyDispatcher);
```

### 在元素更新后触发事件 (Dispatching events after an element updates)

通常，事件应在元素更新并渲染后触发。这可能是在用户交互导致的渲染状态更改时传达更改所必需的。在这种情况下，可以在更改状态后等待组件的 `updateComplete` Promise，然后再触发事件。

```javascript
import {LitElement, html} from 'lit';

class MyDispatcher extends LitElement {
  static properties = {
    open: {type: Boolean},
  };

  constructor() {
    super();
    this.open = true;
  }

  render() {
    return html`
      <p><button @click=${this._notify}>${this.open ? '关闭' : '打开'}</button></p>
      <p ?hidden=${!this.open}>内容!</p>
    `;
  }

  async _notify() {
    this.open = !this.open;
    await this.updateComplete;
    const name = this.open ? 'opened' : 'closed';
    this.dispatchEvent(new CustomEvent(name, {bubbles: true, composed: true}));
  }
}

customElements.define('my-dispatcher', MyDispatcher);
```

### 使用标准事件或自定义事件 (Using standard or custom events)

事件可以通过构造 `Event` 或 `CustomEvent` 来触发。两者都是合理的选择。当使用 `CustomEvent` 时，任何事件数据都可以通过事件的 `detail` 属性传递。当使用 `Event` 时，可以创建事件子类并向其附加自定义 API。

有关构造事件的详细信息，请参阅 MDN 上的 [Event](https://developer.mozilla.org/en-US/docs/Web/API/Event)。

#### 触发自定义事件 (Firing a custom event)

```javascript
const event = new CustomEvent('my-event', {
  detail: {
    message: '发生了重要的事情'
  }
});
this.dispatchEvent(event);
```

有关自定义事件的更多信息，请参阅 [MDN 自定义事件文档](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent)。

#### 触发标准事件 (Firing a standard event)

```javascript
class MyEvent extends Event {
  constructor(message) {
    super();
    this.type = 'my-event';
    this.message = message;
  }
}

const event = new MyEvent('发生了重要的事情');
this.dispatchEvent(event);
```


## 在 Shadow DOM 中处理事件 (Working with events in shadow DOM)

在使用 Shadow DOM 时，标准事件系统有一些重要的修改需要了解。Shadow DOM 主要是为了在 DOM 中提供一种作用域机制，用于封装有关这些 “shadow” 元素的详细信息。因此，Shadow DOM 中的事件会对外部 DOM 元素封装某些详细信息。

### 理解 `composed` 事件的触发 (Understanding composed event dispatching)

默认情况下，在 Shadow Root 内部触发的事件在该 Shadow Root 外部是不可见的。要使事件能够穿过 Shadow DOM 边界传递，必须将 `composed` 属性设置为 `true`。通常情况下，`composed` 会与 `bubbles` 配合使用，以便 DOM 树中的所有节点都可以看到该事件：

```javascript
_dispatchMyEvent() {
  let myEvent = new CustomEvent('my-event', {
    detail: { message: 'my-event 发生了.' },
    bubbles: true,
    composed: true });
  this.dispatchEvent(myEvent);
}
```

如果事件设置了 `composed` 并且是冒泡的，那么它可以被触发该事件的元素的所有祖先接收到，包括外部 Shadow Root 的祖先元素。如果事件设置了 `composed` 但不冒泡，则它只能在触发该事件的元素和包含该 Shadow Root 的宿主元素上被接收。

请注意，大多数标准用户界面事件（包括所有鼠标、触摸和键盘事件）都是既冒泡又设置了 `composed` 属性的。有关更多信息，请参阅 [MDN 上关于 `composed` 事件的文档](https://developer.mozilla.org/en-US/docs/Web/API/Event/composed)。

### 理解事件重定向 (Understanding event retargeting)

从 Shadow Root 内部触发的 `composed` 事件会被重定向（retargeted），这意味着对于 Shadow Root 宿主元素或其祖先上的任何监听器来说，它们似乎是由宿主元素本身触发的。由于 Lit 组件是渲染到 Shadow Root 中的，因此所有从 Lit 组件内部触发的 `composed` 事件都会看起来是由 Lit 组件本身触发的。事件的 `target` 属性会指向 Lit 组件。

```html
<my-element onClick="(e) => console.log(e.target)"></my-element>
```

```javascript
render() {
  return html`
    <button id="mybutton" @click="${(e) => console.log(e.target)}">
      点击我
    </button>`;
}
```

在需要确定事件来源的高级场景中，可以使用 `event.composedPath()` API。该方法返回一个包含所有被事件传递经过的节点的数组，包括那些位于 Shadow Root 中的节点。由于这会打破封装，因此在依赖可能暴露的实现细节时应格外小心。常见的用例包括确定被点击的元素是否为锚点标签（例如用于客户端路由）。

```javascript
handleMyEvent(event) {
  console.log('事件来源: ', event.composedPath()[0]);
}
```

有关更多信息，请参阅 [MDN 上的 `composedPath` 文档](https://developer.mozilla.org/en-US/docs/Web/API/Event/composedPath)。

## 在事件触发器和监听器之间进行通信 (Communicating between the event dispatcher and listener)

事件主要用于从事件触发器向事件监听器传达更改，但事件也可以用于在监听器和触发器之间传递信息。

一种方法是通过在事件中公开 API，供监听器用来自定义组件行为。例如，监听器可以在自定义事件的 `detail` 属性上设置一个属性，然后触发该事件的组件可以使用该属性来自定义行为。

另一种在触发器和监听器之间通信的方法是使用 `preventDefault()` 方法。它可以用来表示事件的标准行为不应发生。当监听器调用 `preventDefault()` 时，事件的 `defaultPrevented` 属性将变为 `true`。然后，触发该事件的组件可以使用该标志来自定义行为。

以下示例演示了这两种技术的使用：

```javascript
import {LitElement, html} from 'lit';

class MyDispatcher extends LitElement {
  static properties = {
    label: {},
    message: {},
  };

  constructor() {
    super();
    this.label = '选中我!';
    this.message = this.defaultMessage;
  }

  defaultMessage = '🙂';
  _resetMessage;

  render() {
    return html`
      <label><input type="checkbox" @click=${this._tryChange}>${this.label}</label>
      <div>${this.message}</div>
    `;
  }

  _tryChange(e) {
    const detail = {message: this.message};
    const event = new CustomEvent('checked', {
      detail,
      bubbles: true,
      composed: true,
      cancelable: true,
    });
    this.dispatchEvent(event);
    if (event.defaultPrevented) {
      e.preventDefault();
    }
    this.message = detail.message;
  }

  updated() {
    clearTimeout(this._resetMessage);
    this._resetMessage = setTimeout(
      () => (this.message = this.defaultMessage),
      1000
    );
  }
}

customElements.define('my-dispatcher', MyDispatcher);
```