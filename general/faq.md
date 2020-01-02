# 常见问题

一些关于 Slate 的常见问题是：

- [为什么将内容粘贴为纯文本？](#why-is-content-is-pasted-as-plaintext)
- [ `Block` 节点可以拥有什么样子子节点？](#what-can-a-block-node-have-as-its-children)
- [Slate 支持哪些浏览器和设备？](#what-browsers-and-devices-does-slate-support)

### 为什么将内容粘贴为纯文本？

Slate 核心的原则之一就是这个，不像其它的编辑器，它 **没有** 规定你用哪种特殊的 "schema" 来编辑内容。这意味着 Slate 核心没有块引用和加粗格式的概念。

最重要的一点是，这样增加了扩展性（而没有其他副作用）,但是有一些情况下你不得不做更多的工作。粘贴就是这些情况之一。

因为 Slate 对于你的域一无所知，它不知道如何解析被粘贴的 HTML 内容（或者其它内容）。所以默认情况下无论用户粘贴什么内容到 Slate 编辑器，它都会解析为纯文本。如果你想要它可以更智能地解析粘贴内容，你需要按照你的需求去重写  `insert_data` 命令和 反序列化  `DataTransfer` 对象的 `text/html` 数据。

### Slate 支持什么浏览器和设备？

Slate 的目标是支持所有桌面和移动设备的现代浏览器。

然而，现在 Slate 是测试版并且是社区驱动的，所以它并做不到理想的支持。目前已经对桌面上最新版本的Chrome，Edge，Firefox 和 Safari 进行了测试。并且它无法在 Internet Explorer 中使用。对于移动设备，iOS 已被支持但没有进行定期测试。安卓 上面的 Chrome 支持 Slate 0.47 而不是最近发布的 Slate 0.50+。如果你想要添加更多的浏览器或设备支持，我们欢迎你提交拉取请求（pull request）！或者对不兼容的浏览器，构建一个插件。

对于更老的浏览器，比如 IE11，大量现代标准的原生 API 是不能用的。Slate 的看法是：在这种情况下取决于用户是否在使用 `el.closest` 这类东西的情况下提供 polyfills (比如 https://polyfill.io) 。否则我们需要捆绑和维护大量的 polyfills，但是其他人一开始甚至并不需要它们！
