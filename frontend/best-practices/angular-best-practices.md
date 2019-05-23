# Angular Best Practices

Here is a set of best practices using Angular.

## Types of Angular Modules

An Angular module, called NgModule or simply module, is a class marked by the `@NgModule` decorator. The modules can be classified into the following types.

- The Root module: it is the root `AppModule` that lauches the app. This module be as simple as possible. For exaple, it should only use one, not many, top-level feature module to organize the UI.
- Feature module: feature modules handles UI features. It has two sub types.
  - eager-loading feature modules: these modules provide the UI features required in application start. Usually a feature module has a top component and many supporting sub-components. It declares and exports the top componnent but hides the sub-components. It is usually imported by the root module and other eager-loading modules. For simplicity, it is called eager-loading module or eager module.
  - routed feature module: these modules proide the UI features that are targets of navigation routes. They can be preloaded or lazy loaded. They don't declare and export components because they are loaded by the router and are usd independently. They are also called as routed modules.
- Routing module: a routing module provides routing configuration for a routed module and should have a matching name. For example, the root module `AppModule` has a routing module `AppRoutingModule`. A routed moudle `FooModule` should have a routing module `FooRoutingModule`. It defines routes, guards and resolver servcie providers. It should re-exports the `RouterModule` thus its corresponding routed mdoule can use route services. To configure routes, the `AppModule` calls `RouterModule.forRoot(routes)` and a feature module calls `RouterModule.forChid(routes)`. A routing module should not have declarations.
- Service module: a service module provides utility services such as data access. It should only has providers and have no declarations. The root `AppModule` is the only module that should import service modules. If a service is only used locally in a feature module, then it should be defined as a service class, not in a module, inside the feature module folder.
- Widget module: a widget module defines shareable components, directives, and pipes for other feature modules. It should only have exported declarables and should not have providers.

The following table summarizes the module types.

| Type    | Declarations | Providers    | Exports        | Imported by   |
| ------- | ------------ | ------------ | -------------- | ------------- |
| Root    | Only root    | No           | No             | None          |
| Eager   | Yes          | Rare         | Top components | Root, Feature |
| Routed  | Yes          | Rare         | No             | None          |
| Routing | No           | Yes (Guards) | RouterModule   | Root, Feature |
| Service | No           | Yes          | No             | Root          |
| Widget  | Yes          | Rare         | Yes            | Feature       |

The "Rare" means that noramlly you should not provide definitions in that type.

To prevent a service module provided by a lazy loading moudle, use the following code in a service module:

```ts
constructor (@Optional() @SkipSelf() parentModule: CoreModule) {
  if (parentModule) {
    throw new Error(
      'CoreModule is already loaded. Import it in the AppModule only');
  }
}
```

A module whose class defined with the above constructor will throw an exception when it is provided more than once.

It is not recommended that a module provides services and declares declarables. If that happens, such as the `RouterModule`, use `forRoot()` to provide and config services for the root module and `forChild()` for other modules.

### 解决 Module 冲突

出现场景：

- 在订单处理模块中有一个国内机票模块，我们命名为：
  - 文件相对路径以及文件名：`./flight/flight.module.ts`
  - `Module` 名： `FlightModule`
- 在基础数据模块中同样有一个国内机票模块，命名也和订单处理一致。
- 在订单处理模块和基础数据模块的 `RoutingModule` 中使用懒加载：

```ts
  {
    path: 'flight',
    loadChildren: './flight/flight.module#FlightModule',
  },
```

那么就会出现下面的错误提示：

```bash
ERROR in Duplicated path in loadChildren detected: "./domestic-flight/domestic-flight.module#DomesticFlightModule" is used in2 loadChildren, but they point to different modules "(/Users/vm/Workspace/Webs/Angular/tehang-system/src/app/routes/order/domestic-flight/domestic-flight.module.ts and "/Users/vm/Workspace/Webs/Angular/tehang-system/src/app/routes/basic-resource/domestic-flight/domestic-flight.module.ts"). Webpack cannot distinguish on context and would fail to load the proper one.
```

通过错误提示，我们可以得知问题的原因是由于 `Webpack` 无法正确的区分它们。  
那么解决办法有两个：

- 在 `RoutingModule` 中的 `loadChildren` 添加一个父级路径，例如：

```ts
  // 基础数据模块的 `RoutingModule`
  {
    path: 'flight',
    loadChildren: '../basic-resource/flight/flight.module#FlightModule',
  },
```

