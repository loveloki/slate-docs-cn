# 迁移

从更早的 Slate 版本迁移到 `0.50.x` 版本不是一个简单的任务。整个框架被重构（ re-considered from the ground up）。这导致了 **更好的** 抽象，让你可以编写更少的代码。但是迁移过程一点都不简单。

强烈建议你在阅读完这篇指南后，阅读全新的 [Walkthroughs](../walkthroughs/01-installing-slate.md) 和另一个 [Concepts](./01-interfaces.md) 来了解如何应用所有新的概念。

## 主要的区别

从架构的角度看，这里是 `0.50.x` 版本的 Slate （和之前版本）_主要_区别的概览。

### JSON！

数据模型现在由简单的 JSON 对象组成。之前，它使用 [Immutable.js](https://immutable-js.github.io/immutable-js/) 数据结构。这是一个巨大的改变，并且它带来了一些其他的东西。它（也很可能）会提升使用 Slate 的平均性能。它也使新手更容易入门。从中迁移是一个巨大的改变，但是这是值得的！

### 接口

数据模型（现在）是基于接口的。之前每一个模型都是一个类的实例。现在，（数据模型）不仅是一个数据纯对象，而且 Slate 也期待这个对象实现一个接口。所以过去位于 `node.data` 的自定义属性现在可以位于节点的顶层。

### 命名空间

在命名空间中，辅助函数集合暴露出大量的辅助函数。比如， `Node.get(root, path)` 或者 `Range.isCollapsed(range)` 。结果就是使代码更加清晰，因为你总是可以快速查看你正在使用的接口。

### TypeScript

现在代码库使用 TypeScript。使用纯 JSON 作为数据模型，并且使用基于接口的 API， 通过迁移到 TypeScript 可以简化这两件事。你不需要自己使用它，但是如果你使用它，则在使用 API 的时候会获得更多的安全性。（而且，如果你使用 VS Code，无论如何你都会获得更好的自动补全！)

### 更少的概念

接口的命令的数量已经减少了。之前的 `Selection`， `Annotation` 和 `Decoration` 曾经是单独的类。现在它们是实现 `Range` 接口的简单对象。之前的 `Block` 和 `Inline` 是单独定义的，现在它们是一个实现 `Element` 接口的对象。 之前有 `Document` 和 `Value`，但是现在顶级的 `Editor` 节点包含了文档本身的子节点。

命令的数量也减少了。之前我们每一个输入类型都拥有一个命令，像是 `insertText`， `insertTextAtRange` 和 `insertTextAtPath`。它们已经被合并到一个更可定制的命令的更小的集合，比如 `insertText` 可以在 ` Path ， Range 或 Point` 中使用。

### 更少的包

为了减轻维护负担，和因为 Slate 核心新的抽象和  API，（它们）使事情变得更加容易，（所以）目前包的数量已经减少了。 像是 `slate-plain-serializer`， `slate-base64-serializer`等已经被移除了，并且如果需要的话，可以在用户域中轻松实现。甚至 `slate-html-deserializer` 可以在用户域中实现（in ~10 LOC leveraging `slate-hyperscript`）。并且不再暴露像是 `slate-dev-environment`， `slate-dev-test-utils`之类的内部包，因为它们是实现细节。

### 命令

添加了新的命令概念。（旧的命令概念现在被称为转换："transforms"。)新概念表达了用户编辑文档的语义化意图。并且它们允许对用户行为进行正确的抽象 — 比如改变当用户按下回车键或删除键时发生的情况等等。你应该重写命令行为来代替使用 `keydown` 事件。

命令通过调用 `editor.*` 的核心函数来触发。然后会依次通过类似中间层的栈，但是它们是由合成函数构建的。任何插件都可以通过重写编辑器行为来扩展它。

### 插件

插件现在是一个纯函数，可以增强接收到的 `Editor` 对象并且返回它们。比如可以通过编写 `editor.exec` 函数来增强命令执行。或者通过编写 `editor.apply` 来监听操作。之前它们依靠自定义的中间件栈，它们只是合成到编辑器的处理程序。现在我们使用纯的老的函数合成方式（也称为包装：wrapping）来替代它。

