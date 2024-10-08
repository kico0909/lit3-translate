
# 内置指令

指令是用于扩展 Lit 的函数，可以自定义表达式的渲染方式。Lit 包含了许多内置指令，帮助处理各种渲染需求：

| 指令                | 说明                                                       |
|---------------------|------------------------------------------------------------|
| **Styling**         |                                                            |
| `classMap`          | 根据对象为元素分配类名列表。                                |
| `styleMap`          | 根据对象为元素设置样式属性列表。                            |
| **Loops and Conditionals** |                                                    |
| `when`              | 根据条件渲染两个模板中的一个。                              |
| `choose`            | 根据键值渲染多个模板中的一个。                              |
| `map`               | 使用函数转换可迭代对象。                                    |
| `repeat`            | 将可迭代对象的值渲染到 DOM 中，并可选使用键值来实现数据差异检测和 DOM 稳定性。 |
| `join`              | 使用连接符值将可迭代对象的值进行连接。                      |
| `range`             | 创建一个数值序列的可迭代对象，适用于遍历特定次数。          |
| `ifDefined`         | 如果值已定义则设置属性，否则移除该属性。                    |
| **Caching and change detection** |                                             |
| `cache`             | 在更改模板时缓存渲染的 DOM 而不是丢弃 DOM。                 |
| `keyed`             | 将渲染值与唯一键关联，如果键发生更改，则强制重新渲染 DOM。   |
| `guard`             | 仅在依赖项更改时重新计算模板。                              |
| `live`              | 如果属性或属性值与实际 DOM 值不同，则设置属性或属性。        |
| **Referencing rendered DOM** |                                                  |
| `ref`               | 获取模板中渲染的元素的引用。                                |
| **Rendering special values** |                                                  |
| `templateContent`   | 渲染 `<template>` 元素的内容。                              |
| `unsafeHTML`        | 将字符串渲染为 HTML 而不是文本。                            |
| `unsafeSVG`         | 将字符串渲染为 SVG 而不是文本。                             |
| **Asynchronous rendering** |                                                   |
| `until`             | 在一个或多个 promise 解析之前渲染占位符内容。               |
| `asyncAppend`       | 在 AsyncIterable 中的值生成时将其追加到 DOM 中。             |
| `asyncReplace`      | 在 AsyncIterable 中的值生成时将其渲染到 DOM 中。             |

这些被称为“内置”指令是因为它们是 Lit 包的一部分。但每个指令都是一个单独的模块，因此应用程序只会打包导入的指令。

