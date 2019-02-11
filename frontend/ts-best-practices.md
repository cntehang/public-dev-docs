# TypeScript Best Practices

- Avoid inheritance and prefer composition.
- 使用 `Object.keys(myObject).forEach(key=> {...})` 代替 `for in`.
- 在 `Classes` 中的一些约定
  - 所有不需要对外部可见的成员应该使用 `private` 修饰
  - 所有**关键的**和使用 `public` 修饰的 `function` 必须有注释。
  - 一个 `function` 如果有 `return` 语句，就**必须**指定返回值类型，否则**建议**添加 `: void`
