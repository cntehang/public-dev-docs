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
| Root    | No           | No           | No             | None          |
| Eager   | Yes          | Rare         | Top components | Root, Feature |
| Routed  | Yes          | Rare         | No             | None          |
| Routing | No           | Yes (Guards) | RouterModule   | Root, Feature |
| Service | No           | Yes          | No             | Root          |
| Widget  | Yes          | Rare         | Yes            | Feature       |

The "Rare" means that noramlly you should not provider definitions in that type.

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
- Use `ngOninit()` to initialize data-bound properties or subscribe to third-party widget events.
- Use `ngDestroy()` to clear resources such as unsubscribing from observables.

Use the other hooks only if you fully understand the consequences:

- Use `ngAfterViewInit()` when you need to do something after the component view is initialized. The `@ViewChild` fields are initialized when this hook is called.
- Use `ngOnChanges()` if you want to track parent bound `@Input` properties.
- Use `ngDoCheck()` if you want to track self-component peroperties and calculated properties. For example, `ngDoCheck() { this.time = Time.getCurrentTime() }`.

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

一些建议：

- 针对复杂表单（表单项的验证规则是可变的情况），使用 `from.valid/invalid` 判断时，需要注意是否是最新的状态，因为可能某个表单项改变了验证规则，但是没有更新.
- 对于大范围改变表单验证规则的情况，可以考虑重新 init form, 但需要小心（注意 html 中需要用 ngif 把不需要的表单 remove 掉）.
- 把定义和获取 control 的代码放到最后 `get controlName() { return this.form.get('controlName')}`, 以更方便 review .
- 如果一个验证规则被重复多次使用，请使用类似 `const pattern = Validators.pattern(moneyRegex)` ,让代码变得更简洁.
- 表单 `control` 统一使用 `touched` 进行错误提示。

## Dependency Injection

- Use `providedIn: 'root'` for all services, either global or local, which should be available as singletons.
- Use file structure to scope services. A module can only use services that are in its subfoler or share the same parent folder.
- Use `providers: []` for services that 1) need initialization (such as `myService.forRoot()`) or 2) inside `@Component` or `@Directive` for scoped mulitple-instance services.

## 异步操作和 Spin

RxJS 有二种常见的数据处理方式，用`subscribe()`或`pipe()`.

```ts
// 方式一: recommended
this.service.getData().subscribe(data => this.onSuccess(data), error => this.handleError(error))

// 方式二
this.service
  .getAll()
  .pipe(
    map(data => this.onSuccess(data)),
    catchError(error => of(this.handleError(error))),
  )
  .subscribe()
```

建议采用方式一的原因在最后一步分别处理正常和错误数据而不用顾虑处理结果， 避免了方式二为了保证统一输出所需要的`of()`转换。

建议`pipe()`只用于调试，log，延时，错误重试或需要数据转换但不能用`subscribe()`的场合。

当运行后台任务时需要显示 Spinner 或 loading 信息。建议采用如下模式：

```ts
// recommended
this.startSpin()
this.service
  .getData()
  .subscribe(data => this.onSuccess(data), error => this.handleError(error))
  .add(() => this.stopSpin())
```

采用`finalize()`来在正常或错误情况下都停止显示 spinner 或 loacing 信息。此处的`finalize()`在`error`或`complete`之后调用。其效果和如下二种方法一致：

```ts
//
this.startSpin()
const subscription = this.service
  .getData()
  .subscribe(data => this.onSuccess(data), error => this.handleError(error))
subscription.add(() => this.stopSpin())

// 或者
this.startSpin()
this.service
  .getData()
  .pipe(finalize(() => this.stopLoading())
  .subscribe(data => this.onSuccess(data), error => this.handleError(error))
```
