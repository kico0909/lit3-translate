
# 自定义指令（Custom Directives）

指令是可以通过自定义模板表达式的渲染方式来扩展 Lit 的函数。指令非常有用且功能强大，因为它们可以是有状态的、访问 DOM、在模板断开连接和重新连接时接收通知，并且能够在 `render` 调用之外独立更新表达式。

在模板中使用指令就像在模板表达式中调用一个函数一样简单：

```js
html`<div>
       ${fancyDirective('一些文本')}
     </div>`
```

Lit 内置了许多指令，如 `repeat()` 和 `cache()`。用户也可以编写自己的自定义指令。

指令有两种类型：

- **简单函数（Simple Functions）**
- **基于类的指令（Class-based Directives）**

简单函数返回一个要渲染的值。它可以接受任意数量的参数，甚至不接受任何参数：

```js
export noVowels = (str) => str.replaceAll(/[aeiou]/ig,'x');
```

基于类的指令可以做简单函数无法完成的任务。使用基于类的指令可以：

- 直接访问渲染的 DOM（例如，添加、删除或重新排列渲染的 DOM 节点）。
- 在渲染之间持久化状态。
- 异步地更新 DOM，而无需调用 `render`。
- 当指令从 DOM 中断开时清理资源。

以下内容描述了如何创建基于类的指令。

## 创建基于类的指令

要创建基于类的指令，请执行以下步骤：

1. 实现一个继承自 `Directive` 类的指令类。
2. 将您的类传递给 `directive()` 工厂函数，以创建一个可以在 Lit 模板表达式中使用的指令函数。

```js
import {Directive, directive} from 'lit/directive.js';

// 定义指令
class HelloDirective extends Directive {
  render() {
    return `Hello!`;
  }
}

// 创建指令函数
const hello = directive(HelloDirective);

// 使用指令
const template = html`<div>${hello()}</div>`;
```

当此模板被解析时，指令函数 (`hello()`) 返回一个 `DirectiveResult` 对象，该对象指示 Lit 创建或更新指令类 (`HelloDirective`) 的实例。然后，Lit 会调用指令实例上的方法来执行更新逻辑。

