# 什么是 Lit?

Lit 是一个用于构建快速、轻量级 Web 组件的简单库。

Lit 的核心是一个消除样板的组件基类，它提供反应状态、范围样式以及小巧、快速且富有表现力的声明式模板系统。

## 我可以使用 Lit 构建什么？

你可以使用 Lit 构建几乎任何类型的 Web UI！

首先需要了解的是，每个 Lit 组件都是标准的 Web 组件。Web 组件具有强大的互操作性：由于浏览器原生支持，Web 组件可以在任何 HTML 环境中使用，无论是否使用框架。

这使得 Lit 成为开发可共享组件或设计系统的理想选择。Lit 组件可以跨多个应用和网站使用，即使这些应用和网站基于不同的前端技术栈。使用 Lit 组件的网站开发者无需编写或接触任何 Lit 代码，他们只需像使用内置 HTML 元素一样使用这些组件即可。

Lit 还非常适合对基础 HTML 站点进行渐进式增强。无论你的网站是手工编写的、由 CMS 管理的、使用服务端框架构建的，还是通过 Jekyll 或 eleventy 等工具生成的，浏览器都会识别你标记中的 Lit 组件，并自动进行初始化。

当然，你也可以使用 Lit 组件构建高度交互、功能丰富的应用程序，就像使用 React 或 Vue 框架一样。Lit 的功能和开发体验可与这些流行的替代方案相媲美，但它通过采用浏览器的原生组件模型，最大程度地减少了锁定效应，提升了灵活性，并促进了代码的可维护性。

当你使用 Lit 构建应用时，可以轻松地混合使用“纯”Web 组件或其他库构建的 Web 组件。甚至可以逐个组件地更新到 Lit 的新版本，或迁移到其他库，而不会影响产品开发进度。

## 使用 Lit 开发是什么体验？

如果你有过现代化、基于组件的 Web 开发经验，那么你应该会对 Lit 感到非常熟悉。即使你之前没有使用组件进行过开发，我们也相信你会发现 Lit 非常易于上手。

每个 Lit 组件都是一个自包含的 UI 单元，由更小的构建模块（标准 HTML 元素和其他 Web 组件）组合而成。反过来，每个 Lit 组件本身也可以作为构建模块，用于 HTML 文档、其他 Web 组件或框架组件中，以构建更大、更复杂的界面。

以下是一个简单但不那么平凡的组件（一个倒计时计时器），它展示了 Lit 代码的样子，并突出了几个关键特性：

```typescript
import {LitElement, html, css} from 'lit';
import {customElement, property, state} from 'lit/decorators.js';
/* playground-fold */
import {play, pause, replay} from './icons.js';
/* playground-fold-end */

@customElement("my-timer")
export class MyTimer extends LitElement {
  static styles = css`/* playground-fold */

    :host {
      display: inline-block;
      min-width: 4em;
      text-align: center;
      padding: 0.2em;
      margin: 0.2em 0.1em;
    }
    footer {
      user-select: none;
      font-size: 0.6em;
    }
    /* playground-fold-end */`;

  @property() duration = 60;
  @state() private end: number | null = null;
  @state() private remaining = 0;

  render() {
    const {remaining, running} = this;
    const min = Math.floor(remaining / 60000);
    const sec = pad(min, Math.floor(remaining / 1000 % 60));
    const hun = pad(true, Math.floor(remaining % 1000 / 10));
    return html`
      ${min ? `${min}:${sec}` : `${sec}.${hun}`}
      <footer>
        ${remaining === 0 ? '' : running ?
          html`<span @click=${this.pause}>${pause}</span>` :
          html`<span @click=${this.start}>${play}</span>`}
        <span @click=${this.reset}>${replay}</span>
      </footer>
    `;
  }
  /* playground-fold */

  start() {
    this.end = Date.now() + this.remaining;
    this.tick();
  }

  pause() {
    this.end = null;
  }

  reset() {
    const running = this.running;
    this.remaining = this.duration * 1000;
    this.end = running ? Date.now() + this.remaining : null;
  }

  tick() {
    if (this.running) {
      this.remaining = Math.max(0, this.end! - Date.now());
      requestAnimationFrame(() => this.tick());
    }
  }

  get running() {
    return this.end && this.remaining;
  }

  connectedCallback() {
    super.connectedCallback();
    this.reset();
  }/* playground-fold-end */

}
/* playground-fold */

function pad(pad: unknown, val: number) {
  return pad ? String(val).padStart(2, '0') : val;
}/* playground-fold-end */

```

