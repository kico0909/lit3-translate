
# 表达式（Expressions）

Lit 模板中可以包含称为表达式的动态值。表达式可以是任意 JavaScript 表达式。表达式在模板被评估时进行计算，并且表达式的结果将在模板渲染时被包含。在 Lit 组件中，这意味着每当 `render` 方法被调用时，表达式都会被重新计算。

表达式只能放置在模板的特定位置，表达式的解析方式取决于它出现的位置。出现在元素标签内的表达式会影响该元素；而出现在元素内容中的表达式（即子节点）则会渲染为该元素的子节点或文本。

表达式的有效值因表达式出现的位置不同而有所差异。通常，所有表达式都接受类似字符串和数字等原始值（primitive values），而有些表达式还支持其他类型的值。此外，所有表达式都可以接受指令（directives），这些指令是用于自定义表达式处理和渲染的特殊函数。有关自定义指令的更多信息，请参见[自定义指令（Custom directives）](#)。

以下是一个快速参考，后面将详细介绍每种表达式类型：


| 类型                 | 示例                                                         |
| -------------------- | ------------------------------------------------------------ |
| **子节点**           | `html`<br/>`<h1>Hello ${name}</h1><ul>${listItems}</ul>`      |
| **属性**             | `html`<br>`<div class=${highlightClass}></div>`              |
| **布尔属性**         | `html`<br>`<div hidden=${!show}></div>`                     |
| **属性（属性绑定）** | `html`<br>`<input .value=${value}>`                          |
| **事件监听器**       | `html`<br>`<button @click=${this._clickHandler}>Go</button>` |
| **元素指令**         | `html`<br>`<input ${ref(inputRef)}>`                         |

以下是一个包含各种表达式类型的基本示例：

```js
render() {
  return html`
    <p>
      ${this.greeting}
      <button @click=${() => (this.show = !this.show)}>Todos</button>
    </p>
    ${
      this.show
        ? html`
          <ul>${this.todos.map((i) => html`<li>${i}</li>`)}</ul>
        `
        : ''
    }`;
}
```

以下部分将更详细地介绍每种表达式类型。有关模板结构的更多信息，请参见[格式良好的 HTML](#)和[有效表达式位置](#)。

## 子节点表达式（Child expressions）

位于元素开始标签和结束标签之间的表达式可以向元素中添加子节点。例如：

```js
html`<p>Hello, ${name}</p>`
```

或：

```js
html`<main>${bodyText}</main>`
```

子节点位置的表达式可以接受多种类型的值：

- 原始值（字符串、数字和布尔值）
- 使用 `html` 函数（或如果表达式位于 `<svg>` 元素内部则使用 `svg` 函数）创建的 `TemplateResult` 对象
- DOM 节点
- 特殊值 `nothing` 和 `noChange`
- 任意支持类型的数组或可迭代对象

### 原始值（Primitive values）

Lit 可以渲染几乎所有的原始值，并在将它们插入到文本内容时将其转换为字符串。

例如：

- 数字值（如 5）将渲染为字符串 `'5'`，BigInt 类型也是类似处理。
- 布尔值 `true` 将渲染为 `'true'`，而 `false` 将渲染为 `'false'`。但是，这样渲染布尔值的情况很少见。布尔值通常用于条件判断来渲染其他合适的值。有关条件判断的更多信息，请参见[条件判断](#)。
- 空字符串 `''`、`null` 和 `undefined` 会被特殊处理，并渲染为空（不渲染任何内容）。有关移除子节点内容的更多信息，请参见[移除子节点内容](#)。

`Symbol` 类型的值无法转换为字符串，当将 `Symbol` 放置在子节点表达式中时会抛出错误。

### 特殊值（Sentinel values）

Lit 提供了几个可以在子节点表达式中使用的特殊值。

- `noChange`：不更改表达式的现有值。通常用于自定义指令。请参见[指示无更改（Signaling no change）](#)。
- `nothing`：不渲染任何内容。请参见[移除子节点内容](#)。

### 模板（Templates）

由于子节点位置的表达式可以返回 `TemplateResult`，因此您可以嵌套和组合模板：

```js
const nav = html`<nav>...</nav>`;
const page = html`
  ${nav}
  <main>...</main>
`;
```

这意味着您可以使用纯 JavaScript 来创建条件模板、重复模板等：

```js
html`
  ${this.user.isLoggedIn
    ? html`Welcome ${this.user.name}`
    : html`Please log in`
  }
`;
```

有关条件模板的更多信息，请参见[条件模板](#)。  
有关使用 JavaScript 创建重复模板的更多信息，请参见[列表](#)。

### DOM 节点（DOM nodes）

任意 DOM 节点都可以传递给子节点表达式。通常应通过指定使用 `html` 函数的模板来渲染 DOM 节点，但在需要时也可以直接渲染 DOM 节点。此时，该节点会被附加到 DOM 树中，并从当前父级节点中移除：

```js
const div = document.createElement('div');
const page = html`
  ${div}
  <p>This is some text</p>
`;
```


### 数组或可迭代对象

表达式还可以返回任意支持类型的数组或可迭代对象，并且这些类型可以任意组合使用。你可以利用这个特性结合标准的 JavaScript 方法（如 `Array` 的 `map` 方法）来创建重复模板和列表。有关更多示例，请参阅 [列表](#) 章节。

### 移除子内容

值为 `null`、`undefined`、空字符串 `''` 和 Lit 的 `nothing` 哨兵值时，会移除任何先前渲染的内容，并不会渲染任何节点。

根据条件设置或移除子内容是一种常见操作。有关更多信息，请参阅 [条件渲染空值](#)。

在包含插槽的 Shadow DOM 元素中，当表达式是某个元素的子节点时，渲染空节点可以确保显示插槽的回退内容。请参阅 [回退内容](#) 以获取更多信息。

## 属性表达式

除了可以使用表达式添加子节点外，还可以使用它们设置元素的属性和属性值。

默认情况下，出现在属性值中的表达式会设置该属性：

```js
html`<div class=${this.textClass}>时尚的文本。</div>`;
```

由于属性值始终是字符串，因此表达式应返回可以转换为字符串的值。

如果表达式组成了整个属性值，可以省略引号。如果表达式只是属性值的一部分，则需要引用整个值：

```js
html`<img src="/images/${this.image}">`;
```

注意，一些基本值在属性中会有特殊的处理方式。例如，布尔值会被转换为字符串，因此 `false` 会被渲染为 `'false'`。`undefined` 和 `null` 在属性中会被渲染为空字符串。

### 布尔属性

要设置布尔属性，可以在属性名前使用 `?` 前缀。当表达式求值为真值时，添加该属性；当表达式求值为假值时，移除该属性：

```js
html`<div ?hidden=${!this.showAdditional}>此文本可能会被隐藏。</div>`;
```

### 移除属性

有时你希望仅在特定条件下设置某个属性，而在其他情况下移除该属性。对于常见的“布尔属性”（如 `disabled` 和 `hidden`），你希望将属性设置为空字符串以表示真值，并在其他情况下移除该属性，可以使用布尔属性语法。然而，有时你可能需要不同的条件来决定是否添加或移除某个属性。

例如，考虑以下代码：

```js
html`<img src="/images/${this.imagePath}/${this.imageFile}">`;
```

如果 `this.imagePath` 或 `this.imageFile` 没有定义，则不应该设置 `src` 属性，以避免无效的网络请求。

Lit 的 `nothing` 哨兵值可以解决这个问题：当属性值中的任何表达式求值为 `nothing` 时，将移除该属性。

```js
html`<img src="/images/${this.imagePath ?? nothing}/${this.imageFile ?? nothing}">`;
```

在这个例子中，`this.imagePath` 和 `this.imageFile` 属性都必须被定义，`src` 属性才会被设置。`??` 空值合并运算符在左侧值为 `null` 或 `undefined` 时会返回右侧值。

Lit 还提供了 `ifDefined` 指令，它的作用与 `value ?? nothing` 类似。

```js
html`<img src="/images/${ifDefined(this.imagePath)}/${ifDefined(this.imageFile)}">`;
```

如果希望属性值为假值时（例如 `false` 或空字符串 `''`）移除该属性，可以使用如下方式。例如，考虑一个元素的 `this.ariaLabel` 默认值为空字符串 `''`：

```js
html`<button aria-label="${this.ariaLabel || nothing}"></button>`;
```

在这个例子中，`aria-label` 属性仅在 `this.ariaLabel` 不为空字符串时渲染。

根据条件设置或移除属性是一种常见的操作。有关更多信息，请参阅 [条件渲染空值](#)。

## 属性表达式

你可以使用 `.` 前缀和属性名在元素上设置 JavaScript 属性：

```js
html`<input .value=${this.itemCount}>`;
```

上述代码的行为与直接设置 `input` 元素的 `value` 属性相同，例如：

```js
inputEl.value = this.itemCount;
```

你可以使用属性表达式语法将复杂数据传递到子组件中。例如，如果你有一个 `my-list` 组件，并且它有一个 `listItems` 属性，你可以传递一个对象数组：

```js
html`<my-list .listItems=${this.items}></my-list>`;
```

注意，本例中的属性名 `listItems` 是驼峰式写法。虽然 HTML 属性对大小写不敏感，但 Lit 在处理模板时会保留属性名的大小写。

有关组件属性的更多信息，请参阅 [响应式属性](#)。


## 事件监听表达式

Lit 模板中可以包含声明式的事件监听器。使用 `@` 前缀加上事件名称。表达式应当解析为一个事件监听器。

```html
html`<button @click=\${this.clickHandler}>Click Me!</button>`;
```

这类似于在按钮元素上调用 `addEventListener('click', this.clickHandler)`。

事件监听器可以是一个普通函数，也可以是一个带有 `handleEvent` 方法的对象——与标准 `addEventListener` 方法的监听器参数相同。

在 Lit 组件中，事件监听器会自动绑定到组件，因此可以在处理程序中使用 `this` 值来引用组件实例。

```javascript
clickHandler() {
  this.clickCount++;
}
```

有关组件事件的更多信息，请参阅[事件](#events)。

## 元素表达式

您还可以添加访问元素实例的表达式，而不是单个属性或元素上的属性：

```html
html`<div \${myDirective()}></div>`
```

元素表达式只能与指令一起使用。其他类型的值在元素表达式中将被忽略。

一个内置的可以用于元素表达式的指令是 `ref` 指令。它提供了对渲染元素的引用。

```html
html`<button \${ref(this.myRef)}></button>`;
```

有关 `ref` 的更多信息，请参阅[ref](#ref)。

## 符合HTML规范

Lit 模板必须符合 HTML 规范。模板在插入任何值之前会由浏览器的内置 HTML 解析器进行解析。请遵循以下规则以确保模板的正确性：

- 当所有表达式被替换为空值时，模板必须是符合 HTML 规范的。
- 模板可以包含多个顶级元素和文本。
- 模板不应包含未关闭的元素——否则它们将被 HTML 解析器自动关闭。

例如：

```html
// HTML 解析器在 "Some text" 后自动关闭这个 div
const template1 = html`<div class="broken-div">Some text`;
// 当合并时，"more text" 不会出现在 .broken-div 中
const template2 = html`\${template1} more text. </div>`;
```

由于浏览器的内置解析器非常宽松，大多数模板格式不正确的情况在运行时是无法检测到的，因此您不会看到警告——只会得到不符合预期的模板。我们建议在开发过程中使用 lint 工具和 IDE 插件来查找模板中的问题。

## 有效表达式位置

表达式只能出现在 HTML 中可放置属性值和子元素的位置。

```html
<!-- 属性值 -->
<div label=\${label}></div>
<button ?disabled=\${isDisabled}>Click me!</button>
<input .value=\${currentValue}>
<button @click=\${this.handleClick()}>
```

```html
<!-- 子内容 -->
<div>\${textContent}</div>
```

元素表达式可以出现在标签名称后的打开标签内：

```html
<div \${ref(elementReference)}></div>
```

### 无效位置

表达式通常不应出现在以下位置：

- 出现在标签或属性名称所在的位置。Lit 不支持在这些位置动态更改值，并且在开发模式中会报错。

```html
<!-- 错误 -->
<\${tagName}></\${tagName}>

<!-- 错误 -->
<div \${attrName}=true></div>
```

- 出现在 `<template>` 元素内容中（但允许出现在 `template` 元素本身的属性表达式中）。Lit 不会递归处理 `template` 内容来动态更新表达式，并且在开发模式中会报错。

```html
<!-- 错误 -->
<template>\${content}</template>

<!-- 正确 -->
<template id="\${attrValue}">static content ok</template>
```

- 出现在 `<textarea>` 元素内容中（但允许出现在 `textarea` 元素本身的属性表达式中）。请注意，Lit 可以将内容渲染到 `<textarea>` 中，但编辑 `<textarea>` 会破坏 Lit 用于动态更新的 DOM 引用，并且 Lit 会在开发模式中发出警告。请改为绑定 `<textarea>` 的 `.value` 属性。

```html
<!-- 注意 -->
<textarea>\${content}</textarea>

<!-- 正确 -->
<textarea .value=\${content}></textarea>

<!-- 正确 -->
<textarea id="\${attrValue}">static content ok</textarea>
```

同样地，不应出现在带有 `contenteditable` 属性的元素内部。请改为绑定元素的 `.innerText` 属性。

```html
<!-- 注意 -->
<div contenteditable>\${content}</div>

<!-- 正确 -->
<div contenteditable .innerText=\${content}></div>

<!-- 正确 -->
<div contenteditable id="\${attrValue}">static content ok</div>
```

- 出现在 HTML 注释中。Lit 不会更新注释中的表达式，并且表达式将以 Lit 的标记字符串形式呈现。不过，这不会破坏后续的表达式，因此在开发过程中注释掉包含表达式的 HTML 片段是安全的。

```html
<!-- 不会更新: \${value} -->
```

- 出现在使用 ShadyCSS polyfill 的 `<style>` 元素中。有关详细信息，请参阅[表达式与样式元素](#expressions-and-style-elements)。

请注意，当使用静态表达式时，上述所有无效位置中的表达式都是有效的，但由于涉及到的效率问题，这些不应用于性能敏感的更新。


## 静态表达式

静态表达式会返回特殊的值，这些值会在模板被 Lit 处理为 HTML 之前被插入到模板中。因为它们成为了模板静态 HTML 的一部分，它们可以放置在模板中的任何地方——即使是通常不允许放置表达式的位置，例如属性名和标签名。

要使用静态表达式，必须从 Lit 的 `static-html` 模块中导入一个特殊版本的 `html` 或 `svg` 模板标签：

```javascript
import {html, literal} from 'lit/static-html.js';
```

`static-html` 模块包含支持静态表达式的 `html` 和 `svg` 标签函数，应该代替 `lit` 模块中提供的标准版本。使用 `literal` 标签函数来创建静态表达式。

你可以使用静态表达式来配置不太可能更改的选项，或者用于自定义模板中无法通过常规表达式进行自定义的部分——详情请参阅“[有效的表达式位置](#valid-expression-locations)”部分。例如，一个 `my-button` 组件可能渲染一个 `<button>` 标签，但一个子类可能会渲染一个 `<a>` 标签。由于该设置不经常更改且无法使用普通表达式自定义 HTML 标签，因此这是使用静态表达式的好地方。

```javascript
import {LitElement} from 'lit';
import {html, literal} from 'lit/static-html.js';

class MyButton extends LitElement {
  static properties = {
    caption: {},
    active: {type: Boolean},
  };

  tag = literal`button`;
  activeAttribute = literal`active`;

  constructor() {
    super();
    this.caption = 'Hello static';
    this.active = false;
  }

  render() {
    return html`
      <${this.tag} ${this.activeAttribute}=${this.active}>
        <p>${this.caption}</p>
      </${this.tag}>`;
  }
}
customElements.define('my-button', MyButton);
```

```javascript
class MyAnchor extends MyButton {
  tag = literal`a`;
}
customElements.define('my-anchor', MyAnchor);
```

更改静态表达式的值是昂贵的操作。使用 `literal` 值的表达式不应频繁更改，因为它们会导致重新解析新模板，并且每个变化都保存在内存中。

在上面的例子中，如果模板重新渲染并且 `this.caption` 或 `this.active` 更改，Lit 会高效地更新模板，仅更改受影响的表达式。然而，如果 `this.tag` 或 `this.activeAttribute` 更改，由于它们是使用 `literal` 标记的静态值，则会创建一个全新的模板；这种更新效率低下，因为整个 DOM 会被重新渲染。此外，传递给表达式的 `literal` 值的更改会增加内存使用，因为每个唯一模板都会被缓存到内存中以提高重新渲染性能。

因此，最好将 `literal` 表达式的更改保持在最小范围内，并避免使用反应性属性来更改 `literal` 值，因为反应性属性通常是为了变化而设计的。

### 模板结构

在静态值插入后，模板必须像普通的 Lit 模板一样结构良好，否则模板中的动态表达式可能无法正常工作。有关更多信息，请参阅“[结构良好的 HTML](#well-formed-html)”部分。

### 非 `literal` 静态值

在某些少见的情况下，你可能需要将不是在脚本中定义的静态 HTML 插入到模板中，因此无法使用 `literal` 函数进行标记。在这些情况下，可以使用 `unsafeStatic()` 函数来基于非脚本源的字符串创建静态 HTML。

```javascript
import {html, unsafeStatic} from 'lit/static-html.js';
```

**仅限受信任的内容。** 请注意 `unsafeStatic` 中 `unsafe` 的使用。传递给 `unsafeStatic()` 的字符串必须是开发者控制的，并且不能包含不受信任的内容，因为它会直接作为 HTML 进行解析而不进行任何净化。非受信任内容的例子包括查询字符串参数和用户输入的值。使用该指令渲染不受信任的内容可能会导致跨站脚本（XSS）漏洞。

```javascript
class MyButton extends LitElement {
  static properties = {
    caption: {},
    active: {type: Boolean},
  };

  constructor() {
    super();
    this.caption = 'Hello static';
    this.active = false;
  }

  render() {
    // 这些字符串必须是受信任的，否则可能会导致 XSS 漏洞
    const tag = getTagName();
    const activeAttribute = getActiveAttribute();
    return html`
      <${unsafeStatic(tag)} ${unsafeStatic(activeAttribute)}=${this.active}>
        <p>${this.caption}</p>
      </${unsafeStatic(tag)}>`;
  }
}
customElements.define('my-button', MyButton);
```

请注意，使用 `unsafeStatic` 的行为与 `literal` 具有相同的注意事项：因为更改值会导致重新解析新模板并将其缓存到内存中，所以这些值不应频繁更改。
