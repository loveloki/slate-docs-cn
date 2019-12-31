# 操作：Operations

操作是在当调用命令和转换时候执行的细粒度（granular）低级操作。单个命令可能导致许多更低级的操作被应用到编辑器。

和命令不同的地方是，操作是不可扩展的。Slate 核心定义了所有在富文本文档上可能用到的操作。比如说：

```js
editor.apply({
  type: 'insert_text',
  path: [0, 0],
  offset: 15,
  text: 'A new string of text to be inserted.',
})

editor.apply({
  type: 'remove_node',
  path: [0, 0],
  node: {
    text: 'A line of text!',
  },
})

editor.apply({
  type: 'set_selection',
  properties: {
    anchor: { path: [0, 0], offset: 0 },
  },
  newProperties: {
    anchor: { path: [0, 0], offset: 15 },
  },
})
```

Slate 会自动将复杂的命令转换为低级操作，然后把它们应用到文档上，所以你基本上不用考虑它们。

> 🤖 Slate 的编辑行为被定义为操作（operations），这使得协同编辑成为可能，因为每个变化都是更容易被定义、应用、组合的，以及更容易实现撤销事件的！