- 把其中一个国内机票模块改个名字(不仅仅是改 `FlightModule` 这个名字, 还要改文件名)

我们建议使用第一种解决方式，理由是：① 重新命名需要修改的地方较多。 ② 命名是编程两大难题之一。

## Architecture: Smart Components and Presentational Components

The overall Angular application could be organized into two types of components: [smart components and presentational components](https://blog.angular-university.io/angular-2-smart-components-vs-presentation-components-whats-the-difference-when-to-use-each-and-why/).

- Smart component: aslo called stateful component or container component. A smart component usually retrieves data using services and may use presentational components to display data.
- Presentational components: also known as pure component or stateless component. It receives its data from parent smart component using `@Input()` binding. It uses `@Output()` event to communicate with its parent component.

Presentational component should only use `@Input()` and `@Output()`. If there are multiple level between a smart component and a presentational component, use RxJS's `subject` to communicate via a service. This should be carefully documented in all components involved in the communication.

The `ngrx/store` is not recommended because the Angular's dependency injection and RxJS can solve most problems in a more elegent way.

## Change Detection Strategy

It is recommended to use the `ChangeDetectionStrategy.OnPush` strategy for better performance. Correspondingly, use `@Input` with immutable objects and the `async` pipe in components.

The default change detection strategy `ChangeDetectionStrategy.Default`runs change detection for all possible operations, thus it is not efficient in many cases. When use `ChangeDetectionStrategy.OnPush` change detection strategy, Angular runs change detection only in two cases: 1) The `@Input` reference changes (the `===` comparison changes), and 2) A DOM event such as a button click originated from the component or one of its children. It ignores all timers, promises and HTTP events. An exception is that when you use the `async` pipe to display data from asynchrounous operations like HTTP requests or promises, the `async` pipe marks the component to be checked by calling the `ChangeDetectorRef.markForCheck()` method.

Use asynchronous methods to update properties and know the `NgZone` concept to avoid the `ExpressionChangedAfterItHasBeenCheckedError` error.

## Constructor and Lifecycle Hooks

For property bindings and change detection theories, check [these blogs](https://blog.angularindepth.com/these-5-articles-will-make-you-an-angular-change-detection-expert-ed530d28930).

Simply, follow the following rules:

- Use `constructor()` only for dependency injection.
- Use `ngOnInit()` to initialize data-bound properties or subscribe to third-party widget events.
- Use `ngDestroy()` to clear resources such as unsubscribing from observables.

Use the other hooks only if you fully understand the consequences. Avoid the following three because they run in all possible UI and asynchronous events.

- Use `ngOnChanges()` to handle any data-bound property of a directive changes
- Use `ngAfterViewInit()` when you need to do something after the component view is initialized. The `@ViewChild` fields are initialized when this hook is called.
- Use `ngDoCheck()` if you want to track self-component peroperties and calculated properties. For example, `ngDoCheck() { this.time = Time.getCurrentTime() }`. It ocurrs evey time there is a possible change event such as button clicked, promise completed or http response received. The method must be very quick because it happens a lot.
- `ngAfterContentChecked()` occures after `ngDoCheck()` and when Angular finishes checking its projected contents -- triggered under the same condition as the `ngDoCheck()`. Should be careful when use this method. This is the last chance that you can change component view data after each change detection run because it happens before view is rendered. But you cann't change the projected contect properties becaue it happens after the content is checked. `ExpressionChangedAfterItHasBeenCheckedError` is throwed if content property is changed in or after the `ngAfterContentChecked()` hook.

## Forms

Only use the reactive Form, not Template Form. Template forms are asynchornous and event-driven. They are also not as flexible as reactive forms.

For most simple scenarios, use `FormBuilder` to create a form. For a complicated form, create `FormGroup` and `FormControl` directly.

```ts
constructor(private formBuilder: FormBuilder) {}

ngOnInit() {
  this.createForm()
}

createForm() {
  this.myForm = this.formBuilder.group(...)
}
```

### 普通表单

对于一些有一定交互的表单，我们建议使用普通的表单去构建，通过在对应的`component`里面去写交互逻辑，推荐使用ng-alain提供的`se`组件，需要注意的是`se`组件需要搭配指令`se-container`使用，否则会在运行时报错。

`se`的用法如下：

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'components-edit-horizontal',
  template: `
  <form nz-form #f="ngForm" se-container="1" labelWidth="150">
    <se label="App Key" error="请填写" optional="(选填)" optionalHelp="通过控制台-查看KEY获取" extra="额外提示信息">
      <input type="text" nz-input [(ngModel)]="i.ak" name="ak" required>
    </se>
    <se label="App Secret" error="请填写，最多32位">
      <input type="text" nz-input [(ngModel)]="i.sk" name="sk" required maxlength="32">
    </se>
    <se>
      <button nz-button nzType="primary" [disabled]="f.invalid">Save</button>
    </se>
  </form>`,
})
export class ComponentsEditHorizontalComponent {
  i: any = {};
}
```

### 动态表单

> [参考资料](https://ng-alain.com/form/getting-started/zh)

对于后台系统的查询页面而言，大部分都是比较规整，为了减少重复编写模板代码，我们建议采用动态表单，详细资料可以参考angular[官方文档](https://angular.cn/guide/dynamic-form)和[ng-alain](https://ng-alain.com/form/getting-started/zh)。因为我们决定大量使用ng-alain所带来的特性，同样我们也决定采用有它提供的基于[JSON Schema](http://json-schema.org/)标准构建的动态表单

**基本原则：**

没有复杂交互的表单都可以采用这种动态表单，如果涉及复杂的交互建议还是使用普通的表单，另外，禁止在动态表单中去写`if...else...`或者其他逻辑性代码，只能写声明式代码。

用法如下：

```ts
@Component({
    selector: 'app-home',
    template: `
      <sf [schema]="schema" (formSubmit)="submit($event)"></sf>
    `
})
export class HomeComponent {
  schema: SFSchema = {
    properties: {
      email: {
        type: 'string',
        title: '邮箱',
        format: 'email',
        maxLength: 20
      },
      name: {
        type: 'string',
        title: '姓名',
        minLength: 3
      }
    }
  };

