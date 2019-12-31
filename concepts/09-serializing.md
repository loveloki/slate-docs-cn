# 序列化

Slate 在构建数据模型的时候就考虑到了序列化。具体来说，它的文本节点的定义方式使其更容易一目了然地阅读，也容易被序列化为其它格式，比如 HTML 和 Markdown 。

并且，因为 Slate 使用纯 JSON 数据，你可以更容易地编写序列化逻辑。

## 纯文本

比如，从编辑器获取值然后返回为纯文本：

```js
import { Node } from 'slate'

const serialize = nodes => {
  return nodes.map(n => Node.string(n)).join('\n')
}
```

这里我们获取到了 `Editor` 的子节点（as a  `nodes` argument ），然后返回一个纯文本结构，其中每一个顶级节点通过换行符来分隔新行。

比如这个输入：

```js
const nodes = [
  {
    type: 'paragraph',
    children: [{ text: 'An opening paragraph...' }],
  },
  {
    type: 'quote',
    children: [{ text: 'A wise quote.' }],
  },
  {
    type: 'paragraph',
    children: [{ text: 'A closing paragraph!' }],
  },
]
```

输出结果为：

```txt
An opening paragraph...
A wise quote.
A closing paragraph!
```

注意这种方式无法区分块引用，这是因为我们讨论的是纯文本。但是你可以序列化数据为任何你喜欢的格式 — 毕竟它只是 JSON 。

## HTML

对于这个例子，这是一个序列化为类 HTML 的 `serialize` 函数：

```js
import escapeHtml from 'escape-html'
import { Node, Text } from 'slate'

const serialize = node => {
  if (Text.isText(node)) {
    return escapeHtml(node.text)
  }

  const children = node.children.map(n => serialize(n)).join('')

  switch (node.type) {
    case 'quote':
      return `<blockquote><p>${children}</p></blockquote>`
    case 'paragraph':
      return `<p>${children}</p>`
    case 'link':
      return `<a href="${escapeHtml(node.url)}">${children}</a>`
    default:
      return children
  }
}
```

这比上面的序列化为纯文本更清晰一些。它实际上执行了递归，通过子节点持续迭代，直到到达文本叶子节点。对于每一个接收到的节点，它都转换为一个 HTML 字符串。

它也可以将单个节点而不是一个数组作为输入，所以如果你像这样传递给编辑器：

```js
const editor = {
  children: [
    {
      type: 'paragraph',
      children: [
        { text: 'An opening paragraph with a ' },
        {
          type: 'link',
          url: 'https://example.com',
          children: [{ text: 'link' }],
        },
        { text: ' in it.' },
      ],
    },
    {
      type: 'quote',
      children: [{ text: 'A wise quote.' }],
    },
    {
      type: 'paragraph',
      children: [{ text: 'A closing paragraph!' }],
    },
  ],
  // `Editor` 对象还有被省略的其他的属性...
}
```

结果为（增加换行符保证可读性）：

```html
<p>An opening paragraph with a <a href="https://example.com">link</a> in it.</p>
<blockquote><p>A wise quote.</p></blockquote>
<p>A closing paragraph!</p>
```

这真的很简单！

## 反序列化

## Deserializing

在 Slate 中的另一个常见的用例是 — 反序列化。当你有一些特定格式的输入想要转换为 Slate 可编辑的 JSON 结构时。比如，当有人粘贴 HTML 到你的编辑器中，然后你想要保证它被解析成对你的编辑器正确的格式。

Slate 已经内建了辅助包 `slate-hyperscript` 来完成这件事。

最常见的用例就是使用 `slate-hyperscript` 来编写 JSX 文档，比如编写测试时。你可能会这样使用它：

```jsx
/** @jsx jsx */
import { jsx } from 'slate-hyperscript'

const input = (
  <fragment>
    <element type="paragraph">A line of text.</element>
  </fragment>
)
```

然后你的编译器（Babel，TypeScript等等）的 JSX 特性会转换这个 `input` 变量为：

```js
const input = [
  {
    type: 'paragraph',
    children: [{ text: 'A line of text.' }],
  },
]
```

这非常适合测试用例，或者当你希望能够易于阅读的格式编写大量的 Slate 对象的地方。

但是！这些不能有助于反序列化。

但是 `slate-hyperscript` 不仅仅是用于 JSX 的。它只是构建 _Slate 内容树_的办法。而这恰恰是当你想要反序列化一些像是 HTML 这样的内容时候想要做的事情。

比如，这里有一个对于 HTML 的 `deserialize` 函数：

```js
import { jsx } from 'slate-hyperscript'

const deserialize = el => {
  if (el.nodeType === 3) {
    return el.textContent
  } else if (el.nodeType !== 1) {
    return null
  }

  const children = Array.from(el.childNodes).map(deserialize)

  switch (el.nodeName) {
    case 'BODY':
      return jsx('fragment', {}, children)
    case 'BR':
      return '\n'
    case 'BLOCKQUOTE':
      return jsx('element', { type: 'quote' }, children)
    case 'P':
      return jsx('element', { type: 'paragraph' }, children)
    case 'A':
      return jsx(
        'element',
        { type: 'link', url: el.getAttribute('href') },
        children
      )
    default:
      return el.textContent
  }
}
```

它接受一个名为 `el` 的 HTML 元素对象，返回一个 Slate 对象片段。所以如果你有一个 HTML 字符串，你可以像这样解析和反序列化它：

```js
const html = '...'
const document = new DOMParser().parseFromString(html, 'text/html')
deserialize(document.body)
```

当你输入：

```html
<p>An opening paragraph with a <a href="https://example.com">link</a> in it.</p>
<blockquote><p>A wise quote.</p></blockquote>
<p>A closing paragraph!</p>
```

你会得到的结果为：

```js
const fragment = [
  {
    type: 'paragraph',
    children: [
      { text: 'An opening paragraph with a ' },
      {
        type: 'link',
        url: 'https://example.com',
        children: [{ text: 'link' }],
      },
      { text: ' in it.' },
    ],
  },
  {
    type: 'quote',
    children: [
      {
        type: 'paragraph',
        children: [{ text: 'A wise quote.' }],
      },
    ],
  },
  {
    type: 'paragraph',
    children: [{ text: 'A closing paragraph!' }],
  },
]
```

就像序列化函数一样，你可以扩展它来满足你自定义域模型的需要。
