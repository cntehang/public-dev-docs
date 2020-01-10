# Front Project

## 简单查询列表展示页

可以使用统一模板：[simple-query-page](./simple-query-page.md)

## Suggestion

- 简单查询条件 `form` 统一使用 `se` 组件，较复杂（需要我们自己控制布局 长短的）统一使用 `nz row + nz col`
- `Component` 中 方法名可以简单 `query add update delete` ，但是必须有注释
- `Http Service` 中方法名参照 Api 命名， 且必须有注释。命名风格 `const getXxxApi = admin/xxx/getXxx`
- 统一使用 `Router Service` 进行页面跳转，并且保证 一个 module 一个 router
- 针对 `SharedModule` (无论全局 shared 还是 子模块的 shared )中的 `models` `utils` 等不在 `SharedModule`  中导出的内容，请添加相应的 `index.ts` 索引。
  - `CoreModule` 中的 `services` `guards` 同理
  - 这样做的好处是，当目录结构发生改变或者文件名发生变化，但相应的 `model` `util` 导出的内容不变的时候，对于外部使用的 `import` 路径不会发生改变，不会出现牵一发而动全身的情况。
- `@input()` 定义输入属性后，依赖该属性的字段使用 `get` 声明，操作使用 `set` 或实现 `OnChanges`。防止输入属性变更时，未及时更新
  
### 日期格式化

项目中有工具库 `DateUtil` 以及过滤器 `_date` ，在函数库 [date-fns](https://date-fns.org/) 基础上作了一层封装。使用时注意以下几点：

- 日期显示格式、提交格式，务必依据接口具体情况具体处理，不需要格式化的地方不要进行格式化
- `dateFns` 在处理字符串格式日期时会做夏令时兼容，这点会将'1991-04-14'格式化成'1991-04-13'，如果依赖它处理 `Date` 要特别注意
