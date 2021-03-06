# tmc-services项目review小记

## 自动出票

目前自动出票的流程大致是，下单之后会执行“预记账”，“出差审批”，“超标授权”，“自动订座”，每个任务执行完成后需要判断是否需要创建出票任务，如果需要则创建自动，并且会推送一条消息到 MQ 中，tmc 会消费自动出票消息，如果可以自动出票会给资源平台推送一条消息，资源平台拿到这条消息以后，会创建出票任务。定时任务每 10 秒钟会调用自动出票接口，随机选出一个出票任务自动出票

### 下单后的任务处理接口

下单之后的任务处理有些没有考虑幂等性问题，比如当前“超标授权”的接口，如果某个订单最后一个任务是“超标授权”任务，由于网络原因，同一个订单被用户点了两次审批请求，或者其他原因导致有两个针对同一个订单的“审批”请求。在创建自动出票任务时，两个同样的请求执行到`FlightOrder order = flightOrderRepository.findByIdEnsured(orderId);`，两个线程通过拿到的订单信息判断出自动出票任务都没有被创建，就会导致同一个订单创建两个自动出票任务。

```java
  @Transactional
  public Long createTicketConfirmTaskIfRequired(long orderId) {
    LOG.info("Enter createTicketConfirmTaskIfRequired. orderId: {}", orderId);
    Long taskId = null;
    FlightOrder order = flightOrderRepository.findByIdEnsured(orderId);
    LOG.debug("requireTicketConfirm(订单状态，付款状态、PNR状态、审批状态、授权状态)? : {}", requireTicketConfirm(order));
    if (requireTicketConfirm(order) && !taskRepository.existsByOrderIdAndTaskType(order.getId(), TICKET_CONFIRM)) {
      //创建出票任务
      FlightTask task = createTicketConfirmTask(order);
      taskRepository.save(task);
      taskId = task.getId();
    }
    LOG.info("Exit createTicketConfirmTaskIfRequired. taskId: {}", taskId);
    return taskId;
  }
```

### 创建自动出票任务

只有最后一个完成任务的线程会创建自动出票任务，目前 B 方案的设计，如果最后一个任务执行完成，但是创建自动出票任务失败会导致“死单”的出现，可以考虑在订单中增加一个“待出票”状态或者其他方式来解决

### 处理自动出票消息

tmc 在处理自动出票消息通过订单状态来判断解决消息幂等性可能会有问题，但是又不能单纯的通过消息的 id 来处理消息幂等性问题，目前是根据订单状态来判断消息是否被处理过，如果正在处理某个订单的一条自动出票的消息，但是此时这个订单的状态还没被更新，又拿到了一条这个订单的自动出票消息，就会导致这个订单重复出两张票

### 处理自动出票任务消息

目前我们处理自动出票任务是通过定时任务的方式，定时任务使用的是 spring 的 scheduled，这个是单线程的，如果某个任务”阻塞“，会导致后面的任务都延期执行，并且如果任务堆积，目前处理自动出票任务是随机选出一个订单，有可能导致先完成的订单一直都没有被随机到。
一种比较好的处理方式是在消费 tmc 自动出票任务消息的时候就处理自动出票任务，但是这么处理就需要通过控制同一个 group 下的消费者的数量和线程池中线程的数量来控制自动出票任务消息的消费速度

## 代码缓存

代码中缓存使用的比较少，在"运营管理"，"基础数据","系统管理"这些模块很多数据都是配置一次，后期基本不会更新，使用缓存带来的收益会很大。经常查询的接口“查航班信息接口”，“订单查询”的接口等等都可以加缓存，之前和 davis 讨论过，有一些缓存策略还是很复杂的，这部分需要详细设计

推荐一些缓存相关的文章：

