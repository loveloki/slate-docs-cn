# 规范化（Normalizing）

Slate 编辑器可以编辑复杂的，嵌套的数据结构。大部分时候它会完成的很好，但是有些情况下不一致的数据结构会被引入 — 通常是因为允许用户粘贴任意格式的富文本内容。

"规范化" 是确保你的编辑器的内容总是正确形式的办法。它与"验证"相似，只是它的任务是修复内容，使它重新有效，而不仅仅是判断内容是否有效。

## 内建的约束

Slate 编辑器内置了开箱即用的约束。这些约束是为了确保内容比 `contenteditable` 的标准内容更具有可预测性。Slate 所有内置的逻辑依靠这些约束，所以很可惜，你不能忽略它们。它们是：

1. **所有 `Element` 节点最后必须包含至少一个 `Text` 节点。**如果一个元素节点不包含任何子节点，那么会添加一个空的文本节点作为它的唯一子节点。这个约束确保选择范围（selection）的锚点（anchor）和焦点（focus）总是指向任意节点内部（通过依赖文本节点的引用）。这样，空元素（或者“void”类型对象）就无法被选择。

2. **两个相邻的有同样属性的文本会被合并。** 如果两个相邻的文本节点有相同的格式，它们会被合并到一个文本节点中。这样会避免文本节点无限制扩展数量，因为添加和删除格式都会分割文本节点。

3. **块节点要么只能包含其他块节点，要么包含行内节点与文本节点。** 比如，一个 `paragraph` 块节点不能包含同时包含另一个 `paragraph` 块节点和一个 `link` 行内元素。可以包含的子节点由第一个子节点所决定，任何其他不被允许的子节点会被移除。这确保了常见的富文本行为（比如“把一个块元素分割成两个”）始终如一。

4. **顶级的编辑器节点只能包含块节点。** 如果任何一个顶级子节点是行内节点或者是文本节点，它们会被移除。这确保了编辑器总是包含块节点，以便像是“分割一个块节点为两个块节点”之类的行为可以按照预期被执行。

这些默认约束都是强制性的，因为它们保证 Slate 文档有_更好的_可预测性。

> 🤖 虽然这些是我们现在能够发现最好的约束，但是我们总会寻找其他办法使 Slate 内置的约束尽可能地变得更少 — 只要它能保持默认行为容易理解。如果你找到一种方法来减少或者移除内置约束，我们都会洗耳恭听！

## 添加约束

内置约束是相当通用的。但是你可以在特定于你的域的内置约束之上添加自己的约束。

为了做到这些，你应该扩展编辑器的 `normalizeNode` 函数。在每一次对一个节点（或者他的后代）应用插入或者更新操作时都会调用 `normalizeNode` 函数，这让你有机会确保这个改变不会使其变得不可控，并且在这种情况下修正节点。

比如这是一个确保 `paragraph` 的子元素只包含文本或行内节点的插件：

```js
import { Element, Node } from 'slate'

const withParagraphs = editor => {
  const { normalizeNode } = editor

  editor.normalizeNode = entry => {
    const [node, path] = entry

    // 对于段落元素，确保它的子元素是有效的。
    if (Element.isElement(node) && node.type === 'paragraph') {
      for (const [child, childPath] of Node.children(editor, path)) {
        if (Element.isElement(child) && !editor.isInline(child)) {
          Transforms.unwrapNodes(editor, { at: childPath })
          return
        }
      }
    }

    // 退回默认的 `normalizeNode` 函数保证其他约束可用。
    normalizeNode(entry)
  }

  return editor
}
```

这个例子是很简单的。不论 `normalizeNode` 函数什么时候被段落元素调用，它会循环每一个子元素确保没有块元素。如果存在块元素，他会被解封，所以这个块元素被移除然后它的子元素替代了它。这样，这个节点就被修复了。

但是如果子元素是嵌套的块元素呢？

## 多重规范化

需要去理解 `normalizeNode` 约束的一点是它是**多重的**。

如果你再次查看这个例子，你会注意到它的 `return` ：

```js
if (Element.isElement(child) && !editor.isInline(child)) {
  Transforms.unwrapNodes(editor, { at: childPath })
  return
}
```

你可能首先认为这是奇怪的，因为有了 `return` ，默认的 `normalizeNodes` 就永远不会被调用，那么内置的约束就不会有机会运行它自己的规范化。

但是，这是一点对于规范化的“假象”。

当你调用 `Editor.unwrapNodes` 的时候，你会自动改变节点的内容，而他们在之前已经被规范化了。所以即使结束了当前的规范化的进行，通过更改节点，你还是开始了新的一轮规范化操作。这导致了某种 _递归式_ 的规范化。

这种多次进行的特性使得编写规范化_更加容易_，因为你一次只需要去担心怎样修复一个单一的问题，不一次性修复_所有_可能的问题（这样可能让节点处于无效状态）。

要明白实际上它是如何工作的，让我们从一个无效的文档开始：

```jsx
<editor>
  <paragraph a>
    <paragraph b>
      <paragraph c>word</paragraph>
    </paragraph>
  </paragraph>
</editor>
```

编辑器从 `<paragraph c>` 开始运行 `normalizeNode` 操作。现在它是有效的，因为它的子节点只有文本节点。

但是接下来，它移动到树的上一层，现在对 `<paragraph b>` 调用 `normalizeNode` 操作。这个段落是无效的，因为它包含了一个块元素（ `<paragraph c>` ）。所以这个块级的子元素被解封，现在新的文档是这样的：

```jsx
<editor>
  <paragraph a>
    <paragraph b>word</paragraph>
  </paragraph>
</editor>
```

随着修复的执行，顶级的 `<paragraph a>` 被改变了。它被规范化了，而且它也是无效的。所以 `<paragraph b>` 被解封，结果是：

```jsx
<editor>
  <paragraph a>word</paragraph>
</editor>
```

当运行 `normalizeNode` 时，没有发生任何变化，所以现在文档是有效的！

> 🤖 大部分情况下你不需要考虑这些内部情况。你只需要知道不管什么时候你调用 `normalizeNode` 时发现无效状态，可以修复这个无效状态并且相信 `normalizeNode` 会被再次调用直到节点变得有效。

## 错误的修复

然而，一个要避免的错误是它创建了无限的规范化循环。这是可能发生的，如果你查看特定的无效结构，但是接下来**没有**实际上通过改变这个节点来修复这个结构，就会发生这种情况。这样导致进入到一个无限循环，因为这个节点继续被标记为无效，但是从未被正确地修复！

比如，考虑规范化一个 `link` 元素，让它有一个有效的 `url` 属性：

```js
// 警告：这是一个错误的例子！
const withLinks = editor => {
  const { normalizeNode } = editor

  editor.normalizeNode = entry => {
    const [node, path] = entry

    if (
      Element.isElement(node) &&
      node.type === 'link' &&
      typeof node.url !== 'string'
    ) {
      Transforms.setNodes(editor, { url: null }, { at: path })
      return
    }

    normalizeNode(entry)
  }

  return editor
}
```

这个修复程序写的不正确。它想要确保所有的 `link` 元素有一个 `url` 属性的字符串。但是修复无效的 `link` 元素时，它被设置为了 null，这不是一个字符串！

在这个例子中你可能会去解封这个链接，完全移除它。_或者_选择扩展验证，接受一个空的 url （`url == null`） 。
