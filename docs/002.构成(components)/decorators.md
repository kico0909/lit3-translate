
# 装饰器 (Decorators)

装饰器是可以用于声明性地注解和修改类行为的函数。

Lit 提供了一组可选的装饰器，使得可以以声明性的方式使用 API 来执行例如注册元素、定义响应式属性、查询属性或为事件处理方法添加事件选项等操作。

例如，`@customElement` 和 `@property()` 装饰器可以让你以紧凑、声明式的方式注册一个自定义元素，并定义一个响应式属性：

```typescript
@customElement('my-element')
export class MyElement extends LitElement {

  @property()
  greeting = 'Welcome';

}
```

```note
Lit 支持 JavaScript 装饰器提案的两个不同版本——一个是由 TypeScript 支持的早期版本（我们称之为实验性装饰器），另一个是新的最终版本（我们称之为标准装饰器）。

这两个提案在使用上有一些细微的差异（标准装饰器通常需要使用 `accessor` 关键字）。我们的代码示例是基于实验性装饰器编写的，因为我们目前推荐在生产环境中使用它们。

有关更多详细信息，请参阅 [装饰器版本](https://lit.dev/docs/components/decorators/#versions)。

```


## 内置装饰器 (Built-in decorators)

| 装饰器 | 描述 | 更多信息 |
| --- | --- | --- |
| `@customElement` | 定义自定义元素。 | [定义](https://lit.dev/docs/components/decorators/#customElement) |
| `@eventOptions` | 添加事件监听器选项。 | [事件](https://lit.dev/docs/components/events/) |
| `@property` | 定义公共属性。 | [属性](https://lit.dev/docs/components/properties/) |
| `@state` | 定义私有状态属性。 | [属性](https://lit.dev/docs/components/properties/#state) |
| `@query` | 定义一个属性，该属性返回组件模板中的元素。 | [Shadow DOM](https://lit.dev/docs/components/shadow-dom/) |
| `@queryAll` | 定义一个属性，该属性返回组件模板中的元素列表。 | [Shadow DOM](https://lit.dev/docs/components/shadow-dom/#queryAll) |
| `@queryAsync` | 定义一个属性，该属性返回一个 Promise，该 Promise 解析为组件模板中的元素。 | [Shadow DOM](https://lit.dev/docs/components/shadow-dom/#queryAsync) |
| `@queryAssignedElements` | 定义一个属性，该属性返回分配给特定 slot 的子元素。 | [Shadow DOM](https://lit.dev/docs/components/shadow-dom/#queryAssignedElements) |
| `@queryAssignedNodes` | 定义一个属性，该属性返回分配给特定 slot 的子节点。 | [Shadow DOM](https://lit.dev/docs/components/shadow-dom/#queryAssignedNodes) |

## 导入装饰器 (Importing decorators)

你可以通过 `lit/decorators.js` 模块导入所有的 Lit 装饰器：

```typescript
import {customElement, property, eventOptions, query} from 'lit/decorators.js';
```

为了减少组件运行时所需的代码量，你还可以单独导入装饰器。所有装饰器均可通过 `lit/decorators/<decorator-name>.js` 进行单独导入。例如：

```typescript
import {customElement} from 'lit/decorators/custom-element.js';
import {eventOptions} from 'lit/decorators/event-options.js';
```

## 启用装饰器 (Enabling decorators)

要使用装饰器，你需要使用编译器（如 TypeScript 或 Babel）来构建你的代码。

未来，当浏览器原生支持装饰器时，将不再需要编译器的支持。

### 在 TypeScript 中使用装饰器 (Using decorators with TypeScript)

TypeScript 支持实验性装饰器和标准装饰器。我们建议 TypeScript 开发者目前使用实验性装饰器以获得最佳的编译器输出。如果你的项目需要使用标准装饰器或设置 `"useDefineForClassFields": true`，请参阅“迁移到标准装饰器”部分。

要使用实验性装饰器，你必须启用 `experimentalDecorators` 编译器选项。

你还应该确保 `useDefineForClassFields` 设置为 `false`。这在 `target` 设置为 `ES2022` 或更高时是必须的，但我们建议显式将其设置为 `false`。这可以避免在声明属性时遇到类字段的问题。

```json
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "useDefineForClassFields": false,
  }
}
```

不需要启用 `emitDecoratorMetadata`，且不推荐启用该选项。

#### 将 TypeScript 实验性装饰器迁移到标准装饰器 (Migrating TypeScript experimental decorators to standard decorators)

Lit 装饰器设计为支持标准装饰器语法（在类字段装饰器中使用 `accessor`）以及 TypeScript 的实验性装饰器模式。

这允许在不改变行为的情况下，从实验性装饰器逐步迁移。迁移的第一步是为所有被装饰的类字段添加 `accessor` 关键字。完成这一步后，你就可以更改编译器选项以完成向标准装饰器的迁移：

```json
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": false, // TypeScript 5.0 及以上版本默认值
    "useDefineForClassFields": true, // 当 "target" 设置为 "ES2022" 或更高时的默认值
  }
}
```

注意：`accessor` 关键字在 TypeScript 4.9 中引入，且带有元数据的标准装饰器需要 TypeScript ≥5.2。

### 在 Babel 中使用装饰器 (Using decorators with Babel)

Babel 自 7.23 版本起支持标准装饰器，并使用 `@babel/plugin-proposal-decorators` 插件。Babel 不支持 TypeScript 的实验性装饰器，因此你必须在带有 `accessor` 关键字的类字段中使用标准装饰器语法来使用 Lit 装饰器。

通过添加 `@babel/plugin-proposal-decorators` 插件并使用以下 Babel 配置启用装饰器：

```json
// babel.config.json
{
  "plugins": [
    ["@babel/plugin-proposal-decorators", {"version": "2023-05"}]
  ]
}
```

注意：Lit 装饰器仅支持 `"version": "2023-05"`。其他版本（包括以前支持的 `"2018-09"`）不受支持。


## 装饰器版本 (Decorator Versions)

装饰器是正在被纳入 ECMAScript 标准的第 3 阶段提案。虽然目前没有任何浏览器原生实现了装饰器，但像 Babel 和 TypeScript 这样的编译器已经支持了装饰器。Lit 的装饰器可以与 Babel 和 TypeScript 一起使用，并且当浏览器原生支持时，Lit 装饰器也能够无缝运行。

```note
什么是第 3 阶段提案？ (What does stage 3 mean?)这意味着规范文本已经完成，并且准备好供浏览器实现。一旦该规范被多个浏览器实现，它将进入最终阶段——第 4 阶段，并被正式纳入 ECMAScript 标准。第 3 阶段的提案只有在实现过程中发现关键问题时才会发生变更。
```

### 早期的装饰器提案 (Earlier decorator proposals)

在 TC39 提案进入第 3 阶段之前，编译器实现了早期版本的装饰器规范。

其中最显著的是 TypeScript 的实验性装饰器（experimental decorators），自 Lit 诞生以来，Lit 一直支持这种装饰器，并且我们目前也推荐使用该版本。

Babel 也曾在不同的时间点支持过不同版本的装饰器规范，这可以从装饰器插件的 `version` 选项中看出。过去，Lit 2 曾支持 Babel 用户使用的 `2018-09` 版本，但现已改为支持下面描述的 `2023-05` 标准版本。

### 标准装饰器 (Standard decorators)

标准装饰器是 TC39 中负责定义 ECMAScript/JavaScript 规范的机构已达成第 3 阶段共识的装饰器版本。

标准装饰器在 TypeScript 和 Babel 中均已支持，并且不久的将来会在浏览器中实现原生支持。

标准装饰器与实验性装饰器最大的区别在于，出于性能原因，标准装饰器不能更改类成员的类型（字段、访问器、方法）并进行替换，而只能生成相同类型的成员。

由于许多 Lit 装饰器会生成访问器，因此在使用标准装饰器时，这些装饰器需要应用于访问器或使用 `accessor` 关键字声明的自动访问器。

#### 自动访问器 (Auto-Accessors)

为了便于使用，标准装饰器规范新增了 `accessor` 关键字来声明“自动访问器”：

```typescript
class MyClass {
  accessor foo = 42;
}
```

自动访问器会创建一对读取和写入私有字段的 getter 和 setter。然后，装饰器可以包装这些 getter 和 setter。

对于在实验性装饰器中作用于类字段的 Lit 装饰器（如 `@property()`、`@state()`、`@query()` 等），在使用标准装饰器时，必须应用于访问器或自动访问器上：

```typescript
@customElement('my-element')
export class MyElement extends LitElement {

  @property()
  accessor greeting = 'Welcome';

}
```

### 编译器输出的考虑 (Compiler output considerations)

标准装饰器的编译器输出较大，因为生成访问器、私有存储以及装饰器 API 相关的其他对象会占用大量代码量。

因此，我们建议希望使用装饰器的用户，尽可能继续使用 TypeScript 的实验性装饰器。

未来，Lit 团队计划在可选的 Lit 编译器中添加装饰器转换功能，以便将标准装饰器编译为更紧凑的输出。此外，浏览器原生支持将消除对任何编译器转换的需求。
