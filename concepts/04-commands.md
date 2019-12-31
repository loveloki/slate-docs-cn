# 命令：Commands

当编辑富文本的时候，用户可能会插入文本，删除文本，分隔段落，添加格式等等。这些编辑行为都可以用两个概念来说明：命令和操作。

命令（Commands）是代表用户特定意图的高级操作。它们是 `Editor` 接口的辅助函数。辅助函数包含了一些富文本常用的核心行为，但是建议你编写针对自己特定模型的命令。

比如，下面这些是内置命令：

```js
Editor.insertText(editor, 'A new string of text to be inserted.')

Editor.deleteBackward(editor, { unit: 'word' })

Editor.insertBreak(editor)
```

但是你可以（一定会）定义自己的自定义命令。比如，你可能会根据允许的内容类型定义 `formatQuote` 命令， `insertImage` 命令，或者是 `toggleBold` 命令。

命令总是描述将要进行的操作，就好像是**用户**自己在执行一样。由于这个原因，它们从不需要定义执行命令的位置（location），因为它们总是对当前的选择范围执行操作。

> 🤖 命令的概念基本上来自于 DOM 内检的 [`execCommand`](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand) API。然而 Slate 定义了自己的简单（并且可扩展）的API ，因为 DOM 版本的API 表现过于不一致（opinionated and inconsistent.）。

因此，Slate 将每一个命令转换为一系列的低级操作（operations），这些操作用来产生一个新的值。这就是协同编辑可以实现的原因。同时你不用担心这些问题，因为它是自动完成的。

## 自定义命令

在编写自定义命令时，你可以创建自己的命名空间：

```js
const MyEditor = {
  ...Editor,

  insertParagraph(editor) {
    // ...
  },
}
```

你通常会使用 Slate 的 `Transforms` 来编写自定义命令。

## 转换：Transforms

转换（Transforms）是一个特殊的辅助函数，它允许你对文档执行各种特定的更改，比如：

```js
// 对选中的所有文本节点设置粗体。
Transforms.setNodes(
  editor,
  { bold: true },
  {
    at: range,
    match: node => Text.isText(node),
    split: true,
  }
)

// 把光标所在的最后一个块转换为引用块。
Transforms.wrapNodes(
  editor,
  { type: 'quote', children: [] },
  {
    at: point,
    match: node => Editor.isBlock(editor, node),
    mode: 'lowest',
  }
)

// 在指定路径（path）处插入一个新的文本用来替代原来的文本。
Transforms.insertText(editor, 'A new string of text.', { at: path })

// ...还有省略的其他变换操作！
```

转换辅助函数被设计为一个集合。所以你可能在每个命令中都会使用到它的一部分。
