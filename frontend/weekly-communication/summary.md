# 每周前端小组内部沟通内容  汇总

## 2018-11-30

### 简单查询列表展示页

可以使用统一模板：[simple-query-page](./simple-query-page.md)

### 慎用继承

继承有如下几个缺点：

- 父类的内部细节对子类是可见的
- 如果对父类的方法做了修改的话（比如增加了一个参数），则子类的方法必须做出相应的修改。

> 继承会带来很多不必要的东西，建议多使用组合，组合也就是设计类的时候把要组合的类的对象加入到该类中作为自己的成员变量。

组合的优点：

- 当前对象只能通过所包含的那个对象去调用其方法，所以所包含的对象的内部细节对当前对象时不可见的
- 当前对象与包含的对象是一个低耦合关系，如果修改包含对象的类中代码不需要修改当前对象类的代码

当然组合也是有缺点的：

- 容易产生过多的对象
- 为了能组合多个对象，必须仔细对接口进行定义

### 谨慎抽象

> 过度抽象很有可能在后期需求的改动时，需要重新修改之前的封装才能完成复用，导致最终成本实际上还不如不做；或者你发现复用的部分所降低的成本实际上还不如包装花费的成本。与之相对的是设计不足。

建议使用的方式是，TDD（测试驱动开发），核心思想是小步增量，不断重构。我们正处在尚未很熟悉 Angular 和系统业务的状态，这种方式很适合我们。

### 慎用 ngOnChanges

> 使用时需要注意下面两点

- 它会在 ngOnInit 前调用，官方有可能会在下个版本修改此行为，使用时需注意

- 为了提高变化检测的性能，对于对象比较，Angular 内部直接使用 `===` 运算符进行值比较。因此当输入属性是引用类型，当改变对象内部属性时，是不会调用 `ngOnChanges` 生命周期钩子的

### 复杂表单验证需小心

> 统一使用 reactive form, 使用时需要注意以下几点.

- 针对复杂表单（表单项的验证规则是可变的情况），使用 from.valid/invalid 判断时，需要注意是否是最新的状态，因为可能某个表单项改变了验证规则，但是没有更新.
- 对于大范围改变表单验证规则的情况，可以考虑重新 init form, 但需要小心（注意 html 中需要用 ngif 把不需要的表单 remove 掉）.
- 把定义和获取 control 的代码放到最后 `get controlName() { return this.form.get('controlName')}`, 以更方便 review .
- 如果一个验证规则被重复多次使用，请使用类似 `const pattern = Validators.pattern(moneyRegex)` ,让代码变得更简洁.

## 2018-12-07

- 使用插件 document this 进行代码注释

- 表单 control 统一使用 touched 进行错误提示

- 使用 Object.keys(xxx).forEach() 代替 for in

- 简单查询条件 form 统一使用 shf-item 组件，较复杂（需要我们自己控制布局 长短的）统一使用 nz row + nz col

- spin 统一使用 BehaviorSubject, 定义一个 showSpin/hideSpin 方法， 如果需要改变 nzTip 则 showSpin(nzTip)

- Component 中 方法名可以简单（query add update delete），但是必须有注释

- Http Service 中方法名参照 Api 命名， 且必须有注释。命名风格 xxxApi

- 统一使用 Router Service 进行页面跳转，并且保证 一个 module 一个 router

- 单一实体服务用 root，各个模块或组件直接 constructor 注入，但必须保证只能用本目录或者父级目录的 service ， 如果需要跨目录使用，请提取到相应级别的 Shared 目录。如果不是单例的 service , 则用 provider 和 constructor 注入。
