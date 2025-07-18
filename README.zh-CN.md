# @rsbuild/plugin-assets-retry

用于在静态资源加载失败时自动发起重试请求。

<p>
  <a href="https://npmjs.com/package/@rsbuild/plugin-assets-retry">
   <img src="https://img.shields.io/npm/v/@rsbuild/plugin-assets-retry?style=flat-square&colorA=564341&colorB=EDED91" alt="npm version" />
  </a>
  <img src="https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square&colorA=564341&colorB=EDED91" alt="license" />
  <a href="https://npmcharts.com/compare/@rsbuild/plugin-assets-retry?minimal=true"><img src="https://img.shields.io/npm/dm/@rsbuild/plugin-assets-retry.svg?style=flat-square&colorA=564341&colorB=EDED91" alt="downloads" /></a>
</p>

## 快速开始

### 安装插件

你可以通过如下的命令安装插件:

```bash
# npm
npm add @rsbuild/plugin-assets-retry -D

# yarn
yarn add @rsbuild/plugin-assets-retry -D

# pnpm
pnpm add @rsbuild/plugin-assets-retry -D

# bun
bun add @rsbuild/plugin-assets-retry -D
```

### 注册插件

你可以在 `rsbuild.config.ts` 文件中注册插件：

```ts
import { pluginAssetsRetry } from '@rsbuild/plugin-assets-retry';

export default {
  plugins: [pluginAssetsRetry()],
};
```

## 选项

你可以通过选项来配置资源加载失败时的重试逻辑。

- **类型：**

```ts
type AssetsRetryHookContext = {
  times: number;
  domain: string;
  url: string;
  tagName: string;
  isAsyncChunk: boolean;
};

type RuntimeRetryOptions = {
  type?: string[];
  domain?: string[];
  max?: number;
  test?: string | ((url: string) => boolean);
  crossOrigin?: boolean | 'anonymous' | 'use-credentials';
  delay?: number | ((context: AssetsRetryHookContext) => number);
  onRetry?: (context: AssetsRetryHookContext) => void;
  onSuccess?: (context: AssetsRetryHookContext) => void;
  onFail?: (context: AssetsRetryHookContext) => void;
};

type AssetsRetryOptions =
  | ({
      inlineScript?: boolean;
      minify?: boolean;
    } & RuntimeRetryOptions)
  | {
      inlineScript?: boolean;
      minify?: boolean;
      rules: RuntimeRetryOptions[];
    };
```

- **默认值：**

```ts
const defaultAssetsRetryOptions = {
  max: 3,
  type: ['script', 'link', 'img'],
  domain: [],
  crossOrigin: false,
  test: '',
  delay: 0,
  addQuery: false,
  inlineScript: true,
  minify: rsbuildConfig.mode === 'production',
};
```

### domain

- **类型：** `string[]`
- **默认值：** `[]`

指定资源加载失败时的重试域名列表。在 `domain` 数组中，第一项是静态资源默认所在的域名，后面几项为备用域名。当某个域名的资源请求失败时，Rsbuild 会在数组中找到该域名，并替换为数组的下一个域名。

比如：

```js
// rsbuild.config.ts
defineConfig({
  plugins: [
    pluginAssetsRetry({
      domain: ['cdn1.com', 'cdn2.com', 'cdn3.com'],
    }),
  ],
  output: {
    assetPrefix: 'https://cdn1.com', // 或者 "//cdn1.com"
  },
});
```

添加以上配置后，当 `cdn1.com` 域名的资源加载失败时，请求域名会自动降级到 `cdn2.com`。

如果 `cdn2.com` 的资源也请求失败，则会继续请求 `cdn3.com`。

### type

- **类型：** `string[]`
- **默认值：** `['script', 'link', 'img']`

用于指定需要进行重试的 HTML 标签类型。默认会处理 script 标签、link 标签和 img 标签，对应 JS 代码、CSS 代码和图片。

比如只对 script 标签和 link 标签进行处理：

```js
pluginAssetsRetry({
  type: ['script', 'link'],
});
```

### max

- **类型：** `number`
- **默认值：** `3`

单个资源的最大重试次数。比如：

```js
pluginAssetsRetry({
  max: 5,
});
```

### test

- **类型：** `string | ((url: string) => boolean) | undefined`
- **默认值：** `undefined`

匹配资源 URL 的正则表达式或函数，默认匹配所有资源。比如：

```js
pluginAssetsRetry({
  test: /cdn\.example\.com/,
});
```

### crossOrigin

- **类型：** `undefined | boolean | 'anonymous' | 'use-credentials'`
- **默认值：** `与 html.crossorigin 一致`

在发起资源重新请求时，Rsbuild 会重新创建 `<script>` 标签，此选项可以设置这些标签的 `crossorigin` 属性。