您还可以创建自己的指令。有关详细信息，请参阅[自定义指令](#)。

## 样式指令

### classMap
根据对象为元素分配类名列表。

**导入**  
`import {classMap} from 'lit/directives/class-map.js';`

**用法**  
`classMap(classInfo: {[name: string]: string | boolean | number})`

**使用位置**  
只能在 `class` 属性表达式中使用（必须是 `class` 属性中的唯一表达式）。

`classMap` 指令使用 `element.classList` API 来基于用户传递的对象有效地为元素添加和移除类名。对象中的每个键被视为类名，如果与该键关联的值为真，则该类名将添加到元素中。在后续渲染中，任何以前设置的类名如果为假或不再出现在对象中，则会被移除。

```javascript
import {LitElement, html} from 'lit';
import {classMap} from 'lit/directives/class-map.js';

class MyElement extends LitElement {
  static properties = {
    enabled: {type: Boolean},
  };

  constructor() {
    super();
    this.enabled = false;
  }

  render() {
    const classes = { enabled: this.enabled, hidden: false };
    return html`<div class=${classMap(classes)}>Classy text</div>`;
  }
}
customElements.define('my-element', MyElement);
```

`classMap` 必须是 `class` 属性中的唯一表达式，但它可以与静态值组合使用：

```javascript
html`<div class="my-widget ${classMap(dynamicClasses)}">Static and dynamic</div>`;
```

### styleMap
根据对象为元素设置样式属性列表。

**导入**  
`import {styleMap} from 'lit/directives/style-map.js';`

**用法**  
`styleMap(styleInfo: {[name: string]: string | undefined | null})`

**使用位置**  
只能在 `style` 属性表达式中使用（必须是 `style` 属性中的唯一表达式）。

`styleMap` 指令使用 `element.style` API 来基于用户传递的对象有效地为元素添加和移除内联样式。对象中的每个键被视为样式属性名，值被视为该属性的值。在后续渲染中，任何以前设置的样式属性如果为 `undefined` 或 `null` 则会被移除（设置为 `null`）。

```javascript
import {LitElement, html} from 'lit';
import {styleMap} from 'lit/directives/style-map.js';

class MyElement extends LitElement {
  static properties = {
    enabled: {type: Boolean},
  };

  constructor() {
    super();
    this.enabled = false;
  }

  render() {
    const styles = { backgroundColor: this.enabled ? 'blue' : 'gray', color: 'white' };
    return html`<p style=${styleMap(styles)}>Hello style!</p>`;
  }
}
customElements.define('my-element', MyElement);
```

对于包含连字符的 CSS 属性，可以使用驼峰命名法或将属性名称放在引号中。例如，可以将 CSS 属性 `font-family` 写为 `fontFamily` 或 `'font-family'`：

```javascript
{ fontFamily: 'roboto' }
{ 'font-family': 'roboto' }
```

对于 CSS 自定义属性（例如 `--custom-color`），请将整个属性名放在引号中：

```javascript
{ '--custom-color': 'steelblue' }
```

`styleMap` 必须是 `style` 属性中的唯一表达式，但它可以与静态值组合使用：

```javascript
html`<p style="color: white; ${styleMap(moreStyles)}">More styles!</p>`;
```


## 循环和条件语句

### `when`
根据条件渲染两个模板之一。

**导入**
```javascript
import { when } from 'lit/directives/when.js';
```

**签名**
```typescript
when<T, F>(
  condition: boolean,
  trueCase: () => T,
  falseCase?: () => F
)
```

**可用位置**
任意位置

当 `condition` 为 `true` 时，返回调用 `trueCase()` 的结果；否则，如果定义了 `falseCase`，返回调用 `falseCase()` 的结果。

这是一个便捷的封装，使得写内联条件表达式时无需使用 `else`。

```javascript
class MyElement extends LitElement {
  render() {
    return html`
      ${when(this.user, () => html`用户: ${this.user.username}`, () => html`请登录...`)}
    `;
  }
}
```

### `choose`
从多个模板中选择一个来渲染，基于匹配给定值的情况。

**导入**
```javascript
import { choose } from 'lit/directives/choose.js';
```

**签名**
```typescript
choose<T, V>(
  value: T,
  cases: Array<[T, () => V]>,
  defaultCase?: () => V
)
```

**可用位置**
任意位置

`cases` 是结构为 `[caseValue, func]` 的数组。`value` 通过严格相等性检查来匹配 `caseValue`。找到的第一个匹配项将被选中。`caseValue` 可以是任意类型，包括原始类型、对象和符号。

这类似于 `switch` 语句，但作为表达式并且没有贯穿效应。

```javascript
class MyElement extends LitElement {
  render() {
    return html`
      ${choose(this.section, [
        ['home', () => html`<h1>主页</h1>`],
        ['about', () => html`<h1>关于</h1>`]
      ],
      () => html`<h1>错误</h1>`)}
    `;
  }
}
```

### `map`
返回一个包含对每个值调用 `f(value)` 结果的可迭代对象。

**导入**
```javascript
import { map } from 'lit/directives/map.js';
```

**签名**
```typescript
map<T>(
  items: Iterable<T> | undefined,
  f: (value: T, index: number) => unknown
)
```

**可用位置**
任意位置

`map()` 是 `for/of` 循环的简单封装，使得在表达式中更容易处理可迭代对象。`map()` 总是在原位置更新任何创建的 DOM——它不执行任何 diff 或 DOM 移动。如果需要这些功能，请参见 `repeat`。如果不需要 diff 和 DOM 稳定性，优先使用 `map()`，因为它比 `repeat()` 更小更快。

```javascript
class MyElement extends LitElement {
  render() {
    return html`
      <ul>
        ${map(items, (i) => html`<li>${i}</li>`)}
      </ul>
    `;
  }
}
```

### `repeat`
根据可迭代对象将值渲染到 DOM 中，并可选通过键启用数据 diff 和 DOM 稳定性。

**导入**
```javascript
import { repeat } from 'lit/directives/repeat.js';
```

**签名**
```typescript
repeat(items: Iterable<T>, keyfn: KeyFn<T>, template: ItemTemplate<T>)
repeat(items: Iterable<T>, template: ItemTemplate<T>)
```

**可用位置**
子表达式

重复从可迭代对象生成的一系列值（通常为 `TemplateResults`），并在可迭代对象变化时高效更新这些值。当提供了 `keyFn` 时，通过在更新时移动生成的 DOM 来保持键与 DOM 的关联，这是最有效的使用方式，因为它仅在插入和删除时执行最少的无效操作。

```javascript
class MyElement extends LitElement {
  static properties = {
    items: {},
  };

  constructor() {
    super();
    this.items = [];
  }

  render() {
    return html`
      <ul>
        ${repeat(this.items, (item) => item.id, (item, index) => html`
          <li>${index}: ${item.name}</li>`)}
      </ul>
    `;
  }
}
customElements.define('my-element', MyElement);
```

### `join`
返回包含 `items` 中的值与连接符值交错的可迭代对象。

**导入**
```javascript
import { join } from 'lit/directives/join.js';
```

**签名**
```typescript
join<I, J>(
  items: Iterable<I> | undefined,
  joiner: J
): Iterable<I | J>;

join<I, J>(
  items: Iterable<I> | undefined,
  joiner: (index: number) => J
): Iterable<I | J>;
```

**可用位置**
任意位置

```javascript
class MyElement extends LitElement {
  render() {
    return html`
      ${join(
        map(menuItems, (i) => html`<a href=${i.href}>${i.label}</a>`),
        html`<span class="separator">|</span>`
      )}
    `;
  }
}
```

### `range`
返回从 `start` 到 `end`（不包括 `end`）的整数可迭代对象，以 `step` 为增量。

**导入**
```javascript
import { range } from 'lit/directives/range.js';
```

**签名**
```typescript
range(end: number): Iterable<number>;

range(
  start: number,
  end: number,
  step?: number
): Iterable<number>;
```

**可用位置**
任意位置

```javascript
class MyElement extends LitElement {
  render() {
    return html`
      ${map(range(8), (i) => html`${i + 1}`)}
    `;
  }
}
```

### `ifDefined`
如果值已定义则设置属性，如果值未定义则移除属性。

**导入**
```javascript
import { ifDefined } from 'lit/directives/if-defined.js';
```

**签名**
```typescript
ifDefined(value: unknown)
```

**可用位置**
属性表达式

对于 `AttributeParts`，如果值已定义则设置属性，如果值为未定义（`undefined` 或 `null`），则移除属性。对于其他部分类型，此指令无效。

```javascript
class MyElement extends LitElement {
  static properties = {
    filename: {},
    size: {},
  };

  constructor() {
    super();
    this.filename = undefined;
    this.size = undefined;
  }

  render() {
    // 如果 `size` 或 `filename` 未定义，则不会渲染 `src` 属性
    return html`<img src="/images/${ifDefined(this.size)}/${ifDefined(this.filename)}">`;
  }
}
customElements.define('my-element', MyElement);
```


## 缓存与变更检测

### 缓存（cache）

缓存切换模板时渲染的 DOM，而不是丢弃它。可以使用该指令在频繁切换大型模板时优化渲染性能。

**导入：**
```javascript
import {cache} from 'lit/directives/cache.js';
```

**签名：**
```javascript
cache(value: TemplateResult|unknown)
```

**可用位置：** 子表达式

当传递给 `cache` 的值在一个或多个 `TemplateResults` 之间更改时，给定模板的渲染 DOM 节点在不使用时被缓存。当模板更改时，该指令在切换到新值之前缓存当前的 DOM 节点，并在切换回先前渲染的值时，从缓存中恢复它们，而不是重新创建 DOM 节点。

**示例**
```javascript
const detailView = (data) => html`<div>...</div>`;
const summaryView = (data) => html`<div>...</div>`;

class MyElement extends LitElement {
  static properties = {
    data: {},
  };

  constructor() {
    super();
    this.data = {showDetails: true, /*...*/ };
  }

  render() {
    return html`${cache(this.data.showDetails
      ? detailView(this.data)
      : summaryView(this.data)
    )}`;
  }
}
customElements.define('my-element', MyElement);
```

当 Lit 重新渲染模板时，它只更新修改的部分：它不会创建或删除更多的 DOM 节点。但当你从一个模板切换到另一个模板时，Lit 会移除旧的 DOM 并渲染新的 DOM 树。

`cache` 指令会为给定的表达式和输入模板缓存生成的 DOM。在上面的例子中，它缓存了 `summaryView` 和 `detailView` 模板的 DOM。当你在两个视图之间切换时，Lit 会切换到缓存的新视图，并使用最新数据更新它。这在这些视图频繁切换时，可以提升渲染性能。

### 键控（keyed）

与唯一键关联的可渲染值。当键更改时，之前的 DOM 会被移除并在渲染下一个值之前被处理，即使值（如模板）相同。

**导入：**
```javascript
import {keyed} from 'lit/directives/keyed.js';
```

**签名：**
```javascript
keyed(key: unknown, value: unknown)
```

**可用位置：** 任意表达式

当渲染有状态的元素时，如果需要确保某些关键数据发生变化时清除元素的所有状态，`keyed` 非常有用。它本质上是选择退出 Lit 的默认 DOM 重用策略。

**示例**
```javascript
class MyElement extends LitElement {
  static properties = {
    userId: {},
  };

  constructor() {
    super();
    this.userId = '';
  }

  render() {
    return html`
      <div>
        ${keyed(this.userId, html`<user-card .userId=${this.userId}></user-card>`)}
      </div>`;
  }
}
customElements.define('my-element', MyElement);
```

### 守护（guard）

仅在其依赖项之一更改时重新评估模板，以通过防止不必要的工作来优化渲染性能。

**导入：**
```javascript
import {guard} from 'lit/directives/guard.js';
```

**签名：**
```javascript
guard(dependencies: unknown[], valueFn: () => unknown)
```

**可用位置：** 任意表达式

根据 `valueFn` 返回的值进行渲染，并且仅在依赖项之一更改时重新评估 `valueFn`。

**示例**
```javascript
class MyElement extends LitElement {
  static properties = {
    value: {},
  };

  constructor() {
    super();
    this.value = '';
  }

  render() {
    return html`
      <div>
        ${guard([this.value], () => calculateSHA(this.value))}
      </div>`;
  }
}
customElements.define('my-element', MyElement);
```

在此情况下，只有 `value` 属性更改时才会运行耗时的 `calculateSHA` 函数。

### 实时（live）

在属性或值与实时 DOM 值不同时更新它，而不是上次渲染的值。

**导入：**
```javascript
import {live} from 'lit/directives/live.js';
```

**签名：**
```javascript
live(value: unknown)
```

**可用位置：** 属性或属性表达式

当确定是否更新值时，会将表达式的值与实时 DOM 值进行比较，而不是 Lit 的默认行为即检查上次设置的值。

**示例**
```javascript
class MyElement extends LitElement {
  static properties = {
    data: {},
  };

  constructor() {
    super();
    this.data = {value: 'test'};
  }

  render() {
    return html`<input .value=${live(this.data.value)}>`;
  }
}
customElements.define('my-element', MyElement);
```

`live()` 会对实际的 DOM 值执行严格相等性检查，如果新值与实际 DOM 中的值相等，则不会执行任何操作。这意味着当表达式可能导致类型转换时，不应使用 `live()`。如果在属性表达式中使用 `live()`，请确保仅传递字符串，否则该表达式会在每次渲染时更新。

可以在 playground 中查看更多 `live()` 的用法。



## 渲染特殊值

### `templateContent`
> 用于渲染 `<template>` 元素的内容

**引入**
```javascript
import { templateContent } from 'lit/directives/template-content.js';
```

**签名**
```typescript
templateContent(templateElement: HTMLTemplateElement)
```

**使用位置**
子表达式

Lit 模板使用 JavaScript 编码，因此可以嵌入使其动态化的 JavaScript 表达式。如果你有一个静态的 HTML `<template>`，并希望将其包含在 Lit 模板中，可以使用 `templateContent` 指令克隆模板内容并将其包含在 Lit 模板中。只要模板元素的引用在渲染之间不变，后续渲染将不会进行操作。

> 注意：模板内容应由开发人员控制，不得使用不受信任的字符串创建。未经信任的模板可能导致跨站脚本攻击 (XSS) 漏洞。

**示例**
```javascript
const templateEl = document.querySelector('template#myContent');

class MyElement extends LitElement {
  render() {
    return html`
      Here's some content from a template element:
      ${templateContent(templateEl)}`;
  }
}
customElements.define('my-element', MyElement);
```

在 playground 中查看更多关于 `templateContent` 的示例。

---

### `unsafeHTML`
> 用于将字符串渲染为 HTML 而不是文本

**引入**
```javascript
import { unsafeHTML } from 'lit/directives/unsafe-html.js';
```

**签名**
```typescript
unsafeHTML(value: string | typeof nothing | typeof noChange)
```

**使用位置**
子表达式

Lit 的模板语法的一个关键特性是，只有源自模板文字的字符串才会被解析为 HTML。由于模板文字只能在可信赖的脚本文件中编写，这可以作为防止 XSS 攻击注入不受信任的 HTML 的自然保障。然而，有时可能需要在 Lit 模板中渲染不来源于脚本文件的 HTML，例如从数据库获取的可信 HTML 内容。`unsafeHTML` 指令会将此类字符串解析为 HTML 并渲染到 Lit 模板中。

> 注意：传递给 `unsafeHTML` 的字符串必须由开发人员控制，且不得包含不受信任的内容。例如，查询字符串参数和用户输入的值。

> 渲染未经信任的内容可能导致 XSS、CSS 注入、数据泄露等安全漏洞。`unsafeHTML` 使用 `innerHTML` 解析 HTML 字符串，因此其安全隐患与 `innerHTML` 的相同，详细信息请参阅 MDN。

**示例**
```javascript
const markup = '<h3>Some HTML to render.</h3>';

class MyElement extends LitElement {
  render() {
    return html`
      Look out, potentially unsafe HTML ahead:
      ${unsafeHTML(markup)}
    `;
  }
}
customElements.define('my-element', MyElement);
```

在 playground 中查看更多关于 `unsafeHTML` 的示例。

### `unsafeSVG`
>用于将字符串渲染为 SVG 而不是文本

**引入**
```javascript
import { unsafeSVG } from 'lit/directives/unsafe-svg.js';
```

**签名**
```typescript
unsafeSVG(value: string | typeof nothing | typeof noChange)
```

**使用位置**
子表达式

与 `unsafeHTML` 类似，可能存在需要在 Lit 模板中渲染不来源于脚本文件的 SVG 内容的情况，例如从数据库中获取的可信 SVG 内容。`unsafeSVG` 指令会将此类字符串解析为 SVG 并渲染到 Lit 模板中。

> 注意：传递给 `unsafeSVG` 的字符串必须由开发人员控制，且不得包含不受信任的内容。例如，查询字符串参数和用户输入的值。未经信任的内容渲染可能导致 XSS 漏洞。

**示例**
```javascript
const svg = '<circle cx="50" cy="50" r="40" fill="red" />';

class MyElement extends LitElement {
  render() {
    return html`
      Look out, potentially unsafe SVG ahead:
      <svg width="40" height="40" viewBox="0 0 100 100"
        xmlns="http://www.w3.org/2000/svg" version="1.1">
        ${unsafeSVG(svg)}
      </svg> `;
  }
}
customElements.define('my-element', MyElement);
```

在 playground 中查看更多关于 `unsafeSVG` 的示例。


## 引用渲染的 DOM

### `ref`
> 用于获取渲染到 DOM 中的元素的引用

**引入**
```javascript
import { ref } from 'lit/directives/ref.js';
```

**签名**
```typescript
ref(refOrCallback: RefOrCallback)
```

**使用位置**
元素表达式

虽然大多数 DOM 操作可以使用 Lit 的模板声明式完成，但在某些高级场景中，可能需要获取模板中渲染的元素引用并以命令式方式进行操作。例如，聚焦表单控件或在容器元素上调用命令式 DOM 操作库。

`ref` 指令放置在模板中的元素上时，可以在渲染后检索该元素的引用。可以通过传递 `Ref` 对象或回调来获取引用。

`Ref` 对象作为元素引用的容器，可以使用 `ref` 模块中的 `createRef` 辅助方法创建。渲染后，`Ref` 的 `value` 属性将被设置为元素，可以在 `updated` 生命周期中访问。

**示例**
```javascript
class MyElement extends LitElement {
  inputRef = createRef();

  render() {
    return html`<input ${ref(this.inputRef)}>`;
  }

  firstUpdated() {
    const input = this.inputRef.value!;
    input.focus();
  }
}
customElements.define('my-element', MyElement);
```

还可以将 `ref` 回调传递给 `ref` 指令。回调将在每次引用的元素更改时被调用。如果 `ref` 回调在随后的渲染中被渲染到不同的元素位置或被移除，将首先用 `undefined` 调用，随后再用新元素（如果有）调用。注意，在 `LitElement` 中，回调将自动绑定到宿主元素。

**示例**
```javascript
class MyElement extends LitElement {
  render() {
    return html`<input ${ref(this.inputChanged)}>`;
  }

  inputChanged(input) {
    input?.focus();
  }
}
customElements.define('my-element', MyElement);
```
在 playground 中查看更多关于 `ref` 的示例。


## 异步渲染

### `until` 指令
在一个或多个 Promise 解析之前渲染占位符内容。

**导入**：
```javascript
import {until} from 'lit/directives/until.js';
```

**函数签名**：
```javascript
until(...values: unknown[])
```

**可用位置**：
任意表达式位置

`until` 接收一系列值（包括 Promise），按照优先级渲染值，第一个参数的优先级最高，最后一个参数的优先级最低。如果某个值是 Promise，则在其解析之前会渲染优先级更低的值。

可以使用值的优先级来创建异步数据的占位符内容。例如，第一个参数可以是包含待定内容的 Promise，而第二个（优先级较低的）参数可以是一个非 Promise 的加载指示器模板。加载指示器会立即渲染，而主要内容会在 Promise 解析时渲染。

```javascript
class MyElement extends LitElement {
  static properties = {
    content: {state: true},
  };

  constructor() {
    super();
    this.content = fetch('./content.txt').then(r => r.text());
  }

  render() {
    return html`${until(this.content, html`<span>Loading...</span>`)}`;
  }
}
customElements.define('my-element', MyElement);
```
了解更多关于 `until` 指令的内容，请在 Playground 中探索。

### `asyncAppend` 指令
按顺序将来自 `AsyncIterable` 的值追加到 DOM 中。

**导入**：
```javascript
import {asyncAppend} from 'lit/directives/async-append.js';
```

**函数签名**：
```javascript
asyncAppend(
  iterable: AsyncIterable<I>,
  mapper?: (item: I, index?: number) => unknown
)
```

**可用位置**：
子表达式

`asyncAppend` 渲染 `async iterable` 的值，将每个新值追加到前一个值之后。请注意，`async generators` 也实现了 `async iterable` 协议，因此也可以被 `asyncAppend` 消费。

```javascript
async function *tossCoins(count) {
  for (let i=0; i<count; i++) {
    yield Math.random() > 0.5 ? 'Heads' : 'Tails';
    await new Promise((r) => setTimeout(r, 1000));
  }
}

class MyElement extends LitElement {
  static properties = {
    tosses: {state: true},
  };

  constructor() {
    super();
    this.tosses = tossCoins(10);
  }

  render() {
    return html`
      <ul>${asyncAppend(this.tosses, (v) => html`<li>${v}</li>`)}</ul>`;
  }
}
customElements.define('my-element', MyElement);
```
了解更多关于 `asyncAppend` 指令的内容，请在 Playground 中探索。

### `asyncReplace` 指令
在 `AsyncIterable` 产生值时将最新值渲染到 DOM 中。

**导入**：
```javascript
import {asyncReplace} from 'lit/directives/async-replace.js';
```

**函数签名**：
```javascript
asyncReplace(
  iterable: AsyncIterable<I>,
  mapper?: (item: I, index?: number) => unknown
)
```

**可用位置**：
任意表达式位置

与 `asyncAppend` 类似，`asyncReplace` 渲染 `async iterable` 的值，每个新值替换之前的值。

```javascript
async function *countDown(count) {
  while (count > 0) {
    yield count--;
    await new Promise((r) => setTimeout(r, 1000));
  }
}

class MyElement extends LitElement {
  static properties = {
    timer: {state: true},
  };

  constructor() {
    super();
    this.timer = countDown(10);
  }

  render() {
    return html`Timer: <span>${asyncReplace(this.timer)}</span>.`;
  }
}
customElements.define('my-element', MyElement);
```
了解更多关于 `asyncReplace` 指令的内容，请在 Playground 中探索。

