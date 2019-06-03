# RxJS重试之retry、retryWhen、catchError、repeat、repeatWhen

最近处理业务时，在符合某种条件的情况下需要进行一次请求重发。RxJS 能够实现重试的操作符有 3 种：

- retry
- retryWhen
- catchError

`retry`、`retryWhen`、`catchError` 都属于 Error Handling 的操作符，通过**捕获异常**来实现重试。此外还有两个操作符 `repeat` ，`repeatWhen` 能够执行重复操作，下面对这 5 个操作符进行一波实战。

## 准备工作

简单实现一个请求函数，模拟调用接口。 [点击查看Angular HttpClient 的实现](https://github.com/angular/angular/blob/master/packages/http/src/backends/xhr_backend.ts)

```ts
import { Observable, Observer } from "rxjs";

export const SUCCESS = 200
export const quest = (param: { code: number }) => {
    return new Observable<string>((observer: Observer<string>) => {
        log('questing')
        setTimeout(() => {
            if (param.code === SUCCESS) {
                observer.next('ok')
                observer.complete()
            } else {
                observer.error('error')
            }
        }, 2000)
    })
}
```

以及 log 函数

```ts
export const log = function (...args: any[]) {
    console.log.apply(console, args)
}
export const logCompleted = () => {
    log('completed')
}
```

## 操作符

### repeat

```ts
public repeat(count: number): Observable
```

重复 `count` 次由源 Observable 所发出的项的流。[官方文档](https://cn.rx.js.org/class/es6/Observable.js~Observable.html#instance-method-repeat)

```ts
const param = {code:200}
quest(param)
    .pipe(
        repeat(2)
    )
    .subscribe(log, log,logCompleted) 

// 执行结果：
// questing
// ok
// questing
// ok
// completed
```

值得注意一下的是，当全部的 repeat 执行完之后 Observable 变为 completed，尽管在 `quest` 中 `observer.complete()` 是紧跟着 `observer.next('ok')`  之后的。

如果 `count` 为0则产生一个空的 Observable （立即完成的 observable）。

### repeatWhen

```ts
public repeatWhen(notifier: function(notifications: Observable): Observable): Observable
```

根据 `notifier` 返回的 Observable 来决定是否重复。 [官方文档](https://cn.rx.js.org/class/es6/Observable.js~Observable.html#instance-method-repeatWhen)

```ts
// 模拟出 repeat 效果
const param = {code:200}
quest(param)
    .pipe(
        repeatWhen(result$ => result$.pipe( //result$ 
            scan(count=>{
              return count + 1
            },0),
            takeWhile(count => {
              return count < 2 
            })
           
        ))
    )
    .subscribe(log,log,logCompleted)
// 执行结果：
// questing
// ok
// questing
// ok
// completed
```

`repeatWhen` 必然会执行 1 次。看下 notifications 出现错误时的输出结果。

```ts
quest(param)
  .pipe(
  repeatWhen(result$ => result$.pipe(
      tap( _=>{
        throw 'repeatWhen error'
      })
    )) 
  )
  .subscribe(log,log,logCompleted)
// questing
// ok
// repeatWhen error
```

### retry

```ts
retry(count: number): Observable
```

返回一个 Observable， 该 Observable 是源 Observable 不包含错误异常的镜像。 如果源 Observable 发生错误, 这个方法不会传播错误而是会不断的重新订阅源 Observable 直到达到最大重试次数 (由 `count` 参数指定)。[官方文档](https://cn.rx.js.org/class/es6/Observable.js~Observable.html#instance-method-retry)

```ts
const param = {code:100}
quest(param)
    .pipe(
         retry(2)
    )
    .subscribe(log, log,logCompleted) 
// questing
// questing 
// questing
// error
```

### retryWhen

```ts
public retryWhen(notifier: function(errors: Observable): Observable): Observable
```

根据 `notifier` 返回的 Observable 来决定是否重试，用法和 `repeatWhen` 类似。[官方文档](https://cn.rx.js.org/class/es6/Observable.js~Observable.html#instance-method-retryWhen)

```ts
let param = {code: 200}
let flag = false
quest(param)
    .pipe(
        tap(_ => {
            if (!flag) throw 'reload'
        }),
        retryWhen(err$ => err$.pipe(
            takeWhile(err => {
              if(err === 'reload'){
                return true
              }else
                throw err
            }),
            tap(_ => {
                flag = true
            }),
        ))
    )
    .subscribe(log, log, logCompleted)
// questing
// questing
// ok
// completed
```

要注意一下的是，一定要清楚上游有哪些 Error ，符合条件的进行 `retry` 不符合条件的要继续 `throw`。

### catchError/catch

```ts
public catch(selector: function): Observable
```

捕获 observable 中的错误，可以通过返回一个新的 observable 或者抛出错误对象来处理。[官方文档](https://cn.rx.js.org/class/es6/Observable.js~Observable.html#instance-method-catch)

`selector` 函数接受两个参数：

- `err` ，错误对象，
- `caught` ，源 Observable，当你想“重试”的时候返回它即可。

```ts
let param = {code: 200}
let flag = false
quest(param)
    .pipe(
        tap(_ => {
            if (!flag) throw 'reload'
        }),
        catchError((err,catch$) => {
              if(err === 'reload'){
                flag = true 
                return catch$
              }else
                throw err
        })
    )
    .subscribe(log, log, logCompleted)
// questing
// questing
// ok
// completed
```

效果和 `retryWhen` 一样，代码更简单，不需要考虑重试次数时可以考虑。

## retry 源码

`retry`、`retryWhen`、`catchError` 可以对错误重试，想要通过它们实现非错误重试，需要伪造一个 Error 以及额外判断。在非错误的情况下，可以进行重试吗？几经验证，没行得通 `this._unsubscribeAndRecycle()` 在 `_next` 函数中使用后，不能正常工作，具体原因未查明。又或者这个方案本身就存在问题，留待以后研究。

```ts
import { Subscriber } from 'rxjs/Subscriber';

export function retry(count = -1) {
		// 这里改了下
   // return this.lift(new RetryOperator(count, this));
    return function (source) {
        return source.lift(new RetryOperator(count, source));
    }
}
class RetryOperator {
    constructor(public count, public source) { }
    call(subscriber, source) {
        return source.subscribe(new RetrySubscriber(subscriber, this.count, this.source));
    }
}
class RetrySubscriber extends Subscriber {
    constructor(destination, public count, public source) {
        super(destination);
    }
    error(err) {
        if (!this.isStopped) {
            const { source, count } = this;
            if (count === 0) {
                return super.error(err);
            }
            else if (count > -1) {
                this.count = count - 1;
            }
            source.subscribe(this._unsubscribeAndRecycle());
        }
    }
}
```

## 参考链接

[演示地址](https://stackblitz.com/edit/rxjs-y4ofkh)

[创建操作符 | operator-creation](https://cloud.tencent.com/developer/section/1489395)