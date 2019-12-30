# 接口

Slate 使用纯 JSON对象。它要求 JSON 对象符合某些接口。比如，一个 Slate 的文本节点必须遵守 `Text` 接口：

```ts
interface Text {
  text: string
  [key: string]: any
}
```

这就意味着它必须有一个文本内容的 `text` 属性。

但是也可以添加**任何**其他的自定义属性，这完全取决于你。你可以为特殊用例和域，添加任何你喜欢的格式逻辑来定制数据，而不会受到 Slate 的干扰。

这种基于接口的办法将 Slate 和大多数其他要求使用它们 hand-rolled 模型的富文本编辑器分开，这让你更容易去理解它。这同样意味着避免了在初始化数据模型时的时间损失。

## 自定义属性

再举一个例子， `Element` 节点在 Slate 中是这样子的：

```ts
interface Element {
  children: Node[]
  [key: string]: any
}
```

这是非常宽松的接口。它要求有一个 `children` 属性来定义所包含的子节点。

但是你可以使用自定义的属性来扩展元素（或者其他接口）。比如，你可能有一个 "paragraph" 和 "link" 元素：

```js
const paragraph = {
  type: 'paragraph',
  children: [...],
}

const link = {
  type: 'link',
  url: 'https://example.com',
  children: [...],
}
```

`type` 和 `url` 属性就是你的自定义 API 。Slate 知道这些属性的存在，但是它不知道这些属性能干嘛。不过，当它渲染一个 `link` 元素的时候，你会收到一个包含这些自定义属性的对象。所以你可以这样去渲染它：

```jsx
<a href={element.url}>{element.children}</a>
```

在使用 Slate 的初始阶段，理解它定义的所有接口是很重要的。在每篇指南中都会有对于接口的一些讨论。

## 辅助函数

除了输入信息之外，Slate 的每个接口也会暴露一系列的辅助函数，使它们更容易使用。

比如，在处理节点时：

```js
import { Node } from 'slate'

// 获取节点的文本内容。
const string = Node.string(element)

// 在根节点的特定路径处获取节点。
const descendant = Node.get(value, path)
```

或者，在使用 ranges （文档片段）的时候：

```js
import { Range } from 'slate'

// 按顺序获取一个 range 的开头和结尾
const [start, end] = Range.edges(range)

// 检查一个 range 是否是一个单点（气质位置和终止位置是同一点）
if (Range.isCollapsed(range)) {
  // ...
}
```

在处理不同的接口时，有大量的辅助函数可以用于所有常见的用例。当你开始写代码的时候，最好把他们都阅读一遍，因为通常你可以把复杂的逻辑简化为几行辅助函数代码。

## 自定义辅助函数

除了内置的辅助函数，你可能想要定义自己的辅助函数并且暴露在自定义的命名空间里面。

举个例子，如果你的编辑器支持图片，你可能需要一个辅助函数来判断一个元素是不是图片元素：

```js
const isImageElement = element => {
  return element.type === 'image' && typeof element.url === 'string'
}
```

你可以轻松地把它们定义为一次性函数。但是你也可以像核心接口那样把它们绑定到命名空间，再去使用它们。比如：

```js
import { Element } from 'slate'

// 你可以在任何地方使用 `MyElement` 来访问你的扩展。
export const MyElement = {
  ...Element,
  isImageElement,
  isParagraphElement,
  isQuoteElement,
}
```

这样可以轻松地将定制域的逻辑与内置的 Slate 辅助函数一起重用。
