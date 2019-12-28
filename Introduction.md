# Slate

> 译者注：本翻译对应于 Slate 的 `v0.57.1` 版本。

[Slate](http://slatejs.org) 是一个_完全_可定制的富文本编辑器框架。

Slate 让你构建像[Medium](https://medium.com/), [Dropbox Paper](https://www.dropbox.com/paper) 或者是 [Google Docs](https://www.google.com/docs/about/) （它们正成为web应用的标杆）这样丰富，直观的编辑器，而不会让你在代码实现上深陷复杂度的泥潭。

它之所以能做到这一点，是因为所有的逻辑都是通过一系列的插件来实现的。所以你甚至不会被 “什么是 'slate' 的核心” 所约束。你可以认为它是基于 [React](https://facebook.github.io/react/) 的一种可拔插的 `contenteditable` 实现。它的灵感来源于 [Draft.js](https://facebook.github.io/draft-js/)，[Prosemirror](http://prosemirror.net/) 和 [Quill](http://quilljs.com/) 这样的库。

> 🤖 **Slate 目前处于测试状态**. 它的核心 API 现在已经可用，但是你可能需要为一些复杂用例 pull 修复请求。它的一些 APIs 还没有 "最终确定"，并且可能会随着时间的推移被改变，如果我们可以找到更好的实现方式。

## Why?

为什么要发明 Slate? 嗯... _(注意：这一部分有一些是 [我的](https://github.com/ianstormtaylor) 个人意见！)_

在发明 Slate 之前，我尝试了大量的富文本编辑器库 — [**Draft.js**](https://facebook.github.io/draft-js/)，[**Prosemirror**](http://prosemirror.net/)，[**Quill**](http://quilljs.com/) 等等等等。我发现使用它们构建简单的demo是没有问题的，但是当你开始构建类似于 [Medium](https://medium.com/)，[Dropbox Paper](https://www.dropbox.com/paper) 或者[Google Docs](https://www.google.com/docs/about/) 这样的项目，你会遇到深层次的问题：

- **编辑器的 "schema" 是硬编码的。** 像是粗体和斜体这样的样式是可以开箱即用的，但是像是注释、嵌入内容，或者其他定制需求需要怎么做呢？

- **编程式转换文档是非常复杂的。**用户的编写体验可能非常不错，但是编程式变换却是不必要的复杂，而这对于构建高级的编辑行为至关重要。

- **对HTML，MarkDown等内容的序列化支持看起来像是事后加上的。**将文档转换为HTML或MarkDown这样简单的事情都需要编写大量的模板代码。而这是非常常见的使用场景。

- **重新发明一个视图层似乎是效率低并且有局限的。**大多数编辑器使用自己的视图层而不是使用 React 这样已有的技术方案，所以你必须学习一个具有“陷阱”的全新系统。

- **协同编辑不是预先设计好的。**通常，编辑器对数据结构的内部实现让它无法用于实时，协同编辑，除非完全重写编辑器。

- **代码仓库是庞大的，并非小而可复用的。**许多编辑器没有对开发者开放可能重用的内部工具，以至于不得不造轮子。

- **无法构建复杂，嵌套的文档。**许多编辑器是围绕着简单的“扁平化”的文档结构来设计的，这使得像是表格，嵌入内容和字幕等内容难以设计，甚至于无法实现。

当然并不是所有的编辑器都会出现这样的问题，但是如果你尝试使用其他编辑器，可能会遇到类似的问题。为了避开这些这些API's的限制，去达到你追求的用户体验，不得不采取一些非常 hack 的办法。而有些体验是根本无法实现的。

如果这些听起来很熟悉，那么你可能会喜欢上 Slate。

让我来介绍一下 Slate 是如何解决这些问题的吧：

## 原则

Slate 尝试通过一些原则去解决 "[Why?](about:blank#why)" 这一节提到的问题：

1. **插件是一等公民。** Slate 最重要的部分就是插件是一等公民实体。这意味着你可以_完全_ 定制编辑体验，去建立像 Medium 或是 Dropbox 这样复杂的编辑器，而不必对各种类库进行猜测。

2. **精简的 Schema 核心。** Slate 的核心逻辑对你编辑的数据结构进行的假设非常少，这意味着当你构建复杂用例时，不会被任何的预制内容所阻碍。

3. **嵌套文档模型。** Slate 文档所使用的模型是一个嵌套的，递归的树，就像 DOM 一样。这意味着对于高级用例来说，构建像表格或是嵌套引用这样复杂的组件是可能的。但是你也可以使用单一层次的结构来保持简单性。

4. **与 DOM 相同。** Slate 的数据模型基于 DOM — 文档是一个嵌套的树，它使用文本选区（selections）和范围（ranges），并且公开所有的标准事件处理函数。这意味着像是表格或者是嵌套引用这样的高级特性是可能的。几乎所有你在 DOM 中可以做到的事情，都可以在 Slate 中做到。

5. **直观的指令。** Slate 文档使用命令（commands）来进行编辑，它被设计为高级并且非常直观地进行编辑和阅读，以便定制功能尽可能地具有表现力。这大大的提高了你理解代码的能力。

6. **可协作的数据模型。** Slate使用的数据模型 — 特别是操作如何应用到文档上 — 被设计为允许协同编辑在最顶层，所以如果你决定要实现协同编辑，不必去考虑彻底重构。

7. **明确的核心划分。**使用插件优先的结构和精简核心，使得核心和定制的边界非常清晰，这意味着核心的编辑体验不会被各种边缘情况所困扰。

## Demo

在 [**live demo**](http://slatejs.org) 查看所有示例。

## 示例

为了理解如何使用Slate，请查看这些示例：

- [**Plain text**](https://www.slatejs.org/examples/plaintext) — 展示最基础的例子：一个经过美化的  `<textarea>` 。
- [**Rich text**](https://www.slatejs.org/examples/richtext) — 展示了最基础的富文本编辑器的特性。
- [**Markdown preview**](https://www.slatejs.org/examples/markdown-preview) — 展示了对于类似于 MarkDown 的快捷键如何添加按键处理。
- [**Links**](https://www.slatejs.org/examples/links) — 展示了如何在行内节点中将文本和关联数据进行包装。
- [**Images**](https://www.slatejs.org/examples/images) — 展示了如何使用 void （无文本）节点添加图像。
- [**Hovering menu**](https://www.slatejs.org/examples/hovering-menu) — 展示了如何实现一个上下文相关的悬停菜单。
- [**Tables**](https://www.slatejs.org/examples/tables) — 展示了如何通过嵌套 block 去渲染更高级的组件。
- [**Paste HTML**](https://www.slatejs.org/examples/paste-html) — 展示了如何使用 HTML 序列化器去处理粘贴的 HTML 的内容。
- [**Mentions**](https://www.slatejs.org/examples/mentions) — 展示了如何使用 void 的 行内节点实现简单的 @ 功能。

每一个例子都包含一个 **View Source** 指向如何实现它的链接。我们还有一些 [其他的例子](https://github.com/ianstormtaylor/slate/tree/master/site/examples) 。

如果你对显示常见需求的示例有任何的想法，欢迎提交拉取请求！

## 文档

如果你是第一次使用 Slate，可以查看 [Getting Started](http://docs.slatejs.org/walkthroughs/01-installing-slate) 实战演练和 [Concepts](http://docs.slatejs.org/concepts) 去熟悉 Slate 的架构和思维模型。

- [**Walkthroughs**](http://docs.slatejs.org/walkthroughs)：实战演练
- [**Concepts**](http://docs.slatejs.org/concepts)：概念
- [**FAQ**](http://docs.slatejs.org/general/faq)：常见问题
- [**Resources**](http://docs.slatejs.org/general/resources)：资源

如果这些还是不够，你可以随时 [阅读源码](https://github.com/ianstormtaylor/slate/tree/master/packages)，它含有大量的注释。

这是被翻译成其他语言的文档:

- [中文](https://doodlewind.github.io/slate-doc-cn/)：0.24.1版本翻译，太旧了。

如果你正在维护一个翻译，请随意拉取请求。

## 贡献！

所有的贡献都是超级受欢迎的！查看 [Contributing instructions](../Contributing.md) 获得更多信息！

Slate 使用 [MIT 许可协议](../License.md)。
