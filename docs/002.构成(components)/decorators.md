
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

:::tip
Lit 支持 JavaScript 装饰器提案的两个不同版本——一个是由 TypeScript 支持的早期版本（我们称之为实验性装饰器），另一个是新的最终版本（我们称之为标准装饰器）。

这两个提案在使用上有一些细微的差异（标准装饰器通常需要使用 `accessor` 关键字）。我们的代码示例是基于实验性装饰器编写的，因为我们目前推荐在生产环境中使用它们。

有关更多详细信息，请参阅 [装饰器版本](https://lit.dev/docs/components/decorators/#versions)。

:::