默认情况下，`crossOrigin` 的值会与 `html.crossorigin` 配置项保持一致，无须额外配置。如果你需要对重新创建的标签进行单独配置，可以使用该选项，比如：

```js
pluginAssetsRetry({
  crossOrigin: true,
});
```

### onRetry

- **类型：** `undefined | (context: AssetsRetryHookContext) => void`

资源重试时的回调函数。比如：

```js
pluginAssetsRetry({
  onRetry: ({ times, domain, url, tagName, isAsyncChunk }) => {
    console.log(
      `Retry ${times} times, domain: ${domain}, url: ${url}, tagName: ${tagName}, isAsyncChunk: ${isAsyncChunk}`,
    );
  },
});
```

### onSuccess

- **类型：** `undefined | (context: AssetsRetryHookContext) => void`

资源重试成功时的回调函数。比如：

```js
pluginAssetsRetry({
  onSuccess: ({ times, domain, url, tagName, isAsyncChunk }) => {
    console.log(
      `Retry ${times} times, domain: ${domain}, url: ${url}, tagName: ${tagName}, isAsyncChunk: ${isAsyncChunk}`,
    );
  },
});
```

### onFail

- **类型：** `undefined | (context: AssetsRetryHookContext) => void`

资源重试超过最大重试次数时的回调函数。比如：

```js
pluginAssetsRetry({
  onFail: ({ times, domain, url, tagName, isAsyncChunk }) => {
    console.log(
      `Retry ${times} times, domain: ${domain}, url: ${url}, tagName: ${tagName}, isAsyncChunk: ${isAsyncChunk}`,
    );
  },
});
```

### addQuery

- **类型：**

```ts
type AddQuery =
  | boolean
  | ((context: { times: number; originalQuery: string }) => string);
```

- **默认值：** `false`

是否在资源重试时添加 query，这样可以避免被浏览器、CDN 缓存影响到重试的结果。

当设置为 `true` 时，请求时会在 query 中添加 `retry=${times}`，按照 `retry=1`，`retry=2`，`retry=3` 依次请求，比如：

1. 假设请求的静态资源为 `https://js.cdn.net/foo.js`，请求失败后会自动重试 `https://js.cdn.net/foo.js?retry=${times}`

2. 假设请求的静态资源为 `https://js.cdn.net/foo.js?version=1`，请求失败后会自动重试 `https://js.cdn.net/foo.js?version=1&retry=${times}`

当你想要自定义 query 时，可以传入一个函数，比如：

- **示例 1：** 请求的所有资源都不含 query：

```js
pluginAssetsRetry({
  addQuery: ({ times }) => {
    return times === 3
      ? `?retryCount=${times}&isLast=1`
      : `?retryCount=${times}`;
  },
});
```

- **示例 2：** 当请求的某些资源中含有 query 时，可以使用 `originalQuery` 读取：

```js
pluginAssetsRetry({
  addQuery: ({ times, originalQuery }) => {
    const query =
      times === 3 ? `retryCount=${times}&isLast=1` : `retryCount=${times}`;
    return originalQuery ? `${originalQuery}&${query}` : `?${query}`;
  },
});
```

### inlineScript

- **类型：** `boolean`
- **默认值：** `true`

是否将 Assets Retry 插件的运行时 JavaScript 代码内联到 HTML 文件中。

如果你不希望在 HTML 文件中插入相关代码，可以将 `inlineScript` 设置为 `false`：

```js
pluginAssetsRetry({
  inlineScript: false,
});
```

添加以上配置后，Assets Retry 插件的运行时代码会被抽取为一个独立的 `assets-retry.[version].js` 文件，并输出到产物目录下。

这种方式的弊端在于，`assets-retry.[version].js` 自身有加载失败的可能性。如果出现这种情况，静态资源重试的逻辑就无法生效。因此，我们更推荐将运行时代码内联到 HTML 文件中。

### minify

- **类型：** `boolean`
- **默认值：** `process.env.NODE_ENV === 'production'`

是否对运行时 JavaScript 代码开启代码压缩。

默认情况下，会受到 [output.minify](/config/output/minify) 配置的影响。

```js
pluginAssetsRetry({
  minify: true,
});
```

### delay

- **类型：** `number | ((context: AssetsRetryHookContext) => number)`
- **默认值：** `0`

在资源重试前的延迟时间，单位为毫秒。

你可以传入一个数字：

```js
// 延时 1s 重试
pluginAssetsRetry({
  delay: 1000,
});
```

也可以传入一个函数，函数的参数为 `AssetsRetryHookContext`，返回值为延迟时间：

```js
// 通过次数来计算延迟时间
pluginAssetsRetry({
  delay: ctx => (ctx.times + 1) * 1000,
});
```

### rules

- **类型：** `RuntimeRetryOptions[]`
- **默认值：** `undefined`

