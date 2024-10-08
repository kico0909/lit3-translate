### 运行时本地化模式

在 Lit Localize 的运行时模式中，每个语言环境都会生成一个 JavaScript 或 TypeScript 模块。每个生成的模块包含该语言环境的本地化模板。当应用切换语言环境时，会自动导入对应的模块，并重新渲染所有本地化组件。

#### 示例输出

```typescript
// locales/es-419.ts
export const templates = {
  h3c44aff2d5f5ef6b: html`Hola <b>Mundo!</b>`,
};
```

#### 使用运行时模式的示例

以下是一个使用运行时模式的应用程序示例：

```typescript
import {LitElement, html} from 'lit';
import {customElement} from 'lit/decorators.js';
import {msg, localized} from '@lit/localize';

@localized()
@customElement('x-greeter')
export class XGreeter extends LitElement {
  render() {
    return html`<p>${msg(html`Hello <b>World</b>!`)}</p>`;
  }
}
```

在 Lit 的 GitHub 仓库中提供了完整的 Lit Localize 运行时模式示例（JavaScript 和 TypeScript），可用作模板。

### 配置运行时模式

1. 在 `lit-localize.json` 中设置 `output.mode` 为 `runtime`，并设置 `output.outputDir` 为生成本地化模板模块的目标位置。
2. 配置 `output.localeCodesModule` 为模块的文件路径。生成的模块会导出以下内容：

```typescript
export const sourceLocale = 'en';
export const targetLocales = ['es-419', 'zh-Hans'];
export const allLocales = ['en', 'es-419', 'zh-Hans'];
```

3. 在项目中调用 `configureLocalization`，并传递以下属性：
    - `sourceLocale`: `output.localeCodesModule` 导出的 `sourceLocale` 变量。
    - `targetLocales`: `output.localeCodesModule` 导出的 `targetLocales` 变量。
    - `loadLocale`: 一个用于加载本地化模板的函数。该函数返回一个 Promise，用于解析特定语言环境的模板模块。

配置示例：

```typescript
import {configureLocalization} from '@lit/localize';
import {sourceLocale, targetLocales} from './generated/locale-codes.js';

export const {getLocale, setLocale} = configureLocalization({
  sourceLocale,
  targetLocales,
  loadLocale: (locale) => import(`/locales/${locale}.js`),
});
```

### 自动重新渲染

要在语言环境切换时自动重新渲染组件，可以在 JavaScript 中使用 `updateWhenLocaleChanges` 函数，或者在 TypeScript 中对类应用 `@localized` 装饰器。

```typescript
import {LitElement, html} from 'lit';
import {customElement} from 'lit/decorators.js';
import {msg, localized} from '@lit/localize';

@customElement('my-element')
@localized()
class MyElement extends LitElement {
  render() {
    return msg(html`Hello <b>World!</b>`);
  }
}
```

### 状态事件

`lit-localize-status` 事件在语言环境切换开始、完成或失败时会触发。你可以使用此事件来执行以下操作：

- 当无法使用 `@localized` 装饰器时重新渲染。
- 当语言环境切换开始时立即渲染（例如显示加载指示器）。
- 执行其他本地化相关操作（例如设置语言环境偏好 cookie）。

事件状态包括：

- `loading`: 新语言环境开始加载。
- `ready`: 新语言环境加载成功，已准备就绪。
- `error`: 新语言环境加载失败。

### 加载语言环境模块的方式

- **懒加载（Lazy-load）**：使用动态导入，仅在激活语言环境时加载。
- **预加载（Pre-load）**：在页面加载时预加载所有语言环境模块。
- **静态导入（Static imports）**：使用静态导入预加载所有语言环境（不推荐）。