```html
<!doctype html>
<head><!-- playground-fold -->
  <link rel="preconnect" href="https://fonts.gstatic.com">
  <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:ital,wght@1,800&display=swap" rel="stylesheet">
  <script type="module" src="./my-timer.js"></script>
  <style>
    body {
      font-family: 'JetBrains Mono', monospace;
      font-size: 36px;
    }
  </style>
  <!-- playground-fold-end -->
</head>
<body>
  <my-timer duration="7"></my-timer>
  <my-timer duration="60"></my-timer>
  <my-timer duration="300"></my-timer>
</body>
```

```typescript
import {html} from 'lit';

export const replay = html`<svg xmlns="http://www.w3.org/2000/svg" enable-background="new 0 0 24 24" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><title>Replay</title><g><rect fill="none" height="24" width="24"/><rect fill="none" height="24" width="24"/><rect fill="none" height="24" width="24"/></g><g><g/><path d="M12,5V1L7,6l5,5V7c3.31,0,6,2.69,6,6s-2.69,6-6,6s-6-2.69-6-6H4c0,4.42,3.58,8,8,8s8-3.58,8-8S16.42,5,12,5z"/></g></svg>`;
export const pause = html`<svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><title>Pause</title><path d="M0 0h24v24H0V0z" fill="none"/><path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z"/></svg>`;
export const play = html`<svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 0 24 24" width="24px" fill="#000000"><title>Play</title><path d="M0 0h24v24H0V0z" fill="none"/><path d="M10 8.64L15.27 12 10 15.36V8.64M8 5v14l11-7L8 5z"/></svg>`;

```

**需要注意的几点：**

- Lit 的主要特性是 `LitElement` 基类，它是原生 `HTMLElement` 的一个便捷且功能多样的扩展。你可以通过继承它来定义自己的组件。
- Lit 提供了简洁明了的声明式模板（使用 JavaScript 的标签模板字面量），使得描述组件如何渲染变得轻而易举。
- 响应式属性代表了组件的公共 API 和/或内部状态；当响应式属性发生变化时，组件会自动重新渲染。
- 样式默认是作用域限定的，这样可以保持 CSS 选择器的简洁，并确保组件的样式不会污染（或被周围环境污染）。
- Lit 可以很好地与原生 JavaScript 一起工作，或者你可以使用 TypeScript 进行开发，通过使用装饰器和类型声明来提升开发体验。
- 在开发过程中，Lit 不需要编译或构建，因此如果你喜欢，可以几乎不使用任何工具进行开发。同时，Lit 提供了完善的 IDE 支持（代码补全、代码检查等）以及生产环境工具（如本地化、模板压缩等）。

## 为什么我要选择 Lit？

正如我们之前提到的，Lit 非常适合用于构建各种 Web UI，它将 Web 组件的互操作性优势与现代、舒适的开发体验相结合。

除此之外，Lit 还具备以下特点：

- **简单易用**。基于 Web 组件标准，Lit 仅增加了让你在开发中感到愉快和高效的必要功能：响应式机制、声明式模板，以及一系列精心设计的特性来减少样板代码，让开发工作更加轻松。
- **高效**。由于 Lit 只跟踪 UI 中动态部分的变化，并在底层状态改变时仅更新这些部分，因此更新速度非常快——无需重建整个虚拟树并与当前 DOM 状态进行比对。
- **轻量级**。经过压缩和精简后，Lit 的体积约为 5 KB，有助于保持打包体积的小巧和加载时间的短暂。
  
Lit 背后的团队自 Web 组件诞生之初便参与其中。我们帮助 Google 维护了数以万计的组件，提供了一套全面的 Web 组件 Polyfill，并积极参与标准和社区的建设。

Lit 的每个特性都是在充分考虑 Web 平台发展方向的前提下精心设计的；我们旨在帮助你充分利用当前平台所提供的功能，同时编写能够从未来平台增强中受益的代码。

## 接下来的步骤

**入门指南：**设置开发环境，开始使用 Lit 进行开发。

**组件：**了解 Lit 组件模型。

**模板：**使用 lit-html 语法编写模板。

**代码组织：**编写可复用、易维护的代码。