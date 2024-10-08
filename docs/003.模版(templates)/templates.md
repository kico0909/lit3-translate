
# 模板概述 (Templates Overview)

Lit 模板是使用带有 `html` 标签的 JavaScript 模板字面量编写的。字面量的内容主要是普通的声明式 HTML：

```javascript
html`<h1>Hello ${name}</h1>`
```

模板语法看起来像是简单的字符串插值。但实际上，在使用带标签的模板字面量时，浏览器会将标签函数传递一个包含字符串数组（模板中的静态部分）和表达式数组（动态部分）的参数。Lit 利用这一点来构建模板的高效表示形式，从而可以只重新渲染那些发生更改的模板部分。

Lit 模板非常具有表现力，并允许你以多种方式渲染动态内容：

### 表达式 (Expressions)

模板可以包含称为“表达式”的动态值，可以用于渲染属性、文本、属性（property）、事件处理程序，甚至其他模板。

### 条件渲染 (Conditionals)

可以使用标准 JavaScript 流控制在表达式中进行条件渲染。

### 列表渲染 (Lists)

通过使用标准 JavaScript 循环和数组技术将数据转换为模板数组来渲染列表。

### 内置指令 (Built-in Directives)

指令（directives）是可以扩展 Lit 模板功能的函数。Lit 库中包含一组内置指令，能够帮助处理各种渲染需求。

### 自定义指令 (Custom Directives)

你还可以编写自己的指令，根据需要自定义 Lit 的渲染行为。

## 独立模板 (Standalone Templating)

你还可以在 Lit 组件之外，单独使用 Lit 的模板库来进行独立的模板渲染。详情请参阅 [独立的 lit-html 模板](https://lit.dev/docs/templates/standalone-templates/)。