一些指令需要在正常更新周期之外异步更新 DOM。要创建异步指令，可以继承 `AsyncDirective` 基类而不是 `Directive`。详情请参阅 [异步指令（Async Directives）](#)。

## 基于类的指令的生命周期

指令类有一些内置的生命周期方法：

- 类构造函数（`constructor`），用于一次性初始化。
- `render()` 方法，用于声明性渲染。
- `update()` 方法，用于命令式 DOM 访问。

所有指令都必须实现 `render()` 回调。`update()` 是可选的。`update()` 的默认实现是调用并返回 `render()` 的值。

异步指令（可以在正常更新周期之外更新 DOM）使用一些额外的生命周期回调。详情请参阅 [异步指令（Async Directives）](#)。

### 一次性设置：`constructor()`

当 Lit 首次在表达式中遇到 `DirectiveResult` 时，它会构造相应指令类的实例（这会导致指令的构造函数和任何类字段初始化程序被执行）：

```js
class MyDirective extends Directive {
  // 类字段将仅初始化一次，并可用于在渲染之间持久化状态
  value = 0;

  // 构造函数只在指令首次用于表达式时执行
  constructor(partInfo) {
    super(partInfo);
    console.log('MyDirective created');
  }
  ...
}
```

只要每次渲染中使用的是相同的指令函数，并且用于相同的表达式位置，那么会重用之前的实例，从而在渲染之间保持实例状态的持久性。

构造函数接收一个 `PartInfo` 对象，该对象提供有关表达式的元数据。对于仅设计用于特定类型表达式的指令，这非常有用（参见 [限制指令的表达式类型](#)）。

### 声明式渲染：`render()`

`render()` 方法应返回要渲染到 DOM 中的值。它可以返回任何可渲染的值，包括另一个 `DirectiveResult`。

除了引用指令实例上的状态外，`render()` 方法还可以接受传递给指令函数的任意参数：

```js
const template = html`<div>${myDirective(name, rank)}</div>`
```

`render()` 方法定义的参数决定了指令函数的签名：

```js
class MaxDirective extends Directive {
  maxValue = Number.MIN_VALUE;
  
  // 定义一个 `render` 方法，可以接受参数
  render(value, minValue = Number.MIN_VALUE) {
    this.maxValue = Math.max(value, this.maxValue, minValue);
    return this.maxValue;
  }
}
const max = directive(MaxDirective);

// 调用指令时传递 `render()` 方法定义的 `value` 和 `minValue` 参数
const template = html`<div>${max(someNumber, 0)}</div>`;
```

### 命令式 DOM 访问：`update()`

在更高级的用例中，指令可能需要访问底层 DOM 并进行命令式读写。可以通过覆盖 `update()` 回调来实现这一点。

`update()` 回调接收两个参数：

- `Part` 对象，提供用于直接管理与表达式关联的 DOM 的 API。
- 包含 `render()` 方法参数的数组。

`update()` 方法应返回 Lit 可以渲染的内容，或者返回特殊值 `noChange`，以指示无需重新渲染。`update()` 回调非常灵活，但典型的用法包括：

- 从 DOM 中读取数据，并使用这些数据生成要渲染的值。
- 使用 `Part` 对象上的 `element` 或 `parentNode` 引用以命令式方式更新 DOM。在这种情况下，通常返回 `noChange`，表示 Lit 无需进一步操作来渲染指令。

#### Parts

每个表达式位置都有其特定的 `Part` 对象：

- `ChildPart`：用于 HTML 子节点位置的表达式。
- `AttributePart`：用于 HTML 属性值位置的表达式。
- `BooleanAttributePart`：用于布尔属性值（名称以 `?` 为前缀）的表达式。
- `EventPart`：用于事件监听器位置（名称以 `@` 为前缀）的表达式。
- `PropertyPart`：用于属性值位置（名称以 `.` 为前缀）的表达式。
- `ElementPart`：用于元素标签上的表达式。

除了 `PartInfo` 中包含的特定元数据外，所有 `Part` 类型都提供了与表达式关联的 DOM 元素的访问权限（对于 `ChildPart`，可以访问 `parentNode`）。例如：

```js
class AttributeLogger extends Directive {
  attributeNames = '';

  update(part) {
    this.attributeNames = part.parentNode.getAttributeNames?.().join(' ');
    return this.render();
  }

  render() {
    return this.attributeNames;
  }
}
const attributeLogger = directive(AttributeLogger);
const template = html`<div a b>${attributeLogger()}</div>`;
// 渲染结果：`<div a b>a b</div>`
```
`directive-helpers.js` 模块包含了许多操作 `Part` 对象的辅助函数，可以用来动态地在指令的 `ChildPart` 中创建、插入和移动部分内容。

#### 从 `update()` 中调用 `render()`

> 链接到 “从 `update()` 中调用 `render()`”

`update()` 的默认实现只是简单地调用并返回 `render()` 的值。如果你重写了 `update()`，并且仍然希望调用 `render()` 来生成值，你需要显式地调用 `render()`。

`render()` 的参数会作为数组传递给 `update()`。你可以像下面这样将参数传递给 `render()`：

```javascript
class MyDirective extends Directive {
  update(part, [fish, bananas]) {
    // ...
    return this.render(fish, bananas);
  }
  render(fish, bananas) { ... }
}
```

### `update()` 和 `render()` 的区别

> 链接到 “`update()` 和 `render()` 的区别”

虽然 `update()` 回调比 `render()` 回调更强大，但有一个重要的区别：当使用 `@lit-labs/ssr` 包进行服务器端渲染（SSR）时，只会在服务器上调用 `render()` 方法。为了兼容 SSR，指令应该从 `render()` 中返回值，并且仅在 `update()` 中使用那些需要访问 DOM 的逻辑。

## 表示无更改

有时指令可能没有新的内容需要 Lit 渲染。你可以通过从 `update()` 或 `render()` 方法中返回 `noChange` 来表示这一点。这与返回 `undefined` 不同，后者会导致 Lit 清除与指令关联的 `Part`。返回 `noChange` 则表示保持之前渲染的值不变。

返回 `noChange` 有以下几种常见情况：

- 基于输入值，没有新的内容需要渲染。
- `update()` 方法以命令式方式更新了 DOM。
- 在异步指令中，由于没有内容需要渲染，`update()` 或 `render()` 可能会返回 `noChange`。

例如，一个指令可以追踪之前传递给它的值，并执行脏检查（dirty checking）来确定指令的输出是否需要更新。`update()` 或 `render()` 方法可以返回 `noChange` 来表示指令的输出不需要重新渲染。

```javascript
import {Directive} from 'lit/directive.js';
import {noChange} from 'lit';

class CalculateDiff extends Directive {
  render(a, b) {
    if (this.a !== a || this.b !== b) {
      this.a = a;
      this.b = b;
      // 昂贵且复杂的文本差异算法
      return calculateDiff(a, b);
    }
    return noChange;
  }
}
```

## 限制指令只应用于一种表达式类型

> 链接到 “限制指令只应用于一种表达式类型”

某些指令仅在特定上下文中有用，比如属性表达式或子表达式。如果放置在错误的上下文中，指令应当抛出一个合适的错误。

例如，`classMap` 指令验证它仅用于 `AttributePart` 中，并且只用于 `class` 属性。

```javascript
class ClassMap extends Directive {
  constructor(partInfo) {
    super(partInfo);
    if (
      partInfo.type !== PartType.ATTRIBUTE ||
      partInfo.name !== 'class'
    ) {
      throw new Error('`classMap` 指令必须用于 `class` 属性中');
    }
  }
}
```


## 异步指令

前面的示例指令都是同步的：它们从 `render()`/`update()` 生命周期回调中同步返回值，因此它们的结果会在组件的 `update()` 回调中写入到 DOM 中。

有时你可能希望指令能够异步更新 DOM，例如当它依赖于某个异步事件（如网络请求）时。

要实现指令结果的异步更新，指令需要继承 `AsyncDirective` 基类，该基类提供了一个 `setValue()` API。`setValue()` 允许指令在模板的正常更新/渲染周期之外，将新的值“推送”到模板表达式中。

以下是一个简单的异步指令示例，该指令会渲染一个 Promise 值：

```javascript
class ResolvePromise extends AsyncDirective {
  render(promise) {
    Promise.resolve(promise).then((resolvedValue) => {
      // 异步渲染：
      this.setValue(resolvedValue);
    });
    // 同步渲染：
    return `Waiting for promise to resolve`;
  }
}
export const resolvePromise = directive(ResolvePromise);
```

在这个示例中，渲染的模板会先显示 "Waiting for promise to resolve"，然后在 Promise 解析后显示解析的值。

异步指令通常需要订阅外部资源。为了防止内存泄漏，当指令实例不再使用时，异步指令应取消订阅或释放这些资源。为此，`AsyncDirective` 提供了以下额外的生命周期回调和 API：

- `disconnected()`：在指令不再使用时调用。在以下三种情况下，指令实例会被断开：
  1. 包含该指令的 DOM 树被从 DOM 中移除。
  2. 指令的宿主元素被断开连接。
  3. 生成该指令的表达式不再解析为同一个指令。
  
  在指令接收到 `disconnected` 回调之后，它应当释放所有在 `update` 或 `render` 中可能订阅的资源，以防止内存泄漏。

- `reconnected()`：在先前断开连接的指令被重新使用时调用。由于 DOM 子树可以暂时断开并稍后再次重新连接，因此断开连接的指令可能需要对重新连接作出反应。这种情况的示例包括当 DOM 被移除并缓存以供后续使用时，或者宿主元素被移动导致断开和重新连接。`reconnected()` 回调应始终与 `disconnected()` 一起实现，以将断开连接的指令恢复到正常工作状态。

- `isConnected`：反映指令的当前连接状态。

需要注意的是，如果包含指令的树被重新渲染，异步指令在断开连接后仍可能接收更新。因此，`update` 和/或 `render` 方法在订阅任何长期资源之前应始终检查 `this.isConnected` 标志，以防止内存泄漏。

下面是一个订阅 `Observable` 的指令，并且能够正确处理断开和重新连接的示例：

```javascript
class ObserveDirective extends AsyncDirective {
  // 当 observable 变化时，取消订阅旧的 observable 并订阅新的
  render(observable) {
    if (this.observable !== observable) {
      this.unsubscribe?.();
      this.observable = observable
      if (this.isConnected)  {
        this.subscribe(observable);
      }
    }
    return noChange;
  }
  // 订阅 observable，每次值发生变化时调用指令的异步 setValue API
  subscribe(observable) {
    this.unsubscribe = observable.subscribe((v) => {
      this.setValue(v);
    });
  }
  // 当指令从 DOM 中断开时，取消订阅以确保指令实例能够被垃圾回收
  disconnected() {
    this.unsubscribe();
  }
  // 如果指令所在的子树被断开连接并随后重新连接，再次订阅以使指令恢复正常
  reconnected() {
    this.subscribe(this.observable);
  }
}
export const observe = directive(ObserveDirective);
```