  submit(value: any) {

  }
}
```



一些建议：

- 针对复杂表单（表单项的验证规则是可变的情况），使用 `from.valid/invalid` 判断时，
  - 需要注意是否是最新的状态，因为可能某个表单项改变了验证规则，但是没有更新 from 状态.
  - 或者改变任意一项，请使用 `control.updateValueAndValidity()` 以保证 form 状态的正确性
- 对于大范围改变表单验证规则的情况，可以考虑重新 init form, 但需要小心（注意 html 中需要用 ngif 把不需要的表单 remove 掉）.
- `formGroup.value`属性是获得 `FormGroup`的 `value` 的最好方法，因为它排除了 `disabled controls`。
- 如果特殊场景需要获得 `disabled controls` 的值，需要使用 `formGroup.getRawValue()` 。
- 如果一个验证规则被重复多次使用，请使用类似 `const pattern = Validators.pattern(moneyRegex)` ,让代码变得更简洁.
- 表单 `control` 统一使用 `touched` 进行错误提示。

## 表格

表格是后台经常会使用的功能，为了简化表格的写法，增加可维护性，我们推荐使用ng-alain提供的[st组件](https://ng-alain.com/components/table/zh)

简单示例如下：

```ts
import { Component } from '@angular/core';
import { STColumn } from '@delon/abc';