### 元素

块元素和行内元素现在是运行时进行的选择。之前它是被通过 `object: 'block'` 或者 `object: 'inline'` 属性被绑定到数据模型中的。现在，它在运行时检查一个元素是否是行内元素。比如，你可能需要检查 `element.type === 'link'` ，然后把它看作是一个行内元素。

### 更加React化

插件已经不再关心渲染和事件处理。之前插件可以完全控制渲染逻辑和编辑器事件处理逻辑，这产生了一个坏的激励：把**所有的**渲染逻辑放到插件里面，这导致了 Slate 完全被 React 包装（这很难做好）。取而代之的是，现在新的行为是插件只专注在富文本方面，而把渲染和事件处理交给 React。

### 内容

之前 `<Editor>` 组件不光是一个控制器对象，还是可编辑的 DOM 元素。这给其它 Slate 组件和它一起使用带来了大量的问题。在新版本中，有一个新的 `<Slate>` 上下文（provider）组件，和一个更简单的 `<Editable>` 类 `contenteditable` 组件。通过提升 `<Slate>` 上下文到你的组件树中更高的位置，你可以使用  `useSlate` hook 把编辑器分享给工具栏，按钮等等。

### Hooks

除了 `useSlate` hook，还有一些其它的 hooks。比如 `useSelected` 和 `useFocused` hooks 有助于了解何时渲染被选择的状态（通常用于空节点）。并且因为使用 React 的 Context API，当状态发生变化的时候它们会自动重新渲染。

### `beforeinput`

我们现在一般只使用 `beforeinput` 事件。代替依赖一系列的 shims 和 React 合成事件的奇怪行为，我们现在使用标准的 `beforeinput` 事件作为我们的基础。它完全支持 Safari 和 Chrome,，不久后就会支持新的基于 Chromium 内核的 Edge，并且目前已经可以在 Firefox 上使用了。与此同时，我们有一些针对 Firefox 的补丁让它运行。开箱即用的 Slate 核心不再支持 Internet Explorer。

### History-less

现在核心利益逻辑最终被提取到一个独立插件中。这使人们更容易实现自定义的历史行为。并且确保了插件有足够的控制权以复杂的方式增强编辑器，因为历史记录需要它。

### 无标记（Mark-less）

标记已经被从 Slate 数据模型移除。现在我们有能力正确地在节点上定义自定义属性，你可以为文本节点的自定义属性建立标记模型。比如加粗可以被简单的建模为 `bold: true` 属性。

### 无注释（Annotation-less）

熟悉地，注释（annotations）已经从 Slate 核心移除。现在，通过定义自定义操作和使用装饰（decorations）来渲染注释范围，从而完整地实现在用户环境（userland）中。但是大多数情况下应该使用自定义文本节点属性或者装饰。没有多少使用注释受益的用例。

## 减少体积

一个目标是大大简化 Slate 的大量逻辑使它更容易理解和迭代。这可以通过重构为更好的抽象表达来做到（凭借现代 DOM API，和迁移到更简单的 React 模式）。

使你对总代码行数的更改有所了解：

```
slate                       8,436  ->  3,958  (47%)
slate-react                 3,905  ->  1,954  (50%)

slate-base64-serializer        38  ->      0
slate-dev-benchmark           340  ->      0
slate-dev-environment         102  ->      0
slate-dev-test-utils           44  ->      0
slate-history                   0  ->    211
slate-hotkeys                  62  ->      0
slate-html-serializer         253  ->      0
slate-hyperscript             447  ->    345
slate-plain-serializer         56  ->      0
slate-prop-types               62  ->      0
slate-react-placeholder        62  ->      0

总共                      13,807  ->  6,468  (47%)
```

这是相当大的变化！这甚至不包括（框架逻辑）流程中摆脱的依赖关系。
