
# 条件渲染

由于 Lit 使用的是普通的 JavaScript 表达式，因此你可以使用标准的 JavaScript 控制流构造（如条件运算符、函数调用、`if` 或 `switch` 语句）来渲染条件内容。

JavaScript 条件语句还允许你结合嵌套的模板表达式，甚至可以将模板结果存储在变量中以供其他地方使用。

## 使用条件（三元）运算符的条件渲染
使用条件运算符 `?` 的三元表达式是一种添加内联条件渲染的好方法：

```javascript
render() {
  return this.userName
    ? html`Welcome ${this.userName}`
    : html`Please log in <button>Login</button>`;
}
```

## 使用 `if` 语句的条件渲染
你可以在模板外使用 `if` 语句来表达条件逻辑，以计算出要在模板中使用的值：

```javascript
render() {
  let message;
  if (this.userName) {
    message = html`Welcome ${this.userName}`;
  } else {
    message = html`Please log in <button>Login</button>`;
  }
  return html`<p class="message">${message}</p>`;
}
```

或者，你可以将逻辑提取到一个单独的函数中，从而简化模板：

```javascript
getUserMessage() {
  if (this.userName) {
    return html`Welcome ${this.userName}`;
  } else {
    return html`Please log in <button>Login</button>`;
  }
}
render() {
  return html`<p>${this.getUserMessage()}</p>`;
}
```

## 缓存模板结果：`cache` 指令
在大多数情况下，JavaScript 条件语句已经足够应对条件模板渲染。然而，如果你在复杂的大模板之间切换时，希望节省每次切换时重新创建 DOM 的开销，则可以使用 `cache` 指令。

`cache` 指令会缓存当前未渲染的模板对应的 DOM。

```javascript
render() {
  return html`${cache(this.userName ?
    html`Welcome ${this.userName}`:
    html`Please log in <button>Login</button>`)
  }`;
}
```
查看更多关于 `cache` 指令的信息。

## 条件渲染空内容
有时，你可能希望在条件分支中渲染空内容。这在子表达式中很常见，有时也会在属性表达式中用到。

对于子表达式，`undefined`、`null`、空字符串 (`''`) 和 Lit 的 `nothing` 哨兵值均不会渲染任何节点。详见[删除子内容](#)以获取更多信息。

以下示例展示了如何在值存在时渲染它，否则不渲染任何内容：

```javascript
render() {
  return html`<user-name>${this.userName ?? nothing}</user-name>`;
}
```

对于属性表达式，Lit 的 `nothing` 哨兵值会移除该属性。详见[删除属性](#)以获取更多信息。

以下示例展示了如何有条件地渲染 `aria-label` 属性：

```javascript
html`<button aria-label="${this.ariaLabel || nothing}"></button>`
```
