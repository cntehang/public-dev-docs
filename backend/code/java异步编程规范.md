# java异步编程规范

## Async使用规范

在java中推荐的异步编程方式是使用Spring的Async注解，该方式的优点是简单易用，当方法标注了Async注解以后，将在异步线程中执行该方法，但有以下限制：

- Async方法必须是类的第一个被调用的方法；

- Async必须是实例方法，且该方法的对象必须是Spring注入的。

使用时除以上限制需要注意之外，我们还需要解决一些其他问题，如下：

### 1. 日志记录

由于Sync方法并非通过Controller进入，绕开了我们的通用异常拦截层，即CommonExceptionHandler，所以对于异步方法发生的异常没有进行日志记录。

为解决此问题，我们编写了**AsyncExceptionLogger**注解，在所有@Async方法加上此注解，即可保证异常得到记录。

例如：

```java
@Async
@AsyncExceptionLogger
public void asyncMethod() {
  //...
}
```

### 2. 与JPA整合

由于Async方法在新的异步线程中执行，JPA的OpenSessionInView无效，导致执行上下文中并不存在对应的EnitityManager，这会产生一系列问题，典型场景就是导致lazyLoading无效。

为解决此问题，我们编写了**OpenJpaSession**注解，在需要访问数据库的@Async方法中加此注解，即可实现与OpenSessionInView类似的效果。

示例代码：

```java
@Async
@OpenJpaSession
@AsyncExceptionLogger
public void asyncMethod() {
  //...
}
```

### 3. 使用事务

我们当前使用事务的方式是使用@Transactional注解，但由于@Transational只能用在对象被调用的的第一个公有方法上，使用起来多有不便（为了事务而创建一个类实在不能接受），而且在很多情况下，我们都需要更加灵活地进行事务范围的控制。

为此，我们编写了一个辅助类**TransactionHelper**，使用方式如下：

```java
//事务辅助对象，需要注入
private TransactionHelper transactionHelper;

@Async
@OpenJpaSession
@AsyncExceptionLogger
public void createTicketConfirmTaskIfRequired(long orderId) {

  //创建出票任务，在事务中执行
  Long newTaskId = transactionHelper.withTransaction(() ->
      doCreateTicketConfirmTaskIfRequired(orderId)
  );

  if (newTaskId != null) {
    LOG.info("出票任务已创建，开始异步调用出票请求");
    ticketingDomainService.autoTicketingAsync(newTaskId);
  }
}

//需要在事务中执行
private Long doCreateTicketConfirmTaskIfRequired(long orderId) {
  //todo
}
```

我们只需要将事务中的代码以lamda调用的方式包裹在一个withTransaction调用中即可。

### 4. 线程同步

有些场景中，我们需要在主线程中的事务尚未完成的情况下发起一个异步方法调用，在异步方法执行时，又希望在主线程的事务成功提交以后再开始。

例如, 我们需要在机票的所有子订单都出票成功的情况下，才开始发送短信通知，代码如下（以下代码仅用来演示）：

```java
//事务辅助对象，需要注入
private TransactionHelper transactionHelper;
//短信通知服务
private SmsNotifyService notifyService;
//订单仓储
private FlightOrderRepository orderRepo;

//主线程的出票确认操作
public void ticketConfirm(FlightOrder order, FlightTask ticketConfirmTask) {

  //定义一个异步通知事件，用来在主事务提交完成以后通知异步线程开始
  AsyncEvent asyncEvent = new AsyncEvent();

  try {
    transactionHelper.withTransaction(() ->
      //对每个子订单进行出票操作
      order.getOrderTickets().foreach(ticket -> {
        //对子订单作出票处理
        ticket.ticketSuccess();

        //异步发送通知
        notifyService.sendNotifyAsync(order, ticket, asyncEvent);
      });

      //其他操作:更新任务已完成
      ticketConfirmTask.updateToCompleted();
      orderRepo.save(order);
    );

    //事务执行成功，通知异步线程开始
    asyncEvent.setAsCompleted(true);

  } catch (Excepton ex) {
    //事务执行失败，也通知异步线程
    asyncEvent.setAsCompleted(false);
    thrown ex;
  }
}

//SmsNotifyService类
@Async
@OpenJpaSession
@AsyncExceptionLogger
private void sendNotifyAsync(long orderId, long ticketId, AsyncEvent asyncEvent) {
  
  //等待主线程执行成功再开始
  if (asyncEvent.await() && asyncEvent.isSuccessful()) {
    //发送短信通知
    //todo
  }
}
```

为实现线程间的同步，我们设计了一个AsyncEvent类，该类内部包含一个CountDownLatch对象，初值为1。异步线程等待这个CountDownLatch对象，当主线程事务完成以后，调用CountDownLatch的countDown()方法，从而触发异步线程的执行。

AsyncEvent的代码如下（只列出了主要实现细节）：

```java
public class AsyncEvent {

  private boolean successful;
  private CountDownLatch countDownLatch;

  public AsyncEvent() {
    successful = false;
    countDownLatch = new CountDownLatch(1);
  }

  /**
   * 异步线程调用此方法以等待主线程执行完成
   * @param timeout 等待时长
   * @param unit
   * @return 如果主线程在等待时间内执行完成为true, 如果等待超时则为false
   */
  public boolean await(long timeout, TimeUnit unit) {
    try {
      return countDownLatch.await(timeout, unit);

    } catch (InterruptedException ex) {
      throw new BusinessException(String.format("AsyncEvent await exception happens, message: %s", ex.getMessage()), ex);
    }
  }

  /**
   * 设置主线程执行已完成，调用此方法将触发异步线程开始执行。此方法只能调用一次。
   * @param successful 执行成功为true, 否则为false
   */
  public void setAsCompleted(boolean successful) {
    if (this.countDownLatch.getCount() > 0) {
      this.successful = successful;
      this.countDownLatch.countDown();

    } else {
      throw new BusinessException("AsyncEvent.setAsCompleted invoke allowed only once");
    }
  }

  public boolean isSuccessful() {
    return successful;
  }
}
```
