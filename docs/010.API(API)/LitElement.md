# LitElement

> LitElement 是一个基类，用于管理元素的属性和属性同步，并渲染 lit-html 模板。

## Import

```javascript
import { LitElement } from 'lit';
```

> 用于定义组件，可以继承 LitElement 并实现 render 方法来提供组件的模板。使用 properties 属性或 @property 装饰器来定义属性。

## 属性（Attributes）


### attributeChangedCallback

> 当某个属性被设置时，系统会自动同步对应的属性值。通常不需要自己实现此回调。如果重写此方法，必须调用 super.attributeChangedCallback(name, _old, value)。更多信息请参阅 MDN 文档中的生命周期回调部分。

```javascript
// 用于在属性更改时同步属性值。
attributeChangedCallback(name, _old, value): void

// 参数
// name: string - 属性名称
// _old: null | string - 旧值
// value: null | string - 新值

```



### observedAttributes

> 返回一个对应已注册属性的属性列表。

```javascript
static observedAttributes: Array<string>

```

## 控制器（Controllers）

### addController

> 如果在元素已连接时调用 addController()，则会立即调用控制器的 hostConnected() 回调。

```javascript
addController(controller): void

// 参数
// controller: ReactiveController - 要注册的控制器
```

### removeController

> 从元素中移除 ReactiveController。

```javascript
removeController(controller): void

// 参数
// controller: ReactiveController - 要移除的控制器
```

## 开发模式（Dev mode）

### static disableWarning

> 禁用指定类别的警告。

```javascript
static disableWarning?: (warningKind: WarningKind) => void

// 禁用所有 ReactiveElement 子类的指定警告
ReactiveElement.disableWarning?.('migration');
// 仅禁用 MyElement 及其子类的指定警告
MyElement.disableWarning?.('migration');

```

### static enabledWarnings?: Array<WarningKind>

> 读取或设置此类中启用的所有警告类别。(**此属性仅用于开发版本**)

```javascript
static enabledWarnings?: Array<WarningKind>

// 启用所有 ReactiveElement 子类的指定警告
ReactiveElement.enableWarning?.('migration');
// 仅启用 MyElement 及其子类的指定警告
MyElement.enableWarning?.('migration');

```

## 生命周期（Lifecycle）

### 

> 当组件添加到文档的 DOM 中时调用。
> 在 connectedCallback() 中可以设置仅在元素连接到文档时才执行的任务，常见的任务是为元素外部的节点添加事件监听器，例如为 window 添加 keydown 事件处理器。

```javascript
// 定义
connectedCallback(): void

// 应用
connectedCallback() {
  super.connectedCallback();
  window.addEventListener('keydown', this._handleKeydown);
}
```

> 通常，connectedCallback() 中的所有操作应在 disconnectedCallback() 中撤销，以确保当元素从文档中移除时可以正确清理。

> 元素被断开连接后可能会重新连接。

## 其他功能（Other）

### static addInitializer(initializer): void

> 向类添加一个初始化函数，该函数会在实例构造期间被调用。
> 这个功能适用于对 ReactiveElement 子类执行操作的代码（例如装饰器），需要为每个实例执行的初始化工作，如设置 ReactiveController。

```javascript
// 定义
static addInitializer(initializer): void

// 参数
// initializer：Initializer - 初始化函数

// Demo

const myDecorator = (target: typeof ReactiveElement, key: string) => {
  target.addInitializer((instance: ReactiveElement) => {
    // 在元素的构造过程中运行此代码
    new MyController(instance);
  });
}

// 当字段被装饰时，每个实例都会运行一个初始化器，为该实例添加控制器：

class MyElement extends LitElement {
  @myDecorator foo;
}

// 初始化器是按构造函数存储的。向子类添加初始化器不会影响父类。
// 因为初始化器在构造函数中运行，它们会按照类层级的顺序执行，从父类到子类。

```

### static finalize()

> 为已注册的属性创建属性访问器，设置元素样式，并确保所有父类也完成初始化。如果元素已被初始化，则返回 true。

```javascript
static finalize(): boolean
```

### static finalized

> 确保此类被标记为已初始化，这是为了优化性能，避免不必要的重复初始化。

```javascript
static finalized: boolean

// 注意，这个属性名是一个字符串，以避免破坏 Closure JS 编译器的优化。更多信息请参阅 @lit/reactive-element。
```

## 属性（Properties）

### static createProperty

