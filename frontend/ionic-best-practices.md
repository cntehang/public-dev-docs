# Ionic Best Practices

Here is a set of best practices using Ionic

## Independent Pages

Each page passes parameters to the next page, the parameters should be side-effects free, i.e., any changes in the next page shouldn't affect the current page. For example, only pass cloned `moment.Moment` date to the next page.

当页面数据独立，可以跳转到堆栈的任一位置。

## 利用 RxJs 的`Subject`在页面之间通信

`Subject`的好处是不用维护专门的变量来管理各种状态。比如，当 Pop 到上层页面或跳到没有直接关系的页面，用`Subject`可以很方便的通知目的页面需要接收数据或更新状态。对于 Pop 页面，可以用`callback`传递`Subject`. 对于不相关页面，则用`servcie`， 如下面代码所示:

```ts
// 下面三段代码分属三个不同文件，省去 import 和 declare service的代码

// 在单独service文件定义service
import { Injectable } from '@angular/core'
import { Subject } from 'rxjs/Subject'

@Injectable()
export class RefreshAppService {
  public refresh$ = new Subject<boolean>()

  sendRefresh() {
    this.refresh$.next(true)
  }
}

// 在from page
this.refreshService.sendRefresh()

// 在 destination page 的constructor
this.refreshService.refresh$.subscribe(() => this.refreshMyStates.bind(this))
```

## 页面跳转

### Tab 跳转

从某个 Tab 页面（或其堆栈里的任一页面），可以用如下代码跳到不同的 Tab。相比`navCtrl.setRoot()`，用`tabs.select(tabIndex)`可以保留`[rootParams]`的参数传递。

```ts
const tabs: Tabs = this.navCtrl.parent
this.navCtrl
  .popToRoot()
  .then(_ => tabs.select(NEXT_PAGE_TABS_INDEX))
  .catch(err => this.log.error(methodName, err))
```

### 同栈跳转

可以用堆栈索引：`this.navCtrl.popTo(this.navCtrl.getByIndex(THE_PAGE_STACK_INDEX)))`。 `THE_PAGE_STACK_INDEX`是 push 到堆栈的索引。Root Page 的索引为 0.

或者如下先搜索页面的 id：

```ts
let index: number
let views: any[] = this.navCtrl.getViews()
let found: boolean = views.some((view, i) => {
  index = i
  return view.id == 'MyAwesomePage'
})
found ? this.navCtrl.popTo(views[index]) : this.navCtrl.pop()
```

## Always Create Loading

It cannot be reused, create it every time you want to present it. Call following actions in `loading.dismiss.then()`.

## `loading.dismiss.then()`

在异步请求之后，关闭`loading`后的`then()`方法，最好只有一条结果处理语句 （log 语句不算）。因为处理结果时可能需要弹出对话框并根据不同结果执行不同后续操作，`loading.dismiss.then()`的多条语句容易造成混乱或逻辑错误。

## Always Create Alert

Same as the loading, create an alert for every use.

## Dynamic Style

`[ngClass]="{'teh-bg-warning': !isValidEmployeeName()}"`

## `NavParams`

通常我们提倡小 method，可是 VS Code 的 TypeScript 检查不能发现`constructor`调用其他 method 初始化一些 class field。还有，`NavParams`传进来的参数通常页面需要用，太多的`undefined`判断语句反而造成系统代码不好看。再有，如果没有检查而传进来的参数 undefined，
也是系统错误。

所以，所有系统用到的`NavParams`参数都应该直接在`constructor`里面获取，不用考虑小 method 甚至重用的规则。

但是所有通过`@Input`传入的参数还需要检查是否`undefined`。

## Binding with Undefined

For input, select etc.
`[ngModel]="traveler?.Document" (ngModelChange)="traveler && (traveler.Document=$event)"`

## Add Shared Component

To add a shared component, use `ionic g component my-component` to generate a component. It will generates three files: `html`, `scss`, `ts` for the component and a module file `component.module.ts` in the paraent folder.

To use Angular and Ionic directives in the new component, you need to `imports: [CommonModule, IonicModule]` in the component module file.

Then in the page module file, import the new module along with `IonicPageModule` as the following:
`imports: [ComponentsModule, IonicPageModule.forChild(MyPageComponent)]`.
