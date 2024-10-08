
# 定义组件

通过创建一个继承自 `LitElement` 的类并将其注册到浏览器中，可以定义一个 Lit 组件：

```typescript
@customElement('simple-greeting')
export class SimpleGreeting extends LitElement { /* ... */ }
```

`@customElement` 装饰器是 `customElements.define` 的简写，它将自定义元素类注册到浏览器，并将其与元素名称（在此例中为 `simple-greeting`）相关联。

如果使用 JavaScript，或者不使用装饰器，也可以直接调用 `define()` 方法：

```typescript
export class SimpleGreeting extends LitElement { /* ... */  }
customElements.define('simple-greeting', SimpleGreeting);
```

## Lit 组件是 HTML 元素

当你定义一个 Lit 组件时，你实际上是在定义一个自定义的 HTML 元素。因此，你可以像使用内置元素一样使用该新元素：

```html
<simple-greeting name="Markup"></simple-greeting>
```

```typescript
const greeting = document.createElement('simple-greeting');
```

`LitElement` 基类继承自 `HTMLElement`，因此 Lit 组件继承了所有标准 `HTMLElement` 的属性和方法。

具体来说，`LitElement` 继承自 `ReactiveElement`，而 `ReactiveElement` 又继承自 `HTMLElement`。

继承关系如下所示：

- `LitElement` 负责模板渲染。
- `ReactiveElement` 负责管理响应式属性和属性。
- `HTMLElement` 是所有原生 HTML 元素和自定义元素共享的标准 DOM 接口。

## 提供良好的 TypeScript 类型支持

TypeScript 会根据标签名称推断从某些 DOM API 返回的 HTML 元素的类型。例如，`document.createElement('img')` 返回一个具有 `src: string` 属性的 `HTMLImageElement` 实例。

通过将自定义元素添加到 `HTMLElementTagNameMap` 中，自定义元素也可以享受相同的类型推断：

```typescript
@customElement('my-element')
export class MyElement extends LitElement {
  @property({type: Number})
  aNumber: number = 5;
  /* ... */
}

declare global {
  interface HTMLElementTagNameMap {
    "my-element": MyElement;
  }
}
```

这样，以下代码就能够正确地进行类型检查：

```typescript
const myElement = document.createElement('my-element');
myElement.aNumber = 10;
```

我们建议为所有使用 TypeScript 编写的自定义元素添加 `HTMLElementTagNameMap` 条目，并确保在 npm 包中发布 `.d.ts` 类型声明文件。
