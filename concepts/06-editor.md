# 编辑器

Slate 编辑器所有的行为，内容，和状态都会汇总到到一个顶级的 `Editor` 对象中。它的接口是这样的：

```ts
interface Editor {
  children: Node[]
  selection: Range | null
  operations: Operation[]
  marks: Record<string, any> | null
  [key: string]: any

  // 自定义模式（schema）的节点行为
  isInline: (element: Element) => boolean
  isVoid: (element: Element) => boolean
  normalizeNode: (entry: NodeEntry) => void
  onChange: () => void

  // 可重写的核心行为。
  addMark: (key: string, value: any) => void
  apply: (operation: Operation) => void
  deleteBackward: (unit: 'character' | 'word' | 'line' | 'block') => void
  deleteForward: (unit: 'character' | 'word' | 'line' | 'block') => void
  deleteFragment: () => void
  insertBreak: () => void
  insertFragment: (fragment: Node[]) => void
  insertNode: (node: Node) => void
  insertText: (text: string) => void
  removeMark: (key: string) => void
}
```

看起来比其他接口要稍微复杂一点，因为它包含了所有你定义的顶级函数和自定义域的操作。

 `children` 属性包含了组成编辑器内容的节点的文档树。

T`selection` 属性包含了用户当前选择的文档片段（如果有的话）。

`operations` 属性包含了所有自上次更改以来所应用的操作。（Slate 在事件循环两个tick之间的分批操作）

`marks` 属性存储了光标附带的格式，该格式将应用到插入的文本上。

## 重写行为

在之前的指南中我们已经暗示了这一点，你可以通过重写编辑器的函数属性来重写它的任何行为。

比如，如果你想定义链接元素是一个行内元素：

```js
const { isInline } = editor

editor.isInline = element => {
  return element.type === 'link' ? true : isInline(element)
}
```

或者你可能想要重写 `insertText` 行为来“链接”地址：

```js
const { insertText } = editor

editor.insertText = text => {
  if (isUrl(text) {
    // ...
    return
  }

  insertText(text)
}
```

或者你甚至想要自定义“规范”，用来确保链接遵从某些约束：

```js
const { normalizeNode } = editor

editor.normalizeNode = entry => {
  const [node, path] = entry

  if (Element.isElement(node) && node.type === 'link') {
    // ...
    return
  }

  normalizeNode(entry)
}
```

无论你怎样重写行为，确保调用默认的函数作为后备行为。除非你真的想要完全移除默认行为。（这不是一个好主意！） 

## 辅助函数

`Editor` 接口和所有的 Slate 接口一样，暴露了一些有用的辅助函数用来实现某些行为。比如说：

```js
// 获取路径上指定节点的起点。
const point = Editor.start(editor, [0, 0])

// 从一个选择范围(range)获取文档片段。
const fragment = Editor.fragment(editor, range)
```

还有一些基于迭代的辅助函数，比如：

```js
// 迭代选择范围（range）的每一个节点。
for (const [node, path] of Editor.nodes(editor, { at: range })) {
  // ...
}

// 迭代当前文档片段（selection）的文本节点的每一点(point)。
for (const [point] of Editor.positions(editor)) {
  // ...
}
```
