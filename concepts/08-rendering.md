# 渲染

Slate 最棒的地方之一就是使用 React 来渲染界面，所以它很适合你的应用程序。Slate 不会重新发明视图层让你去学习。它尝试尽可能保持 React化。

最后，Slate 让你可以控制你的富文本域中自定义节点的渲染行为。

你可以通过传递渲染属性（props）到顶级的 `<Editable>` 组件来定义这些行为。

举例来说，如果你想要渲染自定义元素组件，你可以传递 `renderElement` 属性（prop）：

```jsx
import { createEditor } from 'slate'
import { Slate, Editable, withReact } from 'slate-react'

const MyEditor = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const renderElement = useCallback(({ attributes, children, element }) => {
    switch (element.type) {
      case 'quote':
        return <blockquote {...attributes}>{children}</blockquote>
      case 'link':
        return (
          <a {...attributes} href={element.url}>
            {children}
          </a>
        )
      default:
        return <p {...attributes}>{children}</p>
    }
  }, [])

  return (
    <Slate editor={editor}>
      <Editable renderElement={renderElement} />
    </Slate>
  )
}
```

> 🤖 请确保在你自定义的组件里面混入了 `props.attributes` 并且渲染了 `props.children` ！这些属性必须被添加到组件内部的顶级 DOM 元素中，因为它们是 Slate 的 DOM 辅助函数渲染（处理）所必须的。children 是 Slate 自动为你管理的文档的实际文本内容。

你不只可以使用简单的 HTML 元素，还可以使用自定义的 React 组件：

```js
const renderElement = useCallback(props => {
  switch (props.element.type) {
    case 'quote':
      return <QuoteElement {...props} />
    case 'link':
      return <LinkElement {...props} />
    default:
      return <DefaultElement {...props} />
  }
}, [])
```

## 叶子（节点）

当渲染包含格式的文本时候，这些文本会被分组为一个个“叶子文本”，然后再应用它们相应的格式。

为了自定义每个叶子的渲染，你可以使用自定义的 `renderLeaf` 属性：

```jsx
const renderLeaf = useCallback(({ attributes, children, leaf }) => {
  return (
    <span
      {...attributes}
      style={{
        fontWeight: leaf.bold ? 'bold' : 'normal',
        fontStyle: leaf.italic ? 'italic' : 'normal',
      }}
    >
      {children}
    </span>
  )
}, [])
```

请注意，我们处理它的方式和 `renderElement` 有一点不同。因为文本格式的文本很简单，所以我们放弃使用 `switch` 语句，而是仅仅对少量的样式进行开关切换。（但是如果你更喜欢使用自定义组件，没有谁可以阻止你使用！）

对于文本格式级别的一个缺点是：你不能保证给定的格式都是“可连续的”的 — 这意味着只能作为一个单叶子。这个限制看起来和 DOM 很相似，这是一种无效的情况：

```html
<em>t<strong>e</em>x</strong>t
```

因为上面这个例子的元素没有正确的关闭自己，所以它们是无效的。相应的，你应该把上述 HTML 写成这个样子：

```html
<em>t</em><strong><em>e</em>x</strong>t
```

如果你碰巧在文本中添加了另一个交叉的标签，则有可能会重新排序结束标记。在 Slate 中渲染叶子也是这样 — 即使每一个单词都应用了格式，你也不能保证叶子就会是连续的，因为它取决于和其他格式交叉的方式。

当然，叶子的这些内容听起来十分复杂。但是，只要按照以下目的去使用文本级别格式和元素级别格式，你就不必过多地去思考它们：

- 文本属性用于 **不连续的**，字符级别的格式。
- 元素属性用于文档中 **连续的**，语义级别的元素。

## 装饰：Decorations

装饰是另一个类型的文本级别格式。它们类似于常规的自定义属性，不同之处在于每一个属性都适用于文档片段（ `Range` ），而不是与给定的文本节点关联起来。

然而，装饰是一个完全的基于它的内容在**渲染时期**计算出来的。这是有用的对于动态格式，比如像是语法高亮和搜索文本，当内容（或外部数据）发生变化的时候，有可能会改变格式。

## 工具栏，菜单，叠加层（Overlays），或者更多！

除了在 Slate 内部控制渲染节点，你也可以通过使用 `useSlate` hook，从其他的组件获取到当前的编辑器上下文。

这样其他组件就可以执行命令，查询编辑器状态，以及更多。。。

一个常见的用例就是渲染一个带有格式化按钮的工具栏，这些按钮会基于当前的选择范围（selection）来突出显示（是否可用）。

```jsx
const MyEditor = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  return (
    <Slate editor={editor}>
      <Toolbar />
      <Editable />
    </Slate>
  )
}

const Toolbar = () => {
  const editor = useSlate()
  return (
    <div>
      <Button active={isBoldActive(editor)}>B</Button>
      <Button active={isItalicActive(editor)}>I</Button>
    </div>
  )
}
```

因为 `<Toolbar>` 使用了 `useSlate` hook 去获取内容上下文，所以当编辑器变化的时候他会重新渲染，因此按钮的激活状态是同步的。
