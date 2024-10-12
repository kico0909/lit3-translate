

# 上下文(Context)

`Context` 是一种让整个组件子树都可以使用数据的方式，而无需手动将属性绑定到每个组件。数据是“上下文可用”的，这意味着在数据的提供者和消费者之间的祖先元素甚至不会意识到该数据的存在。

Lit 的上下文实现可以在 `@lit/context` 包中找到：

```bash
npm i @lit/context
```

`Context` 适用于需要被广泛且大量组件使用的数据——例如应用程序的数据存储、当前用户、UI 主题——或者在数据绑定不是一种可选方式时使用，例如当元素需要向其光 DOM 子元素提供数据时。

`Context` 非常类似于 React 的 `Context`，或像 Angular 的依赖注入系统（DI），但有一些重要的区别使得 `Context` 能够与 DOM 的动态特性兼容，并且能够在不同的 Web 组件库、框架和纯 JavaScript 之间实现互操作性。

## 示例

使用上下文涉及到一个上下文对象（有时称为 key）、一个提供者和一个消费者，它们通过上下文对象进行通信。

Context definition (`logger-context.ts`):

```typescript
import {createContext} from '@lit/context';
import type {Logger} from 'my-logging-library';
export type {Logger} from 'my-logging-library';
export const loggerContext = createContext<Logger>('logger');
```

### 提供者

```typescript
import {LitElement, property, html} from 'lit';
import {provide} from '@lit/context';

import {Logger} from 'my-logging-library';
import {loggerContext} from './logger-context.js';

@customElement('my-app')
class MyApp extends LitElement {

  @provide({context: loggerContext})
  logger = new Logger();

  render() {
    return html`...`;
  }
}
```

### 消费者

```typescript
import {LitElement, property} from 'lit';
import {consume} from '@lit/context';

import {type Logger, loggerContext} from './logger-context.js';

export class MyElement extends LitElement {

  @consume({context: loggerContext})
  @property({attribute: false})
  public logger?: Logger;

  private doThing() {
    this.logger?.log('操作已执行');
  }
}
```

## 关键概念

### 上下文协议

Lit 的上下文基于 W3C Web Components 社区组提出的 `Context Community Protocol`。

该协议能够在不同元素（甚至是非元素代码）之间实现互操作性，而不论它们是如何构建的。通过上下文协议，基于 Lit 的元素可以向非 Lit 构建的消费者提供数据，反之亦然。

上下文协议基于 DOM 事件。消费者会触发一个 `context-request` 事件，该事件携带它想要的上下文键，任何位于其上方的元素都可以监听 `context-request` 事件，并为该上下文键提供数据。

`@lit/context` 实现了这一基于事件的协议，并通过一些响应式控制器和装饰器提供了该协议的使用方式。

### 上下文对象

上下文是通过上下文对象或上下文键来识别的。它们是代表某些可能需要通过上下文对象标识共享的数据的对象。你可以将它们视为类似于 `Map` 键的概念。

### 提供者

提供者通常是元素（但也可以是任何事件处理代码），它们为特定的上下文键提供数据。

### 消费者

消费者请求特定上下文键的数据。

### 订阅

当消费者请求某个上下文的数据时，它可以告知提供者希望订阅该上下文中的数据变更。如果提供者有新数据，消费者将会收到通知，并且可以自动更新。


## 使用

### 定义上下文

每个上下文的使用都必须有一个上下文对象来协调数据请求。该上下文对象表示所提供数据的身份和类型。

上下文对象通过 `createContext()` 函数创建：

```typescript
export const myContext = createContext(Symbol('my-context'));
```

建议将上下文对象放在单独的模块中，这样它们可以独立于特定的提供者和消费者进行导入。

### 上下文类型检查

`createContext()` 接受任何值并直接返回该值。在 TypeScript 中，该值会被转换为带有类型的上下文对象，并且该对象会携带上下文值的类型。

以下代码会导致错误：

```typescript
const myContext = createContext<Logger>(Symbol('logger'));

class MyElement extends LitElement {
  @provide({context: myContext})
  name: string;
}
```

TypeScript 会警告 `string` 类型不能分配给 `Logger` 类型。请注意，此检查目前仅适用于公共字段。

### 上下文相等性

上下文对象由提供者用于将上下文请求事件与值进行匹配。上下文通过严格相等性（`===`）进行比较，因此只有当提供者的上下文键与请求的上下文键相等时，它才会处理该上下文请求。

这意味着创建上下文对象主要有两种方式：

- 使用全局唯一的值，如对象 (`{}`) 或符号 (`Symbol()`)
- 使用非全局唯一的值，以便在严格相等性下相等，如字符串 (`'logger'`) 或全局符号 (`Symbol.for('logger')`)

如果你希望两个独立的 `createContext()` 调用指向相同的上下文，则使用在严格相等性下相等的键，如字符串：

```typescript
// 结果为 true
createContext('my-context') === createContext('my-context')
```

但要注意，应用程序中的两个模块可能会使用相同的上下文键来引用不同的对象。为了避免意外的冲突，建议使用相对唯一的字符串，例如使用 `'console-logger'` 而不是 `'logger'`。

通常最好使用全局唯一的上下文对象，符号是实现这一点的最简单方法之一。

### 提供上下文

> 链接到 “提供上下文”

在 `@lit/context` 中，有两种方式可以提供上下文值：`ContextProvider` 控制器和 `@provide()` 装饰器。

