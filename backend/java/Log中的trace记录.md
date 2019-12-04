# 系统 Log 内的 trace 记录

系统中采用了 sleuth+zipkin 进行日志的 trace 记录，在 trace 记录中大致会遇到以下两种情况：

## 从 API 层面进来的请求

此时请求内部已经生成了 traceID，spanID 等需要的信息，不许做额外处理，仅需正常记录日志即可。

## 非外部请求，内部定时器等自启动线程/任务的 trace 记录

这类线程/任务的启动由系统内部自定控制，在执行前，并没有生成相应的 traceID 等，此时需要手动生成以下:

```java
// 需注入此对象
  private Tracer tracer;

  public void execute() {
    // 开启一个新的span
    ScopedSpan span = tracer.startScopedSpan("newSpanName");
    try {
      LOG.debug("start *** task");
      // do the work you need
    } finally {
      // 每次任务结束，请及时调用finish方法，否则会一直在一个span内
      span.finish();
    }
  }
```
