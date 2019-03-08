## 实战演练

产品原型图以及前端组件结构如下所示：
![产品原型图](https://ws1.sinaimg.cn/large/006tKfTcgy1g0ppsonpmpj31y40qbjwt.jpg)

经过分析后，数据状态流向示意图如下：
![数据状态流向示意图](https://ws1.sinaimg.cn/large/006tKfTcgy1g0pptuszkuj30z108uwfr.jpg)

### 状态改变的几个节点

- 发起 Http 请求获取源数据：
  - SearchBar 组件点击查询按钮
  - DateTabs 组件选择新的日期
- 筛选条件过滤处理：
  - 源数据发生改变时
  - 筛选条件发生改变时
    - Filter 组件数据发生变化
    - Sort 组件中的指定数据（仅显示有余票车次）发生变化 （不要问我为什么这里会有这么个条件，我们伟大的产品经理从用户体验的角度考虑的，一切服从产品经理的最高指挥！！！）
- 排序条件进行排序
  - 过滤数据发生改变时
  - Sort 组件数据发生变化
- 最终数据，也就是排序后的数据
  - List 组件订阅数据, 每次得到新数据，就重新渲染。

### Service 的核心结构

```ts
// data.service.ts
import { BehaviorSubject, Observable, combineLatest } from 'rxjs'
import { map } from 'rxjs/operators';
@Injectable({
  providedIn: 'root',
})
export class DataService {
  private searchParams$ = new BehaviorSubject<TrainSearchBo>() // 查询条件状态
  private filterParams$ = new Subject<FilterCondition>() // 过滤条件状态
  private sortType$ = new BehaviorSubject<SortTypeEnum>() // 排序条件状态
  private showTrains$: Observable<TrainInfoBo[]> // 最终数据状态
  private trainDate$  = new BehaviorSubject<Date | undefined>(undefined) // 出发日期，用于 SearchBar 和 DateTabs 通信

  constructor (private httpService:TrainHttpService) {
    this.initShowTrains()
  }

  private initShowTrains(): void {
    const trains$ = this.searchParams$.pipe(siwthMap(params => this.httpService.search(params)))

    const filteredTrains$ = combineLatest(
      trains$,
      this.filterParams$,
    ).pipe(
      map(([trains,filterParams]) => this.filterTrains(trains, filterParams))
    )

    this.showTrains$ = combineLatest(
       filteredTrains$,
       this.sortType$,
    ).pipe(
      map(([filteredTrains,sortType])=> this.sortTrains(filteredTrains, sortType)),
    )
  }

  updateSearchParams(params: TrainSearchBo): void {
    this.searchParams$.next(params)
  }

  updateFilterParams(params: FilterCondition): void {
    this.filterParams$.next(params)
  }

  updateSortType(params: SortTypeEnum): void {
    this.sortType$.next(params)
  }

  getShowTrains(): Observable<TrainInfoBo[]>{
    return this.showTrains$
  }

  updateTrainDate(date: Date): void {
    this.trainDate$.next(date)
  }

  getTrainDate(date: Date): Observable<Date> {
    return this.trainDate$.asObservable()
  }

  private filterTrains() { /* xxx */ }
  private sortTrains() { /* xxx */ }
}
```

- 利用 [Subject](https://www.learnrxjs.io/subjects/subject.html) 的特点, 将**查询参数**、**过滤条件**和**排序条件**转换为3个可观察对象 (observable)。
- 利用 [combineLatest](https://www.learnrxjs.io/operators/combination/combinelatest.html) 操作符，将上述的3个 observable 组合起来，等到每一个 observable 都发出一个值后，combineLatest 首次发出初始值。之后任意一个 observable 发出值，combineLatest 都会发出每个 observable 的最新值。
- 利用 [siwthMap](https://www.learnrxjs.io/operators/transformation/switchmap.html) 操作符拿到参数发起 http 请求获取源数据。
- 利用 [map](https://www.learnrxjs.io/operators/transformation/map.html) 操作符根据过滤条件和排序条件对源数据进行处理，发出最终数据。
- 利用 [BehaviorSubject](https://www.learnrxjs.io/subjects/behaviorsubject.html) 的特点, 在保存数据的同时，还可以对外提供一个可订阅的对象用于获取数据，用于 SearchBar 和 DateTabs 进行通信。

### Components 的核心结构

```ts
// search-bar.component.ts
export class SearchBarComponent{
  form: FormGroup
  private unsubscribe$ = new Subject()

  constructor(private dataService: DataService) {}

  ngOnInit() {
    // 订阅联动日期
    this.dataService.getTrainDate()
      .pipe(takeUntil(this.unsubscribe$))
      .subscribe(
        value=>{
          if(value && value !== this.trainDate.value ){
            this.trainDate.setValue(value, { emitEvent: false })
            this.dataService.updateSearchParam(this.form.value)
          }
        }
      )

    // 日期发生改变 更新 service 中的状态
    this.trainDate.valueChanges.subscribe(
        value=> this.dataService.updateTrainDate(value)
      )
  }

  ngOnDestroy() {
    this.unsubscribe$.next()
  }

  // 点击查询按钮 更新 trainDate 和 searchParam
  onSearchBtnClick() {
    const formValue = this.form.value
    this.dateService.updateTrainDate(formValue.trainDate)
    this.dataService.updateSearchParam(formValue)
  }

  get trainDate(): FormControl {
    return this.form.get('trainDate')
  }
}

// 其他几个组件就不声明了
// DateTabsComponent 和 SearchBarComponent 类似  
// FilterComponent 和 SortComponent 更为简单，监听组件内数据发生变化后，  
// 调用 dataService 相应的update 方法即可，类似上面的 onSearchBtnClick  

// list.component.ts
export class ListComponent {
  showTrains$: Observable<TrainInfoBo[]>

  constructor(private dataService: DataService) {}

  ngOnInit() {
    this.showTrains$ = this.dataService.getShowTrains()
  }

  // 使用 trackBy 优化 ngfor 指令
  trackByTrainNo = (_: number, train: TrainInfoBo) => train.trainNo
}
```

- 可以明显感觉到在各个 Component 内部的逻辑是比较简单的，只要注入 DataService ，然后当自身的状态发生变化时，调用 DataService 相应的 update 方法即可。
- 如果有组件间的通信也是通过订阅 DataService 相应的 get 方法，然后更新自身的状态。
