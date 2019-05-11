# java中stream使用规范

## 1. stream中的filter表达式不要写得太长，对于复杂的表达式建议封装方法；  

例如：

```java
List<FlightOrder> orders = orders.stream()
  .filter(order -> StringUtils.equals(order.status, "Submitted") 
    && StringUtils.equals(order.paymentStatus, "Billed")
    && StringUtils.equals(order.authorizeStatus, "Passed"))
  .collect(Collectors.toList();
```

可以重构为

```java
List<FlightOrder> orders = orders.stream()
  .filter(this::orderCanTicketing)
  .collect(Collectors.toList();
```

## 2. 不要嵌套使用stream，嵌套的steam可读性很差，建议将内层的stream封装成独立的方法；  

## 3. stream要适当地换行，不要写在一行中;

## 4. 不要在stream中访问数据库；  
 
原因： 在循环中访问数据库往往导致性能问题。

## 5. 不要使用stream来更新数据，只用stream来查询

例如：

```java
List<FlightOrder> orders = orders.stream()
  .filter(this::orderCanTicketing)
  .map(this::setTicketSuccess)
  .collect(Collectors.toList();

private FlightOrder setTicketSuccess(FlightOrder order) {
  order.setStatus("TicketSuccess");
  return order;
}
```

可以重构为

```java
List<FlightOrder> orders = orders.stream()
  .filter(this::orderCanTicketing)  
  .collect(Collectors.toList();

orders.foreach(this::setTicketSuccess);

private void setTicketSuccess(FlightOrder order) {
  //...
}
```