#### `@provide()` 装饰器

如果使用装饰器，`@provide()` 是提供值的最简单方法。它会为你创建一个 `ContextProvider` 控制器。

使用 `@provide()` 装饰属性，并提供上下文键：

```typescript
import {LitElement, html} from 'lit';
import {property} from 'lit/decorators.js';
import {provide} from '@lit/context';
import {myContext, MyData} from './my-context.js';

class MyApp extends LitElement {
  @provide({context: myContext})
  myData: MyData;
}
```

你还可以使用 `@property()` 或 `@state()` 将该属性设为响应式属性，这样在设置它时将同时更新提供者元素和上下文消费者。

```typescript
@provide({context: myContext})
@property({attribute: false})
myData: MyData;
```

上下文属性通常是私有的。你可以使用 `@state()` 将私有属性设为响应式：

```typescript
@provide({context: myContext})
@state()
private _myData: MyData;
```

将上下文属性设为公共的，可以让元素向其子树提供公共字段：

```typescript
html`<my-provider-element .myData=${someData}>`
```

#### `ContextProvider`

`ContextProvider` 是一个响应式控制器，它会为你管理 `context-request` 事件处理程序。

```typescript
import {LitElement, html} from 'lit';
import {ContextProvider} from '@lit/context';
import {myContext} from './my-context.js';

export class MyApp extends LitElement {
  private _provider = new ContextProvider(this, {context: myContext});
}
```

`ContextProvider` 可以在构造函数中通过选项接受初始值：

```typescript
private _provider = new ContextProvider(this, {context: myContext, initialValue: myData});
```

你也可以调用 `setValue()`：

```typescript
this._provider.setValue(myData);
```

### 使用上下文

#### `@consume()` 装饰器

如果使用装饰器，`@consume()` 是使用值的最简单方法。它会为你创建一个 `ContextConsumer` 控制器。

使用 `@consume()` 装饰属性，并提供上下文键：

```typescript
import {LitElement, html} from 'lit';
import {consume} from '@lit/context';
import {myContext, MyData} from './my-context.js';

class MyElement extends LitElement {
  @consume({context: myContext})
  myData: MyData;
}
```

当该元素连接到文档时，它会自动触发一个 `context-request` 事件，获取提供的值，将其分配给属性，并触发元素更新。

#### `ContextConsumer`

`ContextConsumer` 是一个响应式控制器，它会为你管理 `context-request` 事件的调度。控制器会在提供新值时导致宿主元素更新。提供的值随后可以通过控制器的 `.value` 属性获取。

```typescript
import {LitElement, property} from 'lit';
import {ContextConsumer} from '@lit/context';
import {myContext} from './my-context.js';

export class MyElement extends LitElement {
  private _myData = new ContextConsumer(this, {context: myContext});

  render() {
    const myData = this._myData.value;
    return html`...`;
  }
}
```

### 订阅上下文

消费者可以订阅上下文值，这样如果提供者有新值，可以将其提供给所有订阅的消费者，从而导致它们更新。

你可以使用 `@consume()` 装饰器进行订阅：

```typescript
@consume({context: myContext, subscribe: true})
myData: MyData;
```

也可以使用 `ContextConsumer` 控制器进行订阅：

```typescript
private _myData = new ContextConsumer(this, {
  context: myContext,
  subscribe: true,
});
```


## 示例使用场景

### 当前用户、本地化等

最常见的上下文使用场景涉及页面全局数据，并且这些数据可能仅在整个页面中零散地被组件需要。如果没有上下文，可能大多数或所有组件都需要接受和传递用于这些数据的响应式属性。

### 服务

应用全局服务（如日志记录器、分析工具、数据存储）可以通过上下文提供。上下文相对于从公共模块导入的优势在于，它提供了松耦合和树作用域的特性。测试时可以轻松提供模拟服务，或者页面的不同部分可以使用不同的服务实例。

### 主题

主题是一组应用于整个页面或页面中整个子树的样式——正是上下文所能提供的数据作用域类型之一。

构建主题系统的一种方法是定义一个 `Theme` 类型，容器可以提供该类型并持有命名样式。希望应用主题的元素可以消费该主题对象，并通过名称查找样式。自定义主题的响应式控制器可以封装 `ContextProvider` 和 `ContextConsumer`，以减少样板代码。

### 基于 HTML 的插件

上下文可以用于将数据从父元素传递给其光 DOM 子元素。由于父元素通常不会创建光 DOM 子元素，它无法利用基于模板的数据绑定将数据传递给它们，但可以监听和响应 `context-request` 事件。

例如，考虑一个具有不同语言模式插件的代码编辑器元素。你可以使用上下文构建一个简单的 HTML 系统来添加功能：

```html
<code-editor>
  <code-editor-javascript-mode></code-editor-javascript-mode>
  <code-editor-python-mode></code-editor-python-mode>
</code-editor>
```

在这种情况下，`<code-editor>` 可以通过上下文提供添加语言模式的 API，而插件元素则可以消费该 API，并将自身添加到编辑器中。

### 数据格式化、链接生成器等

有时可重用的组件需要以应用程序特定的方式格式化数据或 URL。例如，一个文档查看器需要渲染指向另一个条目的链接。组件不会了解应用程序的 URL 空间。

在这些情况下，组件可以依赖上下文提供的函数来将应用程序特定的格式应用于数据或链接。
