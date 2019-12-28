# 安装 Slate

Slate 是一个被发布为多个 npm 包的 monorepo，所以你应该这样子去安装它：

```
yarn add slate slate-react
```

你还要确保安装 Slate 的相应依赖：

```
yarn add react react-dom
```

_注意，如果你更喜欢使用  Slate 的 预构建版本，你可以使用  `yarn add slate`  然后在  `dist/slate.js` 获取打包后的文件！ 查看 [Using the Bundled Source](./using-the-bundled-source.md) 指南获取更多信息。_

一旦你完成 Slate 的安装，那么你就需要导入它。

```jsx
// 导入 Slate 编辑器的工厂函数。
import { createEditor } from 'slate'

// 导入 Slate 组件和 React 插件。
import { Slate, Editable, withReact } from 'slate-react'
```

在我们使用这些导入之前，让我们从一个空的 `<App>` 组件开始吧：

```jsx
// 定义app...
const App = () => {
  return null
}
```

下一步是创建一个新的 `Editor` 对象。我们希望编辑器在渲染中保持稳定，所以我们使用 `useMemo` hook：

```jsx
const App = () => {
  // 创建一个不会在渲染中变化的 Slate 编辑器对象。
  const editor = useMemo(() => withReact(createEditor()), [])
  return null
}
```

当然我们没有渲染任何的东西，所以你不会看到任何变化。

接下来我们要使用 `value` 创建一个状态：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])

  // 跟踪编辑器中 value 的值。
  const [value, setValue] = useState([])
  return null
}
```

接下来渲染一个 `<Slate>` 上下文 provider。

provider 组件会跟踪你的 Slate 编辑器，它的插件，它的值，它的选区，和任何发生的的其他变化。它**必须**被渲染在任何 `<Editable>` 组件上。 不过它也可以通过使用 `useSlate` hook 为其它组件提供编辑器状态，比如工具栏，菜单等等。

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([])
  // 渲染 Slate 上下文。
  return (
    <Slate editor={editor} value={value} onChange={value => setValue(value)} />
  )
}
```

你可以认为 `<Slate>` 组件为它下面的所有组件提供一个"受控的"上下文。

这是略微不同于 `<input>` 或 `<textarea>` 的思维模型，因为富文本文档更加的复杂。你通常需要在可编辑内容旁添加工具栏，实时预览或其它复杂的组件。

通过共享上下文，其它的组件可以执行命令，查询编辑器状态等等。

好的，接下来让我们开始渲染 `<Editable>` 组件吧：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  const [value, setValue] = useState([])
  return (
    // 在上下文中添加一个可编辑的组件
    <Slate editor={editor} value={value} onChange={value => setValue(value)}>
      <Editable />
    </Slate>
  )
}
```

这个 `<Editable>` 组件的行为就像是 `contenteditable` 。在你渲染它的任何地方都会为最近的编辑器上下文渲染一个可编辑的富文本文档。

只剩下最后一步了。到目前为止我们一直使用空数组 `[]` 初始化编辑器的 value，所以它没有内容。让我们定义一个初始化的 value 来修复这个问题。

这个 value 是一个纯 JSON 对象。这是一个包含一些文本的单个段落的例子：

```jsx
const App = () => {
  const editor = useMemo(() => withReact(createEditor()), [])
  // 当设置 value 状态时，添加初始化值。
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

你完成了！

这就是 Slate 最基本的例子。如果你把它渲染到页面上，你应该会看到一个包含 `段落中的一行文本。` 文字的段落。当你打字的时候，你就会看到文本发生了变化！