> 在元素原型上创建一个属性访问器（如果尚不存在），并使用指定的选项为属性存储一个 PropertyDeclaration。
> 此方法可以被重写以自定义属性行为，但在重写时务必调用 super.createProperty 以确保属性正确设置。该方法内部调用 getPropertyDescriptor 以获取描述符。如果需要自定义属性的 get 或 set 行为，可以重写 getPropertyDescriptor。

```javascript
// 定义
static createProperty(name, options?): void

// 参数
// name：PropertyKey - 属性名
// options（可选）：PropertyDeclaration<unknown, unknown> - 属性声明配置

// Demo: 
// 重写 getPropertyDescriptor。
static createProperty(name, options) {
  options = Object.assign(options, { myOption: true });
  super.createProperty(name, options);
}
```

### static elementProperties

> 存储所有元素属性的缓存列表，包括所有父类的属性。该列表会在用户子类初始化时创建。

```javascript
// 定义: 
static elementProperties: PropertyDeclarationMap
```

### static getPropertyDescriptor

> 返回一个描述符，用于定义指定属性。如果未返回描述符，则该属性不会成为访问器。

```javascript
static getPropertyDescriptor(name, key, options): undefined | PropertyDescriptor

// 参数
// name：PropertyKey - 属性名
// key：string | symbol - 属性键
// options：PropertyDeclaration<unknown, unknown> - 属性声明配置

// Demo:

class MyElement extends LitElement {
  static getPropertyDescriptor(name, key, options) {
    const defaultDescriptor = super.getPropertyDescriptor(name, key, options);
    const setter = defaultDescriptor.set;
    return {
      get: defaultDescriptor.get,
      set(value) {
        setter.call(this, value);
        // 自定义操作
      },
      configurable: true,
      enumerable: true
    }
  }
}
```

### static getPropertyOptions(name)

> 返回与指定属性相关联的选项。使用 properties 对象或 @property 装饰器定义的属性选项，通过 createProperty(...) 注册。
> 此方法应被视为“**最终方法**”，不应被重写。如需自定义属性选项，请重写 createProperty。

```javascript

// 定义:
static getPropertyOptions(name): PropertyDeclaration<unknown, unknown>

// 参数
// name：PropertyKey - 属性名

```

### static properties: PropertyDeclarations

> 用户提供的对象，映射属性名称到包含配置选项的 PropertyDeclaration 对象，用于配置响应式属性。当设置响应式属性时，元素将更新并重新渲染。
>
> 默认情况下，properties 是公共字段，因此应主要由元素用户设置，可以通过属性或属性自身来设置。通常，由元素更改的属性应为私有或受保护字段，且应使用 state: true 选项。标记为 state 的属性不会从相应的属性反映出来。但是，有时元素代码确实需要设置公共属性，这通常仅在响应用户交互时进行，并应触发事件告知用户；例如，复选框在点击时设置 checked 属性并触发 changed 事件。对于非基本类型（对象或数组）属性，通常不应更改公共属性。在需要管理状态的情况下，使用带有 state: true 选项的私有属性。当需要时，可以通过公共属性初始化状态属性，以促进复杂交互。

```javascript
// 定义: 
static properties: PropertyDeclarations

```


## 渲染（Rendering）

### createRenderRoot()

> 在调用 attachShadow 时创建渲染根节点，默认返回一个 ShadowRoot。

```javascript
// 定义:
createRenderRoot(): Element | ShadowRoot
```

### render()

> 在每次更新时调用，执行渲染任务。该方法通常返回 lit-html 的 ChildPart 可渲染的任何值，通常是一个 TemplateResult。在此方法中设置属性不会触发元素更新。

```javascript
// 定义: 
render(): unknown

```

### readonly renderOptions

> 渲染选项，为只读属性。

```javascript
// 定义
readonly renderOptions: RenderOptions
```

### readonly renderRoot

> 元素 DOM 应渲染的节点或 ShadowRoot。默认为一个打开的 shadowRoot。

```javascript
readonly renderRoot: HTMLElement | ShadowRoot
```

### static shadowRootOptions: ShadowRootInit

> 调用 attachShadow 时的选项。可以设置此属性来自定义 shadowRoot 的选项，例如创建一个关闭的 shadowRoot：{mode: 'closed'}。

```javascript
// 定义: 
static shadowRootOptions: ShadowRootInit
// 这些选项用于 createRenderRoot。如果自定义了该方法，则应尽可能尊重这些选项。
```

## 样式（Styles）

### static elementStyles: Array<CSSResultOrNative>

> 所有元素样式的缓存列表。该列表在用户子类初始化类时懒创建。

```javascript
// 定义: 
static elementStyles: Array<CSSResultOrNative>
```

