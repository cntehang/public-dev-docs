# Code Documentation

All Angular-based projects use [CompoDoc](https://compodoc.app/) as code documentation tool to generate project document.

## What to Document

Document non-trivial modules, components, methods and important class memebers. Feel free to use Chinese, but please use English for technical terms.

Use `/docs` folder for additional project documents such as design document, implementation explanation etc.

Generate documents to `/dist/documentation` folder. The command is `npx compodoc -p tsconfig.json -n 'My App Documentation' --includes ./docs -d dist/documentation`.

建议项目的开发、设计文档放到 `docs`目录下面。产生的文档统一输出到 `dist/documentation`目录。

To see the output, use `-w -s -o` options with `compodoc`.

## How to Write Document

Compodoc uses JSDoc comments for documentation. 建议使用 VS Code 插件 [`document this`](https://marketplace.visualstudio.com/items?itemName=joelday.docthis) 进行代码注释。

### JSDoc Comments

The documentation should use one of the following styles:

```ts
/**
 * One line comment.
 */

/**
 * First line followed by a blank line.
 *
 * Second line
 */

/**
 * First line.
 * Second line.
 */
```

### JSDoc Tags

The following tags are supported in CompoDoc. 注意参数的注释不要写数据类型。JSDoc 可以从函数声明得到。

```ts
/**
 * A summary of the function。函数总结
 * @param target  The target to process. 这里不用写参数类型。
 * @returns The processed target number
 */
function processTarget(target: string): number

/**
 * 不用给下面的代码产生文档
 * @ignore
 */

/**
 * 参考相关的文档。
 * see link {@link Todo}
 * see link {@link Todo|TodoClass}
 * see link [Todo]{@link Todo}
 * see link [Google]{@link http://www.google.com}
 * see link {@link http://www.apple.com|Apple}
 * see link {@link https://github.com GitHub}
 */

/**
 * Example usage。使用举例。
 *
 * @example
 * <mwl-calendar-day-view
 *     [viewDate]="viewDate"
 *     [events]="events">
 * </mwl-calendar-day-view>
 */
```

### Routing Documentation

CompoDoc requires a const of type `Routes`, that is a parameter of `RouterModule.forRoot()) to generate routing document. Following is an example:

```ts
const appRoutes: Routes = [
  { path: 'about', component: AboutComponent },
  { path: '', component: HomeComponent },
]
RouterModule.forRoot(appRoutes)
```

### Documentation of module and component

每个项目有想过的设计文档和一些关键点的说明，放在 `docs`目录下面。

Add a `.md` file to the module or component with the same file name (except `.ts`). For example, for `my.component.ts`, add a `my.component.md` to document it with a file.

### Additional documentation

Adding additional markdown document files in a folder named `docs`. Then add a `summary.json` file as the following to give the structure of the additonal files:

```json
[
  {
    "title": "A TITLE",
    "file": "a-file.md"
  },
  {
    "title": "A TITLE",
    "file": "a-file.md",
    "children": [
      {
        "title": "A TITLE",
        "file": "a-sub-folder/a-file.md"
      }
    ]
  }
]
```

Use `--includes` flag to include the addtional files in the final documentation.

## Resources

- [CompoDoc Website](https://compodoc.app/)
- [A Live Demo](https://compodoc.github.io/compodoc-demo-todomvc-angular/)
- [The Live Demo Source Code](https://github.com/compodoc/compodoc-demo-todomvc-angular)
- [A tutorial of an Ionic project](https://compodoc.app/guides/tutorial.html)
- [Another video tutorial](https://youtu.be/90lnNtPmL8Y)
