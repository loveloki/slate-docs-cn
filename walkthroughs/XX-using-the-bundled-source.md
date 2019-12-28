# 使用捆绑源

对于大多数人来说，你会希望通过 `npm` 安装 Slate，在这种情况下，你可以遵循常规的 [安装指南](./installing-slate.md) 。

但是，如果你更愿意简单地在你的应用程序中添加一个 `<script>` 标签来安装 Slate，那么本指南将对你有所帮助。为了使"捆绑"的情况下使用更加简单，Slate的每一个版本都附带了一个名为 `slate.js` 的文件。

要获得 `slate.js` 的副本，可以从 npm 下载你想要的指定版本：

```
npm install slate@latest
```

然后在 `node_modules` 文件夹里获取到 `slate.js` 文件：

```
node_modules/
  slate/
    dist/
      slate.js
      slate.min.js
```

为了方便起见，还包含了一个名为 `slate.min.js` 的压缩版本。

在你添加 `slate.js` 到页面之前，你需要自己提供 `react` ， `react-dom` 和 `react-dom-server` 文件，像这样：

```html
<script src="./vendor/react.js"></script>
<script src="./vendor/react-dom.js"></script>
<script src="./vendor/react-dom-server.js"></script>
```

这会确保 Slate 不会绑定自己的 Immutable 和 React 版本，这样（如果捆绑）会大大增加你的应用程序的大小。

然后你可以添加 `slate.js` 在以上代码之后：

```html
<script src="./vendor/slate.js"></script>
```

为了让事情更加简单，对于原型制作，你也可以使用 [`unpkg.com`](https://unpkg.com/#/) 分发网络，这使得与捆绑的 npm modules 一起使用更加方便。在这种情况下，你应该像下面这样：

```html
<script src="https://unpkg.com/react/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/react-dom/umd/react-dom-server.browser.production.min.js"></script>
<script src="https://unpkg.com/slate/dist/slate.js"></script>
<script src="https://unpkg.com/slate-react/dist/slate-react.js"></script>
```

就是这样，你已经准备好一切了！
