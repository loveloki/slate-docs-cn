# 保存到数据库

现在你已经学会了如何向 Slate 编辑器添加功能的基本知识，你可能想要知道如何保存你已经编辑过的内容，这样你就可以在之后返回应用程序并且加载它。

在这篇指南中，我们会像你展示如何添加逻辑，以便将你在 Slate 中编辑的内容保存到一个数据库中并在随后读取它。

让我们从一个基本的编辑器开始：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([
    {
      type: 'paragraph',
      children: [{ text: '段落中的一段文本。' }],
    },
  ])

  return (
    <Slate editor={editor} value={value} onChange={value => setValue(value)}>
      <Editable />
    </Slate>
  )
}
```

它将会渲染一个基本的 Slate 编辑器在页面上，随着你输入内容会发生变化。但是如果你刷新页面，所有内容都将回到一开始的时候 -- 什么都没有被保存！

我们需要做的是保存你在某个地方做的改变。对于这个例子来说，虽然我们仅仅使用了 [Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) ，但是这会给你怎么保存数据带来灵感。

所以，在我们的 `onChange` 事件处理函数中，我们需要保存 `value` ：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([
    {
      type: 'paragraph',
      children: [{ text: '段落中的一段文本。' }],
    },
  ])

  return (
    <Slate
      editor={editor}
      value={value}
      selection={selection}
      onChange={value => {
        setValue(value)

        // 在 Local Storage 里保存值。
        const content = JSON.stringify(value)
        localStorage.setItem('content', content)
      }}
    >
      <Editable />
    </Slate>
  )
}
```

现在无论何时你编辑页面，如果你查看 Local Storage，你应该会看到 `content` 的值已经被改变。

但是，如果你刷新页面，还是和之前一样，什么都不存在了。这是因为我们需要确保初始值是从同一个 Local Storage 读取的。就像这样：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  // 如果 Local Storage 存在值，用它来更新初始值。
  const [value, setValue] = useState(
    JSON.parse(localStorage.getItem('content')) || [
      {
        type: 'paragraph',
        children: [{ text: '段落中的一段文本。' }],
      },
    ]
  )

  return (
    <Slate
      editor={editor}
      value={value}
      onChange={value => {
        setValue(value)
        const content = JSON.stringify(value)
        localStorage.setItem('content', content)
      }}
    >
      <Editable />
    </Slate>
  )
}
```

现在你应该能够保存刷新之间的变化了！

成功了 — 你已经有一个 JSON 对象在你的数据库了。

但是如果你想要保存的数据格式不是 JSON 呢？ 没问题，只要你用不同的方式去序列化你的值就好了。比如说你想要保存纯文本而不是 JSON，我们可以编写一些逻辑来序列化和反序列化纯文本的值。

```jsx
// 从 Slate 导入 `Node` 帮助函数接口。
import { Node } from 'slate'

// 定义一个参数为 `value` 返回值是纯文本的序列化函数。
const serialize = value => {
  return (
    value
      // 返回这个 value 中每一个段落中的子节点的字符串内容。
      .map(n => Node.string(n))
      // 用换行符（用换行符来区分段落）来连接他们。
      .join('\n')
  )
}

// 定义一个参数是字符串返回值是 `value` 的反序列化函数
const deserialize = string => {
  // 分隔字符串，返回一个包含value的child数组。
  return string.split('\n').map(line => {
    return {
      children: [{ text: line }],
    }
  })
}

const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  // 使用我们的反序列化函数来从 Local Storage 中读取数据。
  const [value, setValue] = useState(
    deserialize(localStorage.getItem('content')) || ''
  )

  return (
    <Slate
      editor={editor}
      value={value}
      onChange={value => {
        setValue(value)
        // 序列化 `value` 并将产生的字符串保存到 Local Storage。
        localStorage.setItem('content', serialize(value))
      }}
    >
      <Editable />
    </Slate>
  )
}
```

它也正常工作了！现在你保存的是纯文本。

你可以将你所喜欢的格式仿照这个策略去保存。你可以序列化为为 HTML，Markdown，甚至是根据你的实际情况定制的 JSON 格式。

> 🤖 请注意，虽然你可以用任何喜欢的方式去序列化值，但是他们需要被权衡。序列化过程本身是有成本的，有些格式可能比其他格式更加难以使用。通常来说，我们建议仅当在你有特殊用途的时候才编写自己的格式。另外，通常使用 Slate 默认的格式是最好的。