配置多个重试规则，每个规则可以有不同的选项。规则会按顺序进行评估，第一个匹配的规则将用于重试逻辑。这在你对不同类型的资源或域名有不同的重试需求时非常有用。

使用 `rules` 时，插件会：

1. 按顺序通过 `test` `domain` `type` 检查每个规则

2. 如果匹配到规则，会使用规则的配置进行重试

3. 如果没有匹配到规则，则不会重试该资源

每个规则支持与顶层配置相同的所有选项，包括 `type`、`domain`、`test`、`max`、`crossOrigin`、`delay`、`onRetry`、`onSuccess` 和 `onFail`。

- **示例 1：** 不同 CDN 的不同重试策略：

```js
pluginAssetsRetry({
  rules: [
    {
      // 主 CDN 的规则
      test: /cdn1\.example\.com/,
      domain: ['cdn1.example.com', 'cdn1-backup.example.com'],
      max: 3,
      delay: 1000,
    },
    {
      // 次要 CDN 的规则，更多重试次数
      test: /cdn2\.example\.com/,
      domain: ['cdn2.example.com', 'cdn2-backup.example.com'],
      max: 5,
      delay: 500,
    },
    {
      // 其他资源的默认规则
      domain: ['default.example.com', 'default-backup.example.com'],
      max: 2,
    },
  ],
});
```

- **示例 2：** 不同资源类型的不同重试策略：

```js
pluginAssetsRetry({
  rules: [
    {
      // 关键 JavaScript 文件获得更多重试次数
      test: /\.js$/,
      // 或者 type: ['script'],
      max: 5,
      delay: 1000,
      onFail: ({ url }) => console.error(`关键 JS 失败: ${url}`),
    },
    {
      // CSS 文件获得较少的重试次数
      test: /\.css$/,
      max: 2,
      delay: 500,
    },
    {
      // 图片获得最少的重试次数
      test: /\.(png|jpg|gif|svg)$/,
      max: 1,
      delay: 0,
    },
  ],
});
```

## 注意事项

当你使用 Assets Retry 插件时，Rsbuild 会分别向 HTML 和 [Rspack Runtime](https://rspack.dev/zh/misc/glossary#runtime) 中注入运行时代码，并将 Assets Retry 插件配置的内容序列化后插入到这些代码中，因此你需要注意：

- 避免在 Assets Retry 插件中配置敏感信息，比如内部使用的 token。
- 避免在 `onRetry`，`onSuccess`，`onFail` 中引用函数外部的变量或方法。
- 避免在 `onRetry`，`onSuccess`，`onFail` 中使用有兼容性问题的语法，因为这些函数会被直接内联到 HTML 中。

以下是一个错误示例：

```js
import { someMethod } from 'utils';

pluginAssetsRetry({
  onRetry() {
    // 错误用法，包含了敏感信息
    const privateToken = 'a-private-token';

    // 错误用法，使用了外部的方法
    someMethod(privateToken);
  },
});
```

## 使用限制

以下场景 Assets Retry 插件可能无法生效：

### 同步 script 标签加载的资源

`<script src="..."></script>` 标签加载的资源是同步加载的，如果进行重试无法保证资源加载的顺序，因此 Assets Retry 插件不会对同步加载的 script 标签进行重试。只会对 async/defer 的 script 标签进行重试。

### 模块联邦

对于模块联邦加载的远程模块，你可以使用模块联邦 2.0 的 [@module-federation/retry-plugin](https://www.npmjs.com/package/@module-federation/retry-plugin) 来实现静态资源重试。

### 微前端

如果你的工程是微前端应用（比如 Garfish 子应用），那么 Assets Retry 插件可能无法生效，因为微前端子应用通常不是基于 `<script>` 标签直接加载的。

如果你需要对微前端场景的资源加载进行重试，请联系微前端框架的开发者，以寻找相应的解决方案。

### 自定义模版中的资源

Assets Retry 插件通过监听页面 error 事件来获悉当前资源是否加载失败需要重试。因此，如果自定义模版中的资源执行早于 Assets Retry 插件，那 Assets Retry 插件无法监听到该资源加载失败的事件，retry 无法对其生效。

如果想要 Assets Retry 插件对自定义模版中的资源生效，可参考 [自定义插入示例](https://github.com/rspack-contrib/html-rspack-plugin/tree/main/examples/custom-insertion-position) 来修改 [html.inject](/config/html/inject) 配置和自定义模版。

```diff
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>custom template</title>
+   <%= htmlPlugin.tags.headTags %>
    <script src="https://example.com/assets/a.js"></script>
  </head>
  <body>
    <div id="root" />
+    <%= htmlPlugin.tags.bodyTags %>
  </body>
</html>
```

## License

[MIT](./LICENSE).