### static finalizeStyles(styles?): Array<CSSResultOrNative>

> 将用户通过静态 styles 属性提供的样式转换为应用于元素的样式数组。可重写此方法以集成到样式管理系统中。
>
> 样式会进行去重，保留列表中的最后一个实例。这是一个性能优化，可以避免通过子类组合时产生的重复样式。保留最后一个项是为了尽量保持样式的层叠顺序，因为通常最后添加的样式优先级最高。

```javascript
// 定义:
static finalizeStyles(styles?): Array<CSSResultOrNative>

// 参数
// styles?: CSSResultGroup - 用户提供的样式组
```

### static styles?: CSSResultGroup

> 应用于元素的样式数组。应使用 css 标签函数、可构造样式表或从原生 CSS 模块脚本中导入的方式定义样式。

```javascript
// 定义: 
static styles?: CSSResultGroup
```

> 关于内容安全策略（CSP）：当浏览器不支持 adopted StyleSheets 时，元素样式会使用 \<style> 标签。要使用此类 \<style> 标签并满足 style-src CSP 指令要求，style-src 值必须包含 'unsafe-inline' 或 nonce-\<base64-value>（其中 \<base64-value> 是服务器生成的随机值）。若要在生成的 \<style> 元素上使用随机值，请在页面 HTML 中设置 window.litNonce 为服务器生成的随机值，确保在加载应用代码之前已设置。
> 
```html
<script>
  // 每次请求生成唯一的随机值
  window.litNonce = 'a1b2c3d4';
</script>
```

## 更新（Updates）

### enableUpdating(_requestedUpdate): void

> 此方法应视为“最终方法”，不应被重写。它会在元素实例上被覆盖，以触发首次更新。

```javascript
// 定义: 
enableUpdating(_requestedUpdate): void

// 参数
// _requestedUpdate: boolean
```

### firstUpdated(_changedProperties): void

> 当元素首次更新时调用。可以在该方法中实现一些仅需执行一次的工作。

```javascript
// 定义:
firstUpdated(_changedProperties): void

// 参数
// _changedProperties: Map<PropertyKey, unknown> | PropertyValueMap<any> - 包含更改属性及其旧值的映射

// 详情(在该方法中设置属性会在更新周期完成后再次触发更新。)
firstUpdated() {
  this.renderRoot.getElementById('my-text-area').focus();
}
```

### getUpdateComplete(): Promise<boolean>

> 更新完成的 Promise。
>
> 由于 TypeScript 的限制，不能直接重写 updateComplete getter。应重写 getUpdateComplete() 方法，而非直接重写 updateComplete。

```javascript
// 定义: 
getUpdateComplete(): Promise<boolean>

// Demo:
class MyElement extends LitElement {
  override async getUpdateComplete() {
    const result = await super.getUpdateComplete();
    await this._myChild.updateComplete;
    return result;
  }
}
```

### hasUpdated: boolean

> 首次更新后设置为 true。元素代码不能假设 renderRoot 在 hasUpdated 为 true 之前存在。

```javascript
// 定义:
hasUpdated: boolean
```

### isUpdatePending: boolean

> 当调用 requestUpdate() 产生待处理更新时为 true。此属性仅供读取。

```javascript
// 定义:
isUpdatePending: boolean
```

### performUpdate(): void | Promise<unknown>

> 执行元素更新。如果在更新期间抛出异常，firstUpdated 和 updated 将不会被调用。
> 
> 调用 performUpdate() 可立即处理待处理的更新。通常不需要这么做，但在需要同步更新的罕见情况下可以使用。注意：不要重写此方法以确保 performUpdate() 同步完成更新。在 LitElement 2.x 中建议通过重写 performUpdate() 来自定义更新调度，而现在推荐重写 scheduleUpdate()。
	
```javascript
// 定义
performUpdate(): void | Promise<unknown>
```

### requestUpdate(name?, oldValue?, options?): void

> 请求异步更新。通常在某些状态发生变化但未通过设置响应式属性触发时调用。在这种情况下，不需传递参数。若手动实现属性 setter，可以传递属性 name 和 oldValue 以确保任何配置的属性选项被正确处理。

```javascript
// 定义:
requestUpdate(name?, oldValue?, options?): void

// 参数
// name?: PropertyKey - 请求更新的属性名称
// oldValue?: unknown - 请求更新的属性旧值
// options?: PropertyDeclaration<unknown, unknown> - 用于更新的属性配置选项
```

### scheduleUpdate(): void | Promise<unknown>

