# 定制 Block 节点

在之前的例子中，我们从实现一个段落开始，但是实际上我们从来没有告诉过 Slate 关于 `paragraph` block 节点的任何信息。我们仅仅是使用了内置的默认渲染器，它使用的是普通古老的 `<div>` 。

但是你能做到的不止如此。Slate 允许我们定义任何类型的自定义 block 节点，比如块引用，代码块，列表项等。

我们将给你展示如何做到。让我们从之前的应用程序继续吧：

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
      <Editable
        onKeyDown={event => {
          if (event.key === '&') {
            event.preventDefault()
            editor.insertText("and")
          }
        }}
      />
    </Slate>
  )
}
```

现在让我们添加“代码块”到我们的编辑器中。

问题是，代码块不仅仅要渲染为一个普通的段落，它还需要以不同的方式渲染出来。为了做到这一点，我们需要为 `code` 元素节点定义一个特定的渲染器。

元素渲染器仅仅是一个简单的 React 组件。像是这样：

```jsx
// 为 code 节点定义一个 React 组件渲染器。
const CodeElement = props => {
  return (
    <pre {...props.attributes}>
      <code>{props.children}</code>
    </pre>
  )
}
```

非常简单。

看到 `props.attributes` 参数了吗？Slate 将需要在 block 顶层元素上渲染的属性通过这种方式传入。这样你就不必自己去构建它们了。你**必须**在你的组件中传入这些属性。

另外，看到 `props.children` 参数了吗？Slate 会自动为你渲染 block 的所有子元素，并且就像在其他 React 组件中那样，通过 `props.children` 传递给你。这样你就不必去为如何正确渲染文本节点或其他类似的事情而费神了。你**必须**将这些子节点作为最终的叶子节点在你的组件中渲染。

下面是一个默认元素的组件：

```jsx
const DefaultElement = props => {
  return <p {...props.attributes}>{props.children}</p>
}
```

现在，让我们为  `Editor` 添加一些渲染器：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([
    {
      type: 'paragraph',
      children: [{ text: '段落中的一段文本。' }],
    },
  ])

  // 基于传递的 `props` 定义一个渲染函数。
  // 我们在这里使用 `useCallback` 在随后的渲染中记住这个函数。
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
        // 传递 `renderElement` 函数。
        renderElement={renderElement}
        onKeyDown={event => {
          if (event.key === '&') {
            event.preventDefault()
            editor.insertText("and")
          }
        }}
      />
    </Slate>
  )
}

const CodeElement = props => {
  return (
    <pre {...props.attributes}>
      <code>{props.children}</code>
    </pre>
  )
}

const DefaultElement = props => {
  return <p {...props.attributes}>{props.children}</p>
}
```

好了，但是我们还需要一个办法让用户实际转换一个 block 为代码块。所以让我们修改 `onKeyDown` 函数，添加一个 `` Ctrl-` `` 快捷键来做这件事：

```jsx
// 从 Slate 导入 `Editor`
import { Editor } from 'slate'

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
            // 阻止插入 "`" 的默认行为。 
            event.preventDefault()
            // 否则，把当前选择的 blocks 的类型设为 "code"。
            Transforms.setNodes(
              editor,
              { type: 'code' },
              { match: n => Editor.isBlock(editor, n) }
            )
          }
        }}
      />
    </Slate>
  )
}

const CodeElement = props => {
  return (
    <pre {...props.attributes}>
      <code>{props.children}</code>
    </pre>
  )
}

const DefaultElement = props => {
  return <p {...props.attributes}>{props.children}</p>
}
```

现在，如果你按下 `` Ctrl-` `` ，你光标所在的块应该会转换为一个代码块！多么神奇！

但是我们忘记了一件事。当我们再次按下 `` Ctrl-` `` ，它应该从代码块变回普通段落。为了做到这点，我们需要添加一点点逻辑，基于我们当前选择的块是否已经是一个代码块来改变我们设置的类型：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [selection, setSelection] = useState(null)
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
            // 确定当前选中的块是否为任意的代码块。
            const [match] = Editor.nodes(editor, {
              match: n => n.type === 'code',
            })
            // 根据是否已经存在匹配项来切换 block 的类型。
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

现在你完成了！如果你在一个代码块中按下 `` Ctrl-` `` ，它将会变回一个段落！
