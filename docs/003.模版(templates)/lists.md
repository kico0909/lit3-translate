
# 列表

你可以使用标准的 JavaScript 构造来创建重复的模板。

Lit 还提供了一个 `repeat` 指令，用于更高效地构建某些类型的动态列表。

## 渲染数组

当子位置的表达式返回一个数组或可迭代对象时，Lit 会渲染数组中的所有项目：

```javascript
static properties = {
  colors: {},
};

constructor() {
  super();
  this.colors = ['red', 'green', 'blue'];
}

render() {
  return html`<p>Colors: ${this.colors}</p>`;
}
```

在大多数情况下，你会希望将数组项转换成更有用的形式。

## 使用 `map` 重复模板

要渲染列表，你可以使用 `map` 方法将数据列表转换为模板列表：

```javascript
static properties = {
  colors: {},
};

constructor() {
  super();
  this.colors = ['red', 'green', 'blue'];
}

render() {
  return html`
    <ul>
      ${this.colors.map((color) =>
        html`<li style="color: ${color}">${color}</li>`
      )}
    </ul>
  `;
}
```

注意，此表达式返回的是 `TemplateResult` 对象数组。Lit 会渲染一个由子模板和其他值组成的数组或可迭代对象。

## 使用循环语句重复模板

你也可以构建一个模板数组，并将其传递到模板表达式中。

```javascript
render() {
  const itemTemplates = [];
  for (const i of this.items) {
    itemTemplates.push(html`<li>${i}</li>`);
  }

  return html`
    <ul>
      ${itemTemplates}
    </ul>
  `;
}
```

## `repeat` 指令

在大多数情况下，使用循环或 `map` 是构建重复模板的高效方法。然而，如果你想要重新排序一个大型列表，或通过添加和删除个别条目来改变它，这种方法可能会涉及更新大量的 DOM 节点。

`repeat` 指令在这种情况下可以派上用场。

`repeat` 指令根据用户提供的键值高效地更新列表：

```javascript
repeat(items, keyFunction, itemTemplate)
```

其中：

- `items` 是一个数组或可迭代对象。
- `keyFunction` 是一个函数，接收单个项目作为参数，并返回该项目的唯一键值。
- `itemTemplate` 是一个模板函数，接收项目和其当前索引作为参数，并返回 `TemplateResult`。

例如：

```javascript
import {repeat} from 'lit/directives/repeat.js';

render() {
  return html`
  <ul>
    ${repeat(
      this.employees,
      (employee) => employee.id,
      (employee, index) => html`
      <li>${index}: ${employee.familyName}, ${employee.givenName}</li>
    `
    )}
  </ul>
  <button @click=${this.toggleSort}>Toggle sort</button>
`;
}
```

如果你重新排序 `employees` 数组，`repeat` 指令会重新排列现有的 DOM 节点。

## 何时使用 `map` 或 `repeat`

使用 `repeat` 是否更高效取决于你的使用场景：

- 如果更新 DOM 节点比移动它们更耗费性能，请使用 `repeat` 指令。
- 如果 DOM 节点的状态不受模板表达式控制，请使用 `repeat` 指令。

例如，考虑以下列表：

```javascript
html`${this.users.map((user) =>
  html`
    <div><input type="checkbox"> ${user.name}</div>
  `)
}`
```

复选框具有选中或未选中的状态，但该状态并未受到模板表达式的控制。

如果在用户选中一个或多个复选框后重新排序列表，Lit 会更新与复选框相关的名称，但不会更改复选框的状态。

如果上述两种情况都不适用，请使用 `map` 或循环语句。
