# 添加事件处理程序

好的，你已经安装了 Slate 并且在页面上渲染了出来。并且当你输入的时候，会看到发生的变化。但是你一定希望做更多而不是仅仅输入一些纯文本。

 Slate 很棒的原因是你可以如此轻松地去定制它。就像是在其它 React 组件中你曾经那样做的一样，当某些事件触发的时候 Slate 允许你传递事件处理程序。

当我们按下按键时候，让我们使用 `onKeyDown` 事件去改变编辑器的内容。

这是我们之前的app：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([
    {
      type: 'paragraph',
      children: [{ text: '段落中的一行文本。' }],
    },
  ])

  return (
    <Slate editor={editor} value={value} onChange={value => setValue(value)}>
      <Editable />
    </Slate>
  )
}
```

现在让我们添加一个 `onKeyDown` 处理程序：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([
    {
      type: 'paragraph',
      children: [{ text: '段落中的一行文本。' }],
    },
  ])

  return (
    <Slate editor={editor} value={value} onChange={value => setValue(value)}>
      <Editable
        // 定义一个新的处理程序在控制台打印按下的键。
        onKeyDown={event => {
          console.log(event.key)
        }}
      />
    </Slate>
  )
}
```

非常棒！现在当我们在编辑器中按下按键，它相应的 keyCode 会被打印到控制台中。

现在我们想要让它实际上去改变内容。就我们的示例而言，让我们实现在输入时将所有的 `&` 转换为 `and` 。

我们的 `onKeyDown` 事件处理程序可能会是这个样子：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([
    {
      type: 'paragraph',
      children: [{ text: '段落中的一行文本。' }],
    },
  ])

  return (
    <Slate editor={editor} value={value} onChange={value => setValue(value)}>
      <Editable
        onKeyDown={event => {
          if (event.key === '&') {
            // 阻止插入 `&` 字符的默认事件。
            event.preventDefault()
            // 执行insertText方法插入某些文本。
            editor.insertText("and")
          }
        }}
      />
    </Slate>
  )
}
```

添加完成后，试着输入 `&` ，你会发现它突然变成了插入 `and` ！

这个示例展示了 Slate 的事件处理程序可以做到什么。`editor` 可以让你运行命令去执行所有的 `event` 对象。就是这么简单！
