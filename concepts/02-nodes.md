# 节点：编辑器，元素和文本

最重要的类型是 `Node` 对象：

- 包含整个文档内容的 `Editor` 根节点。
- 在自定义域中拥有语义的 `Element` 容器节点。
- 包含文档文本的 `Text` 叶子节点。

这三个接口组合在一起形成一棵树 -- 就像 DOM 一样。举例来说，这是一个简单的纯文本值：

```js
const editor = {
  children: [
    {
      type: 'paragraph',
      children: [
        {
          text: 'A line of text!',
        },
      ],
    },
  ],
  // 编辑器还有被省略的其他属性。
}
```

尽可能地对应 DOM 是 Slate 的原则之一。人们总是使用 DOM 来描述类似富文本结构的文档。对应 DOM 有助于新用户熟悉类库，并且可以让我们重用经过重重考验的结构模式，而不是自己造一个新的轮子。

> 🤖 下面来自于 **MDN Web 文档** 的内容可以帮助你理解更多相应的 DOM概念：
>
> - [Document](https://developer.mozilla.org/en-US/docs/Web/API/Document)
> - [块级元素](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Block-level_elements)
> - [行内元素](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Inline_elements)
> - [文本元素](https://developer.mozilla.org/en-US/docs/Web/API/Text)

Slate 文档是一个嵌套递归的结构。在文档中，元素可以有子节点 — 所有元素都可以无限制地拥有子节点。嵌套递归的结构确保你可以建模简单的行为，比如用户的 @提及 和 # 标签或是带标题的表格和图片。

##  编辑器： `Editor`

Slate 的顶级节点就是 `Editor` 。它封装了文档的所有富文本内容。它的接口是这样的：

```ts
interface Editor {
  children: Node[]
  ...
}
```

我们稍后会介绍他的功能，但是对于节点来说最重要的部分是它的 `children` 属性，其中包含一个 `Node` 对象树。

## 元素：`Element`

元素组成了富文本文档的中间层。它们是对你的域定制的节点。它们的接口是这样的：

```ts
interface Element {
  children: Node[]
  [key: string]: any
}
```

你可以为任何类型的内容定义自定义元素。比如你可能想要一个段落和引用在你的数据模型中，它们通过 `type` 属性区分：

```js
const paragraph = {
  type: 'paragraph',
  children: [...],
}

const quote = {
  type: 'quote',
  children: [...],
}
```

需要提醒的是，你可以使用任何的自定义属性。在这个例子中， Slate 并不关心 `type` 属性具体是什么。如果你自定义了 "link" 节点，你可能有一个 `url` 属性：

```js
const link = {
  type: 'link',
  url: 'https://example.com',
  children: [...],
}
```

或者你可能给所有的节点定义一个 ID 属性：

```js
const paragraph = {
  id: 1,
  type: 'paragraph',
  children: [...],
}
```

重要的是元素总是有一个 `children` 属性。

## 块（Blocks）和行内（Inlines）

根据你的用例，你可能想要为 `Element` 定义一个另一个行为，这个行为决定了它的编辑流。

所有元素默认是块元素。它们被垂直空间隔开，并且永不重叠。

但是在某些情况下，比如链接，你可能想要把它作为行内元素流。 这样的话，它就会和文本节点位于同一级别和同样的流。

> 🤖 这个概念借用了 DOM 的行为，参考 [块级元素](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Block-level_elements) 和 [行内元素](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Inline_elements) 。

可以通过重写 `editor.isInline` 函数来定义哪些节点属于行内节点。（默认情况下，它总是返回false。）

元素可以将块元素作为子元素包含。或者混合行内元素和文本元素，把它们作为子节点。但是元素**不能**同时包含行内元素和块元素。

## 空元素（Voids）

类似于块元素和行内元素，另一个你可以定义的特殊元素为空元素：它们的 "void" 性：

元素默认是非空元素，意味着它的子元素是完全可以像文本一样编辑。但是有时候，比如图像， 你可能想要确保 Slate 不会将元素的内容作为可编辑的文本，而是看做一个黑箱。

> 🤖 这个概念是从 HTML 借用的，请查看 [空元素](https://www.w3.org/TR/2011/WD-html-markup-20110405/syntax.html#void-element) 。

可以通过定义 `editor.isVoid` 函数来定义哪些元素被视为 void。（默认情况下，它总是返回false。）

## 文本：`Text`

文本节点是树中的最低级节点，包含文档的文本内容以及任何格式。它的接口是：

```ts
interface Text {
  text: string
  [key: string]: any
}
```

比如这个例子，加粗文本：

```js
const text = {
  text: 'A string of bold text',
  bold: true,
}
```

文本节点可以包含任意的自定义属性，这就是你如何实现像是**粗体**，_斜体_，`代码`等自定义格式的办法。
