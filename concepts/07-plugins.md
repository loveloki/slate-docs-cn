# 插件

你已经知道了 Slate 编辑器 的行为可以被重写。这些重写可以被打包到一个插件里用于复用，测试以及分享。这就是关于 Slate 架构最强大的地方之前。

插件是一个简单的函数，接收一个 `Editor` 对象，并在以某种方式对其更改后返回它。

比如，一个标记图像节点为空（void）的插件：

```js
const withImages = editor => {
  const { isVoid } = editor

  editor.isVoid = element => {
    return element.type === 'image' ? true : isVoid(editor)
  }

  return editor
}
```

然后可以这样使用它：

```js
import { createEditor } from 'slate'

const editor = withImages(createEditor())
```

这种插件组合模型使 Slate 极易扩展！

## 辅助函数

除了插件功能，你可能还想要暴露和插件一起使用的辅助函数。比如：

```js
import { Editor, Element } from 'slate'

const MyEditor = {
  ...Editor,
  insertImage(editor, url) {
    const element = { type: 'image', url, children: [{ text: '' }] }
    Transforms.insertNodes(editor, element)
  },
}

const MyElement = {
  ...Element,
  isImageElement(value) {
    return Element.isElement(element) && element.type === 'image'
  },
}
```

然后你就可以在任何地方同时使用 `MyEditor` 和 `MyElement` 了。
