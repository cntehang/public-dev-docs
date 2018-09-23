# Code Documentation

All Angular-based projects use [CompoDoc](https://compodoc.app/) as code documentation tool to generate project document.

## Documentation

Compodoc uses JSDoc comments for documentation.

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

The following tags are supported in CompoDoc:

```ts
/**
 * A summary of the function
 * @param {string} target  The target to process
 * @returns The processed target number
 */
function processTarget(target: string): number

/**
 * @ignore
 */

/**
 * see link {@link Todo}
 * see link {@link Todo|TodoClass}
 * see link [Todo]{@link Todo}
 * see link [Google]{@link http://www.google.com}
 * see link {@link http://www.apple.com|Apple}
 * see link {@link https://github.com GitHub}
 */

/**
 * Shows all events on a given day. Example usage:
 *
 * @example
 * <mwl-calendar-day-view
 *             [viewDate]="viewDate"
 *             [events]="events">
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

Use `--include` flag to include the addtional files in the final documentation.

## Resources

- [CompoDoc Website](https://compodoc.app/)
- [A Live Demo](https://compodoc.github.io/compodoc-demo-todomvc-angular/)
- [The Live Demo Source Code](https://github.com/compodoc/compodoc-demo-todomvc-angular)
- [A tutorial of an Ionic project](https://compodoc.app/guides/tutorial.html)
- [Another video tutorial](https://youtu.be/90lnNtPmL8Y)
