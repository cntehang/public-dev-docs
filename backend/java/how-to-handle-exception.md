# 服务当中，如何处理异常

- 不允许把产生的异常直接抛出到前端，必须转换成前端可处理的信息抛出

- 如果是 cache 底层异常，然后再此转换成 applicationException 再抛出的话，统一用下面这种方式：

```java
catch (***Exception ex) {
  // 将原有异常的信息一并抛出
  throw new ApplicationException(ORDER_NOT_EXIST_CODE, ex.getMessage(), ex);
}
```

- 如果是在 application 出现错误，这时候需定义清楚 code 和 message，如：

```java
if (***true***) {
  throw new ApplicationException(FEE_EXCEED_TOTAL_CODE, FEE_EXCEED_TOTAL_MSG);
}
```

- domain 层抛出 System Error，application 不需要捕获。