> 调度元素更新。可以重写此方法以更改更新的时机，通过返回 Promise 来延迟更新。更新会等待 Promise 完成，在 Promise 解析后继续更新。如果重写了此方法，必须调用 super.scheduleUpdate()。

```javascript
// 定义:
scheduleUpdate(): void | Promise<unknown>

// Demo: 
// 将更新安排在下一帧之前：
override protected async scheduleUpdate(): Promise<unknown> {
  await new Promise((resolve) => requestAnimationFrame(() => resolve()));
  super.scheduleUpdate();
}

```

### shouldUpdate(_changedProperties): boolean

> 控制当元素请求更新时是否调用 update()。默认情况下，此方法总是返回 true，但可以自定义以控制何时更新。

```javascript
// 定义:
shouldUpdate(_changedProperties): boolean

// 参数
// _changedProperties: Map<PropertyKey, unknown> | PropertyValueMap<any> - 包含更改属性及其旧值的映射
```

### update(changedProperties): void

> 更新元素。此方法将属性值反映到特性中，并通过 lit-html 调用 render 来渲染 DOM。在此方法中设置属性不会再次触发更新。

```javascript
// 定义:
update(changedProperties): void

// 参数
// changedProperties: Map<PropertyKey, unknown> | PropertyValueMap<any> - 包含更改属性及其旧值的映射
```

### updateComplete: Promise<boolean>

> 返回一个 Promise，当元素完成更新时解析。Promise 值为 true 表示更新完成且未触发其他更新；为 false 表示 updated() 中设置了属性。如果 Promise 被拒绝，则表示更新期间抛出了异常。
>
> 若要等待其他异步操作完成，可重写 getUpdateComplete()。例如，有时需要等待某个渲染的元素以满足此 Promise。首先等待 super.getUpdateComplete()，然后再等待后续状态。

```javascript
// 定义:
updateComplete: Promise<boolean>
```

### updated(_changedProperties): void

> 每次元素更新完成后调用。可实现此方法来通过 DOM API 执行更新后任务，例如使元素获得焦点。

```javascript
// 定义:
updated(_changedProperties): void

// 参数
// _changedProperties: Map<PropertyKey, unknown> | PropertyValueMap<any> - 包含更改属性及其旧值的映射
// 详情
在此方法中设置属性将触发元素在当前更新周期完成后再次更新。
```

### updated(_changedProperties): void

> 在 update() 之前调用，用于计算更新过程中所需的值。

```javascript
// 定义
updated(_changedProperties): void

// 参数
// _changedProperties: Map<PropertyKey, unknown> | PropertyValueMap<any>

// Demo:
// 实现 willUpdate 用于计算依赖其他属性的属性值，并在更新过程的其余部分中使用这些值
willUpdate(changedProperties) {
  // 仅在需要时检查更改的属性以进行耗时的计算
  if (changedProperties.has('firstName') || changedProperties.has('lastName')) {
    this.sha = computeSHA(`${this.firstName} ${this.lastName}`);
  }
}

render() {
  return html`SHA: ${this.sha}`;
}
```

# type RenderOptions

> RenderOptions 是一个对象，用于指定控制 lit-html 渲染的选项。需要注意的是，尽管可以多次对相同的容器（以及 renderBefore 引用节点）调用 render 以高效更新渲染内容，但仅首次渲染时传入的选项会在后续渲染中生效。

## Import

```javascript
import { RenderOptions } from 'lit';
```

## 方法和属性

### creationScope?: { importNode: (node: Node, deep?: boolean) => Node }

> 用于克隆模板的节点（importNode 会在此节点上被调用）。此选项控制渲染 DOM 的 ownerDocument 以及任何继承的上下文。默认为全局的 document。

```javascript
// 定义:
creationScope?: { importNode: (node: Node, deep?: boolean) => Node }
```

### host?: object

> 用于事件监听器中 this 值的对象。通常将其设置为渲染模板的主组件。

```javascript
// 定义:
host?: object
```

### isConnected?: boolean

> 顶级渲染部分的初始连接状态。如果未设置 isConnected 选项，则 AsyncDirectives 默认会被连接。将其设置为 false 表示初次渲染发生在断开连接的 DOM 树中，且 AsyncDirectives 的初始渲染 isConnected 状态应为 false。在后续渲染中，可通过 part.setConnected() 方法更改连接状态。

```javascript
// 定义:
isConnected?: boolean
```

### renderBefore?: null | ChildNode

> 在容器中渲染内容之前的 DOM 节点。

```javascript
// 定义:
renderBefore?: null | ChildNode
```
