# 应用自定义格式

在之前的指南中我们学习了怎么创建一个自定义的 block 类型，去渲染不同的文本块。但是 Slate 支持自定义的不仅仅是 "blocks" 。

在这篇指南中，我们会像你展示如何添加自定义格式选项，如**粗体**, _斜体_, `代码` 或者 ~~删除线~~。

让我们从之前的应用程序继续前进吧：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([
    {
      type: 'paragraph',
      children: [{ text: '段落中的一段文本。' }],
    },
  ])

  const renderElement = useCallback(props => {
    switch (props.element.type) {
      case 'code':
        return <CodeElement {...props} />
      default:
        return <DefaultElement {...props} />
    }
  }, [])

  return (
    <Slate editor={editor} value={value} onChange={value => setValue(value)}>
      <Editable
        renderElement={renderElement}
        onKeyDown={event => {
          if (event.key === '`' && event.ctrlKey) {
            event.preventDefault()
            const { selection } = editor
            const [match] = Editor.nodes(editor, {
              match: n => n.type === 'code',
            })
            Transforms.setNodes(
              editor,
              { type: match ? 'paragraph' : 'code' },
              { match: n => Editor.isBlock(editor, n) }
            )
          }
        }}
      />
    </Slate>
  )
}
```

现在，我们编辑 `onKeyDown` 函数，当你按下 `control-B` 时，它会添加一个 `bold` 格式到你所选择的文本上：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([
    {
      type: 'paragraph',
      children: [{ text: '段落中的一段文本。' }],
    },
  ])

  const renderElement = useCallback(props => {
    switch (prop.element.type) {
      case 'code':
        return <CodeElement {...props} />
      default:
        return <DefaultElement {...props} />
    }
  }, [])

  return (
    <Slate editor={editor} value={value} onChange={value => setValue(value)}>
      <Editable
        renderElement={renderElement}
        onKeyDown={event => {
          if (!event.ctrlKey) {
            return
          }

          switch (event.key) {
            // 当按下 "`" ，保留我们代码块存在的逻辑。
            case '`': {
              event.preventDefault()
              const [match] = Editor.nodes(editor, {
                match: n => n.type === 'code',
              })
              Transforms.setNodes(
                editor,
                { type: match ? 'paragraph' : 'code' },
                { match: n => Editor.isBlock(editor, n) }
              )
              break
            }

            // 当按下 "B" ，加粗所选择的文本。
            case 'b': {
              event.preventDefault()
              Transforms.setNodes(
                editor,
                { bold: true },
                // 应用到文本节点上，
                // 如果所选内容仅仅是全部文本的一部分，则拆分它们。
                { match: n => Text.isText(n), split: true }
              )
              break
            }
          }
        }}
      />
    </Slate>
  )
}
```

非常棒，我们设置好了这个按键处理函数。但是！如果你已经尝试选择文本并按下 `Ctrl-B` ，你并不会看到任何的变化。这是因为我们还没有告诉 Slate 如何去渲染 "bold" 格式。

对于你添加的所有格式，Slate 都会把文本打散到叶子（leaves）节点，然后你需要告诉 Slate 如何去理解它，就像是对待 block 元素那样。所以让我们来定义一个 `Leaf` 组件：

```jsx
// 定义一个 React 组件来加粗渲染叶子节点。
const Leaf = props => {
  return (
    <span
      {...props.attributes}
      style={{ fontWeight: props.leaf.bold ? 'bold' : 'normal' }}
    >
      {props.children}
    </span>
  )
}
```

看起来很熟悉，对吗？

接下来，让我们告诉 Slate 这个叶子节点。为了做到这点，我们会通过 prop 传递一个名为 `renderLeaf` 的属性到编辑器。同时，让我们添加主动检查逻辑来改变格式。

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([
    {
      type: 'paragraph',
      children: [{ text: '段落中的一段文本。' }],
    },
  ])

  const renderElement = useCallback(props => {
    switch (props.element.type) {
      case 'code':
        return <CodeElement {...props} />
      default:
        return <DefaultElement {...props} />
    }
  }, [])

  // 通过 `useCallback` 定义一个可以记忆的渲染叶子节点的函数。
  const renderLeaf = useCallback(props => {
    return <Leaf {...props} />
  }, [])

  return (
    <Slate editor={editor} value={value} onChange={value => setValue(value)}>
      <Editable
        renderElement={renderElement}
        // 传递渲染叶子节点函数。
        renderLeaf={renderLeaf}
        onKeyDown={event => {
          if (!event.ctrlKey) {
            return
          }

          switch (event.key) {
            case '`': {
              event.preventDefault()
              const [match] = Editor.nodes(editor, {
                match: n => n.type === 'code',
              })
              Transforms.setNodes(
                editor,
                { type: match ? null : 'code' },
                { match: n => Editor.isBlock(editor, n) }
              )
              break
            }

            case 'b': {
              event.preventDefault()
              Transforms.setNodes(
                editor,
                { bold: true },
                { match: n => Text.isText(n), split: true }
              )
              break
            }
          }
        }}
      />
    </Slate>
  )
}

const Leaf = props => {
  return (
    <span
      {...props.attributes}
      style={{ fontWeight: props.leaf.bold ? 'bold' : 'normal' }}
    >
      {props.children}
    </span>
  )
}
```

现在，如果你尝试选择一些文本然后按下 `Ctrl-B` ，你会看到它变成了粗体！再一次让人惊叹！
