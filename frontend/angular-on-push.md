# OnPush 尝试

```ts
// 一个简单的查询列表 下面几种尝试的公共代码
  query() {
    const queryParam = this.getQueryParam()
    this.startSpin('正在查询中,请稍候...')
    this.httpService
      .getCorpWriteOffs(queryParam)
      .pipe(finalize(() => this.stopSpin()))
      .subscribe(
        //... 省略
      )
  }
```

```html
<!-- 页面上会有两个地方用到 isSpinning, 如果是异步数据流的时候 会加上 | async-->
<button (click)="simpleTable.reset()" [disabled]="isSpinning">查询</button>

<nz-spin [nzTip]="spinTip" [nzSpinning]="isSpinning"> </nz-spin>
```

## 默认模式下

```ts
  _isSpinning = false
  count = 0
  get isSpinning() {
    console.log(`get isSpinning count : ${this.count++}`)
    return this._isSpinning
  }
  set isSpinning(isSpinning: boolean) {
    this._isSpinning = isSpinning
  }
```

- 页面渲染完成： 调用了 171 次
  get isSpinning count : 170
  get isSpinning count : 171

- 然后每 2-3s 一次轮询 每次调用 4 次

- 每次切换 table 页 调用 60 次

优点： 不用我们操心数据和页面渲染的关系，完全交给 Angular

缺点：渲染页面时执行了太多次的数据获取，渲染完成后仍然不停的轮询数据，另外每次 table 页切换也调用多次，性能损失严重

## 开启 OnPush

### 使用同步数据

```ts
  _isSpinning = false
  count = 0
  get isSpinning() {
    console.log(`get isSpinning count : ${this.count++}`)
    return this._isSpinning
  }
  set isSpinning(isSpinning: boolean) {
    this._isSpinning = isSpinning
  }
  constructor(private cd: ChangeDetectorRef) {}

  ngDoCheck() {
    // 不做任何优化 直接调用 markForCheck
    this.cd.markForCheck()
  }
```

- 页面渲染完成 调用了 85 次
  get isSpinning count : 84
  get isSpinning count : 85

- 然后每 2-3s 一次轮询 每次调用 2 次
- 每次切换 table 页 调用 15 次

优点： 依旧不用我们操心数据和页面渲染的关系，在 ngDoCheck 去执行 markForCheck() 没有做任何优化， 让 Angular 帮我们处理： 性能提升 1 倍（当然 如果我们做了手动优化 性能会提升更多）。

缺点：如果完全不手动优化，性能提升有限，但是如果手动优化 比较繁琐

### 使用异步数据流

```ts
_isSpinning$ = new BehaviorSubject<boolean>(false);
count = 0;
isSpinning$ = this._isSpinning$.pipe(
  map(value => {
    console.log(`get isSpinning count : ${this.count++}`);
    return value;
  })
);
```

- 页面渲染完成 调用了 3 次
  get isSpinning count : 3
- 不再执行自动轮询 0 次
- 每次切换 table 页 调用 4 次

优点： 性能提升巨大！！！ ！！！而且通过异步数据流的话 也不需要我们手动去做什么优化

缺点： 必须使用异步数据流

## 感受

我只想说！ OnPush 大法好！！！  
但是 OnPush 在带来性能的同时也带来了挑战：

- 它要求我们对 Rxjs 的掌握得更好
- 当无法使用异步数据流的时候 处理数据需要更谨慎

我接下来在空闲的时候会针对下面几点进行学习，希望大家有空的时候也多看看：

- 了解 zone.js 是什么，干什么用的。
- 了解 angular 如何做变更检查。
- 了解 OnPush 是如何进行优化的
