
# 响应式属性

Lit 组件通过 JavaScript 类字段或属性接收输入，并存储其状态。响应式属性是指当属性发生更改时，可以触发响应式更新周期并重新渲染组件的属性。此外，它们还可以被读取或写入到 HTML 属性（attribute）。

```typescript
class MyElement extends LitElement {
  @property()
  name?: string;
}
```

Lit 会管理响应式属性及其对应的 HTML 属性。具体如下：

- **响应式更新**：Lit 会为每个响应式属性生成一个 getter/setter 对。当响应式属性更改时，组件会安排更新。
- **属性处理**：默认情况下，Lit 会为响应式属性设置相应的观察属性（observed attribute），并在属性发生更改时更新属性值。属性值还可以选择性地反映回 HTML 属性。
- **父类属性**：Lit 会自动应用由父类声明的属性选项。除非需要更改选项，否则不需要重新声明这些属性。
- **元素升级**：如果 Lit 组件在元素已经存在于 DOM 中后才定义，Lit 会处理升级逻辑，确保在元素升级时，任何已设置的属性都会触发正确的响应式副作用。

## 公共属性和内部状态

公共属性是组件公共 API 的一部分。通常，公共属性（特别是公共响应式属性）应被视为组件的输入。除非响应用户输入，组件不应改变自身的公共属性。

例如，一个菜单组件可能有一个 `selected` 公共属性，该属性可以由元素的所有者初始化为特定值，但在用户选择某个选项时，该属性会被组件自身更新。这种情况下，组件应触发事件，向组件的所有者表明 `selected` 属性已更改。详见 [事件派发](https://lit.dev/docs/components/events/)。

Lit 还支持内部响应式状态。内部响应式状态是指不属于组件 API 的响应式属性。这些属性没有对应的 HTML 属性，并且在 TypeScript 中通常标记为 `protected` 或 `private`。

```typescript
@state()
private _counter = 0;
```

组件可以自行操作内部响应式状态。在某些情况下，内部响应式状态可能从公共属性中初始化，例如，当用户可见的属性与内部状态之间有昂贵的转换时。

与公共响应式属性一样，更新内部响应式状态也会触发更新周期。更多信息请参阅[内部响应式状态](https://lit.dev/docs/components/properties/#internal-reactive-state)。

## 声明公共响应式属性

可以使用装饰器或静态 `properties` 字段来声明元素的公共响应式属性。

### 使用装饰器声明属性

使用 `@property` 装饰器和类字段声明来声明响应式属性。

```typescript
class MyElement extends LitElement {
  @property({type: String})
  mode?: string;

  @property({attribute: false})
  data = {};
}
```

传递给 `@property` 装饰器的参数是一个选项对象。如果省略该参数，则相当于为所有选项指定默认值。

### 在静态 `properties` 字段中声明属性

在静态 `properties` 字段中声明属性：

```typescript
class MyElement extends LitElement {
  static properties = {
    mode: {type: String},
    data: {attribute: false},
  };

  constructor() {
    super();
    this.data = {};
  }
}
```

空的选项对象等同于为所有选项指定默认值。

## 避免在声明属性时与类字段发生冲突

类字段与响应式属性之间可能存在冲突。类字段在元素实例上定义，而响应式属性则作为访问器定义在元素原型上。根据 JavaScript 规则，实例属性会优先于原型属性，因此使用类字段时响应式属性访问器将无法生效，从而导致设置属性时无法触发元素更新。

```typescript
class MyElement extends LitElement {
  static properties = {foo: {type: String}}
  foo = 'Default'; // ❌ 这会导致 `foo` 无法成为响应式属性
}
```

在 JavaScript 中，不应在声明响应式属性时使用类字段。相反，应在元素构造函数中初始化属性：

```typescript
class MyElement extends LitElement {
  static properties = {foo: {type: String}}
  constructor() {
    super();
    this.foo = 'Default';
  }
}
```

### 在 TypeScript 中使用类字段

在 TypeScript 中，只要使用以下模式之一，就可以在声明响应式属性时使用类字段：

- 将 `useDefineForClassFields` 编译器选项设置为 `false`。在 TypeScript 中使用装饰器时，推荐使用该选项。

```json
// tsconfig.json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "useDefineForClassFields": false,
  }
}
```

```typescript
class MyElement extends LitElement {
  static properties = {foo: {type: String}}
  foo = 'Default';

  @property()
  bar = 'Default';
}
```

- 在字段上添加 `declare` 关键字，并在构造函数中初始化字段。

```typescript
class MyElement extends LitElement {
  declare foo: string;
  static properties = {foo: {type: String}}
  constructor() {
    super();
    this.foo = 'Default';
  }
}
```

- 使用 `accessor` 关键字在字段中使用自动访问器。

```typescript
class MyElement extends LitElement {
  static properties = {foo: {type: String}}
  accessor foo = 'Default';

  @property()
  accessor bar = 'Default';
}
```

## 属性选项

选项对象可以包含以下属性：

- `attribute`：属性是否与 HTML 属性关联，或是自定义关联的属性名。默认值：`true`。如果 `attribute` 为 `false`，则忽略 `converter`、`reflect` 和 `type` 选项。
- `converter`：用于在属性和 HTML 属性之间进行转换的自定义转换器。
- `hasChanged`：在属性设置时调用的函数，用于判断属性是否发生变化，并决定是否应触发更新。未指定时，`LitElement` 使用严格不等式 `newValue !== oldValue` 来判断属性是否更改。
- `noAccessor`：设置为 `true` 时，不生成默认的属性访问器。通常不需要使用该选项。
- `reflect`：属性值是否反映回关联的 HTML 属性。默认值：`false`。
- `state`：设置为 `true` 时，将属性声明为内部响应式状态。内部响应式状态触发更新，但不会生成 HTML 属性，且不应从组件外部访问该属性。
- `type`：在将字符串值的 HTML 属性转换为属性时，默认的属性转换器会将字符串解析为给定的类型。更多信息请参阅[使用默认转换器](https://lit.dev/docs/components/properties/#using-the-default-converter)。