- [缓存相关知识](https://app.yinxiang.com/shard/s47/nl/13163762/845a3580-d409-4a76-92c5-a6289adef114/)
  这篇文章包括了在缓存使用过程中“缓存穿透”，“缓存击穿”，“缓存雪崩”等等概念的介绍和解决方案

- [Redis 深度历险：核心原理与应用实践](https://juejin.im/book/5afc2e5f6fb9a07a9b362527/section/5b336601f265da598e13f917)
  很多文章写关于 redis 的文章或者书都会包括很多运维相关的知识，这本小册子里面干货很多，特别适合开发看，包括布隆过滤器，限流，LRU，分布式锁，介绍的都很清楚，里面的例子都有 java 版本的，推荐去看一下

## 代码中的事务处理

官方文档上说`Transactions are atomic units of work that can be committed or rolled back`，是一组工作的原子提交或者回滚，和`ACID`是紧密结合在一起的。
我的理解也不是很深刻，但是我认为有些代码事务太大了，有些不合理，例如`placeOrder`接口事务就用的比较大，查找员工信息，加载航班信息，这些应该都没有必要放到下单这个事务里。

```java
@Transactional
  public PlaceOrderResult placeOrder(PlaceOrderRequestBo request, Long bookingEmployeeId, Company company, boolean fromAdmin, Long staffId, String staffName) {
    LOG.info("bookingEmployeeId: {}", bookingEmployeeId);

    Employee bookingEmployee = employeeRepo.findByIdEnsured(bookingEmployeeId);

    //加载航班信息
    loadFlightInfo(request);

    //下单
    PlaceOrderResult placeOrderResult = doPlaceOrder(request, company, bookingEmployee, fromAdmin, staffId, staffName);

    LOG.info("Exit placeOrder. orderId: {}", placeOrderResult.getOrder().getId());
    return placeOrderResult;
  }
```

另外消费自动出票消息，是否有必要整个消费过程都加一个事务，`FlightTask task = getFlightTask(body);`，是否有必要加锁？

```java
@Override
  @Transactional
  public void consume(String tag, String key, String body) {
    LOG.info("Enter. body(taskId): {}", body);

    FlightTask task = getFlightTask(body);
    try {
      LOG.debug("task.isWasSystem() => {}, task.getTaskStatus() => ", task.isWasSystem(), task.getTaskStatus());
      if (!isTaskConsumedRepeatedly(task)) {
        attachSupplierInfoToOrder(task);
        doTicketing(task);
      }
    } catch (Exception ex) {
      LOG.error("Exception happen. ex: {}", ex.getMessage(), ex);
      handleTicketingExceptionCase(task);
    }

    LOG.info("Exit.");
  }
```

## MQ 的使用

MQ 虽然具有序解耦，异步等优点，但是 MQ 也会让系统变得复杂。MQ 并不能保证消息 100%的被正确投递，“丢消息”，“重复投递”都有可能会发生，目前我们的代码都没有考虑这些情况的发生。如果丢消息真实发生了，最好要有状态记录和补偿机制

推荐一门课程：

- [RabbitMQ 消息中间件技术精讲](https://coding.imooc.com/class/chapter/262.html#Anchor)
  这个是一门视频课程，虽然讲的是 RabbitMQ，但是这门课的第三章还是有很多干货，里面包括了如何保证消息 100%被投递成功，“大厂”在使用 MQ 的时候的解决方案，限流等等

---

> 下面都是杆精，说的不一定对

---

## 状态变量的存储

我认为在数据库中存储状态变量，没必要直接存储字符串，直接使用`0,1,2,3...`等状态码，没必要直接使用状态变量的文案。
首先这部分文案会占用很多数据库空间，存这部分字符串占用的空间是存状态码的几十倍，其次后期订单操作历史数据变多以后，对数据库性能也会有影响

```java
private void recordErrorInfo(FlightOrder order) {
    order.addOrderHisByAdmin(TaskAssigner.SYSTEM_STAFF_NAME, "申请自动出票失败", "发送自动出票消息时发生异常，转入人工处理流程");
    flightOrderRepository.save(order);
  }
```

## 代码中的硬编码

比如说下面代码，1，2，6 分别代表什么？新人写代码的时候并不知道`Applicationexception`，哪些 code 被别人使用过，新的 code 应该用多少。

```java
/**
   * 重设 Identity Step 1 => 检验密码和原identity并发送验证码
   *
   * @param resetCheckBo 相关参数
   */
  public void resetIdentityCheck(IdentityResetCheckBo resetCheckBo, long curEmployeeId) {
 validateCodeApplicationService.checkValidateCode(resetCheckBo.getValidateToken(), resetCheckBo.getValidateCode());
    try {
   employeeIdentityResetDomainService.preCheckAndSendVerifyCodeForIdentityReset(resetCheckBo, curEmployeeId);
    } catch (EmailNotCorrectException ex) {
      throw new ApplicationException(2, ex.getMessage());
    } catch (MobileNotCorrectException ex) {
      throw new ApplicationException(1, ex.getMessage());
    } catch (PasswordNotCorrectException ex) {
      throw new ApplicationException(6, ex.getMessage());
    }
  }
```

## 重复的代码

有些接口存在任务重复执行情况,比如”orderPrebilling“,消费"自动出票"消息等代码都有一些地方重复执行了`FlightOrder order = flightOrderRepo.findByIdEnsured(orderId);`，针对同一个订单，查一次以后就可以直接在代码里面传递`order`，没必要多次查库，会降低接口性能。

## ID 的处理

目前我们有些订单 id 是通过数据库自增的方式，性能比较差，并且订单量容易被看出来。有些地方用了"snowflake"方案，还是建议单独独立出来一个”发号“服务，便于业务拓展。

```java
@Transactional
  public String getNextSeqValue(String seqName) {
    LOG.debug("Get next sequence value with seqName: {}", seqName);

    sequenceRepository.incrementSequence(seqName);
    long result = sequenceRepository.getCurrentSequenceValue(seqName);

    LOG.debug("Get next sequence value of: {} with result: {}", seqName, result);
    return String.valueOf(result);
  }
```

推荐好文：

- [美团点评分布式 ID 生成系统](https://juejin.im/entry/58fb22655c497d0058f5febb)

## 参数的传递

代码中一些参数的传递太大，比如说下面几个例子，其实只是需要`PlaceOrderRequestBo`中的某个字段，没必要把整个对象传进去，在阅读代码的时候会很困惑。

```java
/**
   * 创建订单乘机人列表
   */
  @LoggerAnnotation
  public List<FlightOrderPassenger> createOrderPassengers(PlaceOrderRequestBo request, FlightOrder order, Company company, Employee bookingEmployee) {

    List<FlightOrderPassenger> passengers = new ArrayList<>();
    int seqNo = 0;

    for (PassengerBo passengerBo : request.getPassengers()) {
      FlightOrderPassenger passenger = createOrderPassenger(passengerBo, order, company, bookingEmployee, seqNo);
      //passenger.setSeqNo(++seqNo);挪到创建乘客过程中，避免子订单号创建出错
      passenger.setOrder(order);
      passengers.add(passenger);
      ++seqNo;
    }
    return passengers;
  }

  /**
   * 根据下单请求参数添加或更新员工信息
   * @param request 下单请求参数
   */
  @Transactional
  public void updateEmployeeInfoIfRequired(PlaceOrderRequestBo request, long corpId) {
    LOG.debug("Enter");

    request.getPassengers().forEach(passengerBo -> {
      if (StringUtils.equalsIgnoreCase(passengerBo.getPassengerType(), FlightPassengerType.ADULT)) {
        updateEmployeeInfoIfRequired(passengerBo, corpId);
      }
    });
    LOG.debug("Exit");
  }

  /**
   * 创建订单行程列表
   *
   * @param request
   * @param order
   * @param fromAdmin
   * @return
   */
  @LoggerAnnotation
  public List<FlightOrderRoute> createOrderRoutes(PlaceOrderRequestBo request, FlightOrder order, FlightConfig flightConfig, boolean fromAdmin) {

    List<FlightOrderRoute> routes = new ArrayList<>();
    int seqNo = 0;

    for (RouteBo routeBo : request.getRoutes()) {
      FlightOrderRoute route = createOrderRoute(routeBo, flightConfig, fromAdmin);
      route.setSeqNo(++seqNo);
      route.setOrder(order);
      routes.add(route);
    }
    return routes;
  }
```
