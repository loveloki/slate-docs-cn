# 位置：路径，点，文档片段，选择范围

位置（Locations）是你使用 Slate 编辑器执行插入，删除或其他操作时引用文档中特定位置的方式。有几种不同种类的接口，对应着不同的使用情况。

## 路径：`Path`

路径是引用路径的最低级方式。每个路径都是一个简单的数字数组，它通过文档树中每个祖先节点的索引来引用一个节点：

```ts
type Path = number[]
```

比如，在这个文档中：

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
}
```

文本叶子节点的路径为： `[0, 0]` 。

## 点： `Point`

点比路径稍微清晰一点，它包含一个 `offset` 属性（偏移量）对于特定的文本节点：

```ts
interface Point {
  path: Path
  offset: number
  [key: string]: any
}
```

举个例子，同一片文档中，如果你想要引用第一个可以放置光标的位置，你可以这样：

```js
const start = {
  path: [0, 0],
  offset: 0,
}
```

或者，你想要引用这篇文章的结尾：

```js
const end = {
  path: [0, 0],
  offset: 15,
}
```

把点看做是选择范围（selection）的一个光标是很有用的。

> 🤖 点总是指向文本节点！因为它们是唯一一种可以拥有光标的字符串。

## 文档片段： `Range`

文档片段不仅指文档中的一个点，它是指两点之间的内容。（当你选择文本制造了一个选择范围（selection）的时候。）它的接口是：

```ts
interface Range {
  anchor: Point
  focus: Point
  [key: string]: any
}
```

> 🤖  "anchor" 和 "focus" 这两个属术语借用了 DOM 的概念，参考 [锚点](https://developer.mozilla.org/zh-CN/docs/Web/API/Selection/anchorNode) 和 [焦点](https://developer.mozilla.org/zh-CN/docs/Web/API/Selection/focusNode) 。

锚点和焦点是通过用户交互建立的。锚点并不一定在焦点的_前面_。就像在 DOM 一样，锚点和焦点的排序取决于选择范围的方向（向前或向后）。

这里是 MDN 的解释：

> 用户可能从左到右（与文档方向相同）选择文本或从右到左（与文档方向相反）选择文本。**anchor**指向用户开始选择的地方，而**focus**指向用户结束选择的地方。如果你使用鼠标选择文本的话，anchor 就指向你按下鼠标键的地方，而focus就指向你松开鼠标键的地方。anchor 和 focus 的概念不能与选区的起始位置和终止位置混淆，因为anchor指向的位置可能在focus指向的位置的前面，也可能在focus指向位置的后面，这取决于你选择文本时鼠标移动的方向（也就是按下鼠标键和松开鼠标键的位置）。
> — [`Selection`, MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Selection)

一个重要的区别是锚点和焦点**总是引用文档中的文本叶子节点。**而从不引用元素。这种行为和 DOM 不同，但是它简化了处理文档片段（ranges）的工作，因为需要处理的边缘情况更少了。

## 选择范围：Selection

当你需要引用两点之间的范围时，在 Slate's API 的多个地方都使用了 Ranges 。最常见的就是引用用户当前的选择。



选择范围是 `Editor` 的一个特殊范围。比如，某人选择了的整个句子：

```js
const editor = {
  selection: {
    anchor: { path: [0, 0], offset: 0 },
    focus: { path: [0, 0], offset: 15 },
  },
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
}
```

> 🤖 选择范围的概念也借用了 DOM（查看 [`Selection`, MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Selection)），虽然它的格式非常简单，因为 Slate 不允许在单个选择范围里包含多个文档片段，这使得更加容易去使用。

`Selection` 不是一个特殊的接口，它只是一个正好遵循了更通用的 `Range` 接口的对象。