@Component({
  selector: 'components-table-basic',
  template: `
  <st [data]="url" [req]="{params: params}" [columns]="columns"></st>`,
})
export class ComponentsTableBasicComponent {
  url = `/users?total=2&field=list`;
  params = { a: 1, b: 2 };
  columns: STColumn[] = [
    { title: '编号', index: 'id' },
    { title: '头像', type: 'img', width: '50px', index: 'picture.thumbnail' },
    { title: '邮箱', index: 'email' },
    { title: '电话', index: 'phone' },
    { title: '注册时间', type: 'date', index: 'registered' },
  ];
}
```

表格中需要用到的CheckBox，RadioButton, 点击, 分页等等都由`change`事件统一管理，通过`eventType`去判断对应的类型，切记组件里面要去定义change的处理回调把一些不需要的事件排除，避免发不需要的请求。

示例如下：

```ts
class DemoTable {
  onChange(e: STChange) {
    const eventType = e.eventType
    if(eventType === 'ps' || eventType === 'pi') {
      this.query()
    } else if(eventType === 'checkbox') {
      // do something
    }
  }
}
```

## Dependency Injection

- Use `providedIn: 'root'` for all services, either global or local, which should be available as singletons.
- Use file structure to scope services. A module can only use services that are in its subfoler or share the same parent folder.
- Use `providers: []` for services that 1) need initialization (such as `myService.forRoot()`) or 2) inside `@Component` or `@Directive` for scoped mulitple-instance services.

## RxJS 数据流和 Spin

在 UI 层，RxJS 有二种常见的数据获取方式，

- 用代码的`source$.subscribe(data=>...)`。这种方式的最大好处是对数据的处理非常灵活，可以分别处理网络层和应用层的错误。缺点是在 `OnPush` 的变化检查策略时要做额外的工作。我们建议采用这种方式，依靠下面的统一代码模式保证正确性。
- 用 HTML 里的`async`管道： `source$ | async` 这种方式有二个优点：1 支持 `ChangeDetectionStrategy.OnPush` 2 自动释放资源,不需要手动 unsubscribe。缺点是数据处理不够灵活。

### `Subscribe` 方式

为了高效，以下代码假设都采用 `OnPush` 的 change detection 策略。`changeDetectionRef` 是注入的 `ChangeDetectionRef`。如果采用缺省的 change detection 策略， 则不需要 markForCheck()。

```ts
this.service.getData()
  .pipe(finalize(() => this.changeDetectionRef.markForCheck())
  .subscribe(data => this.onSuccess(data), error => this.handleError(error))
```

下面给出当运行后台任务时需要显示 Spinner 或 loading 信息的例子。

```ts
// recommended
this.startSpin() // 当前页面事件会触发 change detection, 开始spin。否则此处也需要 markForCheck()
this.service
  .getData()
  .pipe(finalize(() => {
    this.stopLoading()
    this.changeDetectionRef.markForCheck()})
  .subscribe(data => this.onSuccess(data), error => this.handleError(error))
```

### `unsubscribe` 方式

- Angular 内置的 Observable 是不需要我们手动 unsubscribe 例如：

  - HttpClient
  - Router
  - ActivatedRoute
  - html 中的 async 管道

- 使用 Subject + takeUntil 去 unsubscribe

  ```ts
  export class XxxComponent implements OnInit, OnDestroy {
    data1: any
    data2: any
    private unsubscribe$ = new Subject()
    constructor(private xxxService: XxxService) {}

    ngOnInit(): void {
      this.xxxService
        .getData1()
        .pipe(takeUntil(this.unsubscribe$))
        .subscribe(data1 => {
          this.data1 = data1
        })

      this.xxxService
        .getData2()
        .pipe(takeUntil(this.unsubscribe$))
        .subscribe(data2 => {
          this.data2 = data2
        })
    }

    ngOnDestroy(): void {
      // 这里的 unsubscribe$ 不再需要 complete 或者再次 unsubscribe
      // 因为它是 Component 的一个 property 会自动释放
      this.unsubscribe$.next()
    }
  }
  ```

## Components 风格指南

### 成员顺序

- 属性成员 => constructor => 生命周期钩子 => 自定义方法
- 属性成员： @Input/Output =>  `public` 成员 => `private` 成员
  - 相关的成员应该放在一起，例如： `spinTip` 和 `isSpinning` , `STData`、`STColumn` 和 `STComponent`
- constructor中的依赖注入顺序： `default` => `public` => `private`
- 生命周期钩子：按执行顺序排序
- 自定义方法：关键的和 `public` 修饰的放在前面
- 注意：当组件包含大量表单元素，需要使用 `getter` 时，请放到最后并使用 `// #region controls` 与 `// #endregion` 包裹

  ```ts
    // #region controls
    get bunkCode() {
      return this.form.get('bunkCode')
    }
    get discount() {
      return this.form.get('discount')
    }
    // #endregion
  ```

## Templates 风格指南

- **Template**保持足够简单，尽量避免计算以及表达式，如若需要，可将其移到**Component**里面使用计算属性表示：

  ```ts
  export class DemoComponent {
    /// 此处略去其他代码

    get isVisible() {
      return this.list.length > 0 && balabala……
    }
  }
  ```

- 对于后端返回的时间格式，不要滥用 date pipe 来对时间进行格式转换，只有在标准[ISO 8601]格式下才能使用 date pipe, 如果是其他格式的字符串（例如： yyyy-MM-dd HH:mm）, 直接使用 date pipe 的话在 IE 和 Safari 上会存在兼容性问题。