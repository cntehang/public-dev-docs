# Java命名规范

## 0. 原则

- 命名作为软件中最困难的两个问题之一，需要认真对待；
- 命名有不同的风格习惯，保持团队的命名风格统一有利于大家聚焦在更有价值的事情上；
- 函数和变量命名要有意义，容易理解；
- 良好的命名可以提高软件的可读性，从而易于更于理解和维护。

## 1. 项目名

- 全部小写，单词间以中划线分隔，如: staff-service, config-service。

## 2. 包名

- 全部小写，用名词，如: com.tehang.tmc.services.domain.flight。

## 3. 类名和接口名

- 类名遵守Pascal命名法（即首字母大写，多个单词组成时，每个单词的首字母大写)。如:

```java
 public class FlightOrder {}
 ```

- 接口的命名和类保持一致

## 4. 方法名

- 方法名遵守camel命名法(即首字母小写，多个单词组成时，从第二个单词开始，每个单词的首字母大写。
- 方法名应为动词或动词词组，如：

```java
public void cancelOrder()
```

## 5. 变量名

### 5.1 普通变量名

- 采用camel命名法，一般为名词形式，如:

```java
FlightOrder order = null;
```

- 禁止使用i, j等单字母变量，应使用更有意义的名字；
- 代码中一般不允许直接写入数字（magic number)和字符串，应定义成更有意义的常量名称后使用；

  - **例外**：对于0和1, 在某些情况下，直接写入数字可能更易于理解，如下：

  ```java
  /**
   * 获取订单的第一个乘机人，这种情况下，直接使用0比常量更易于理解，是可以接受的
   */
  FlightOrderPassenger passenger = order.getPassengers.get(0);
  ```

### 5.2 常量名

- 全部大写，单词间以下划线分隔, 如:

```java
public static final String ORDER_STATUS = "TicketConfirmed";
```

## 6.  关于缩写

- 一般不允许使用缩写，除非该缩写是大家公认的，没有异议的，比如 id，dto等
- 引入新的缩写词，需经团队成员共同确认，并列在下表中
  - id: 实体类的主键字段
  - dto: 数据传输对象
  - bo: 业务层使用的数据传输对象
  - repo：数据访问层使用的JPA的Repository类型的变量，以xxxRepo命名
  - utils: 辅助性质的工具类，以xxxUtils命名
  - spec: 单元测试的类名, 以xxxSpec命名，其中xxx表示待测试的原始类名
