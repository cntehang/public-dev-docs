---
layout: post
title: 使用 Service + Rxjs 进行 Angular 的状态管理
date: 2019-03-01
author: Vm
catalog: true

tags:
  - Angular
  - Rxjs
---

# 使用 Service + Rxjs 进行 Angular 的状态管理

在 React 和 Vue 的应用中，状态管理库似乎是全家桶中必不可少的一环，其中以 Redux 和 Vuex 最为常见。  
虽然 Angular 的第三方库中也有一个类似的 Ngrx, 但是却处于一个可有可无的地位。

## 为什么在 Angular 中，状态管理库不是必须的呢？

因为 Angular 中内置了两大利器： Service 和 Rxjs

- 在 Angular 我们只要定义了一个 Service， 就可以通过依赖注入的方式在组件中使用它。
  - 这个 Service 通常都是单例的，我们把数据保存在 Service 中，那么组件间很轻松的就能共享数据。
  - 我们可以把与数据相关的处理定义在 Service 中，交给组件的数据就是组件最终渲染的数据。
  - 可以参考官方示例[英雄编辑器](https://stackblitz.com/angular/vkglbnmmbojm?file=src%2Fapp%2Fhero.service.ts) 中的 `HeroService`, 多个组件都注入了它，去获取同一份英雄列表的数据。

如果仅仅只有 Service 的话，那也是不够用的，举个栗子: 一个 input 输入框组件，根据输入内容请求接口，查询回数据后传递给一个 table 组件进行渲染。  
这个需求，我们需要小心点几点有：

- 防抖和确认输入内容是否发生改变，这是为了给服务端降低压力，我们需要减少没必要的请求。
- 网络请求相应的不稳定性，我们需要保证最终 table 组件渲染在页面上的数据是最后一次请求响应的结果。
- 如何把最终的数据传递给 table 组件。

针对上面3点我们可能需要做的事情有：

- 定义 防抖函数 或者使用第三方库，例如：underscore.debounce()
- 比较前后两次value的函数:

  ```ts
  let preValue
  function isChange(value) {
    if(preValue !== value){
      preValue = value
      return true
    }
    return false
  }
  ```

- 要保证请求结果的准确性的话，搜索到的几种解决方案都比较 hack，或者直接牺牲用户体验(在发起请求的时候，就不允许用户输入)。
- 可以使用 @Output 到父组件接收，然后 table 组件通过 @Input 来获取。

上面的解决方案中的代码有这么几个缺点：

- 要实现保证请求结果的准确性的话，要么牺牲用户体验，要么实现复杂度较高
- 需要定义一些易被污染的变量(preValue)
- 耦合了父组件(input 和 table 必须在同一个父组件内)

那么如何优雅的解决上述的问题呢，这个时候我们的另一个利器是时候展现它真正的技术了。

```ts
// data.service.ts
import { BehaviorSubject, Observable } from 'rxjs'
class DataService{
  private data$ = new BehaviorSubject<DataType>()
  updateData(data){
    this.data$.next(data)
  }
  getData():Observable<DataType>{
    return this.data$.asObservable()
  }
}

// input.component.ts
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';
import { DataService } from './data.service'
class InputComponent{
  constructor(private dataService:DataService){}
  this.input.valueChanges.pipe(
        debounceTime(300), // 防抖
        distinctUntilChanged(), // 确定发生变化
        switchMap((value: string) => this.httpClient.post(api,value)),// 取消前一次的请求结果 发起一次新的请求
      )
      .subscribe(data => this.dataService.updateData(data)) // 拿到最终请求结果 通知data service 更新数据
}

// table.component.ts
import { DataService } from './data.service'
class TableComponent{
  constructor(private dataService:DataService){}
  this.data$ = this.dataService.getData()
}
```

在上面的代码中:

- 定义了一个 Service ,利用 Rxjs 的 [BehaviorSubject](https://www.learnrxjs.io/subjects/behaviorsubject.html) 的特点, 在保存数据的同时，还可以对外提供一个可订阅的对象用于获取数据。
- 在 InputComponent 中使用了(debounceTime,distinctUntilChanged,switchMap)等内置操作符来达到我们对请求的优化和保证数据的正确性。
  - debounceTime: 舍弃掉在两次输出之间小于指定时间的发出值
  - distinctUntilChanged: 当前值与之前最后一个值不同时才将其发出
  - switchMap: 映射成 observable，完成前一个内部 observable，发出值。
    - 可以取消上一次订阅,保证网络请求结果的准确性
- 在 TableComponent 中只要注入一下 DataService 就可以轻易的拿到我们想要的数据。
- 假设后续有另一个 ListComponent 也需要这一份数据，不用修改之前的任何代码，只需要新建在 ListComponent 中注入 DataService 然后获取数据即可。

代码简洁，链式调用，组件解耦，易于维护...总之，吹爆。  
这二者的结合基本满足了我们在实际开发过程中对于状态管理与组件通信的需求。

## 总结

在 Angular 应用中，通过 Service 来管理状态，把需要共享的数据通过 Observable 包装起来，提供相应的订阅接口即可。这样的状态流非常的简单清晰，易于维护。  
对于组件通信，在 Angular 中有多种方式，我建议的方式是：

- 对于父 => 子，使用 @Input()
- 对于子 => 父，使用 @Output() EventEmitter。(其实 EventEmitter 就是一个 Observable 对象)
- 对于其他情况，在没有特殊需求的条件下，尽可能的使用 Service + Rxjs 的方式通信，让组件解耦。