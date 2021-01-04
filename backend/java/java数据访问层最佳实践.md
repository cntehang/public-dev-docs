# Java 数据库访问最佳实践

团队项目主要使用 Spring Data JPA (以下简称 JPA) 和 Spring JdbcTemplate (以下简称 JdbcTemplate) 进行数据库访问。对这两种工具的选择、使用及特性做如下总结。

## 1 根据场景选择工具

- 对数据的增，删，改主要使用 JPA（底层基于 hibernate 的实现)，简单的查询也使用 JPA;

  - 优点：简单，可读性好

- 如果遇到一定需要手写 update 语句的场景，使用@Query（value = "update ...", nativeQuery = true)，再配上@Modifying 和@Transactional

  - 优点：执行修改语句更灵活，想更新什么字段就更新什么字段

- 对于复杂的查询，比如条件比较复杂，或者用嵌套的子查询，分页等，使用 JdbcTemplate；

  - 优点：使用原生 sql 语句，更灵活

## 2 规避 JPA 的常见问题

下面的几个问题互相依赖，归根结底，是 Spring Boot 默认开启 OSIV 带来的。此行为的的理由有：

- 减少 LazyInitializationException，让代码编写更符合直觉，无需关心数据库访问细节
- 傻瓜化数据库访问，数据库连接的概念基本对用户不可见，无需关心连接的获取和释放
- 简单化编码，无需获取 EntityManager 对象即可进行数据库访问，无需显式获取和释放连接

在关闭 OSIV 后，需要解决很多的 LazyInitializationException 问题，如果项目已经深陷 OSIV 泥潭，至少先心里有个数，明白这些问题的来龙去脉。

### 2.1 N+1 问题

该问题为 ORM 的常见问题。在开启 OSIV 的情况下容易静默地出现。考虑有如下数据库表对应的 Java 代码中的实体类数据结构：

```java
@Entity
@Table(name = "corp_employee")
public class Employee {

  @Id
  private long id;

  /**
   * 员工常用联系人，每个员工可有多个联系人，为一对多的关系
   */
  @OneToMany(
      cascade = CascadeType.ALL,
      orphanRemoval = true,
      fetch = FetchType.LAZY)
  @JoinColumn(name = "employee_id")
  private Set<EmployeeDoc> docs;
```

存在列表查询中，分页查询 Employee 并转换为如下 Dto 的需求：

```java
public class EmployeeDto {

  @ApiModelProperty(value = "员工id")
  private long id;

  @ApiModelProperty(value = "证件")
  private Set<EmployeeDocDto> docs;
```

实现此需求的伪代码为：

```java
List<Employee> employees = employeeRepository.findByConditions(xx);
return employees.stream().map(EmployeeDocDto::build).collect(Collectors.toList());
```

由于 Employee 下的 EmployeeDoc List 对象是懒加载的（懒加载是 ORM 中的一种推荐做法），在遍历查询出来的 N 个 Employee 转换成 EmployeeDto 的过程中，会再发起 N 次针对 EmployeeDoc 的查询，即一次查询带出了 N 个额外的查询，故称之为 N+1 问题。解决方案有：

- 1 使用 JPA 中的 NamedEntityGraph 注解，以非懒加载的方式按预先设置的加载模版，加载出需要的所有数据。参考[教程](https://www.baeldung.com/spring-data-jpa-named-entity-graphs)。其局限是只能够非懒加载[一组子对象](https://stackoverflow.com/a/63044707/9304616)。
- 2 使用 JdbcTemplate 首先查询出 Employee 相关数据，再以此为基础查询相关的 EmployeeDoc 数据，再在内存中进行组装。比较麻烦的地方是需要额外定义一些 BO。
- 3 关闭 OSIV。

关闭 OSIV 值得拉出来单独探讨。关闭 OSIV 后，获取数据的时候需要显式地获取连接才能进行（在 @Transactional 注解范围内或者直接操控 EntityManager 对象）。对于使用 Spring Boot 默认配置被 OSIV 惯坏了的程序员，会惊讶于以下的简单代码会抛出 LazyInitializationException。

```java
    // 外层无事务
    Optional<Employee> employee = repository.findById(1L);
    System.out.println(employee.get().getDocs());
```

这正是关闭 OSIV 的代价，也是 Spring Boot 开发人员心心念念要[默认开启 OSIV 的原因](https://github.com/spring-projects/spring-boot/issues/7107#issuecomment-260633493)——让程序开发更符合直觉和简单，但是掩盖了背后的真正行为。

### 2.2 异步线程中访问懒加载子对象失败问题

这是刚开始使用 Spring Data JPA 和 @Async 注解时容易遇到的问题，异常信息为：

- org.hibernate.LazyInitializationException - could not initialize proxy - no Session

常见场景有：

- 在主线程中取出了一个 Employee 对象，传递到一个 @Async 异步方法中，在该异步线程中访问了 Employee 下的 docs 等懒加载字对象。
- 测试代码中取出对象并访问其一对多子对象，若未特殊处理，也会出现此问题。

同上一部分所述，被 Spring Boot 的 OSIV 惯坏了的程序员一开始面对此问题会手足无措。关闭 OSIV 的情况下此类型问题会更早被暴露。关于如何在关闭 OSIV 的情况下（以及在异步线程中）做一个负责任的程序员，正确的处理 LazyInitializationException，[这篇文章](https://vladmihalcea.com/the-best-way-to-handle-the-lazyinitializationexception/)有很好的解释，综合归纳为以下解决方案：

- 1 和第一个问题一样，可以使用 EntityGraph 来精确控制每次要同时加载的字对象，而不用二次加载。限制仍然是只能加载一个一对多子对象。
- 2 使用非懒加载，即 @OneToMany 中加上 fetch = FetchType.EAGER。[不推荐这种做法](https://vladmihalcea.com/the-best-way-to-handle-the-lazyinitializationexception/)，会在很多情况下取出多余数据。
- 3 在配置文件中设置 enable_lazy_load_no_trans: true。[不推荐这种做法](https://vladmihalcea.com/the-hibernate-enable_lazy_load_no_trans-anti-pattern/)，这是一种非常丑陋的模式。
- 4 在异步线程中加上 @Transactional 注解，或者使用其他方法开启数据库连接。不推荐将实体类在线程间传递，推荐在异步线程中接受 ID，并重新查询出实体类。

### 2.3 影响数据库性能（真的！）

考虑下面这段代码：

```java
Optional<Employee> employee = repository.findById(1L);
if (employee.ifPresent()) {
    // do remote call
}
```

在开启 OSIV 的情况下，每个请求会在一开始就绑定一个 Session 对象，并在第一次执行数据库访问的时候为 Session 对象绑定一个数据库连接（从服务的数据库连接池中取一个），Session 在本次请求结束的时候自动关闭，并归还数据库连接。上面的代码中，当有远程调用，如果这个远程调用需要消耗蛮长的时间（如 10s），那么只需要同时并发 10 个请求，就能让[默认配置数据库连接池配置](https://stackoverflow.com/a/55026845/9304616)（10 个连接）下的 Spring Boot 服务陷入无数据库连接可用的情况，带来巨大的性能问题。

## 3 一些推荐的代码写法

- 在@Query 中返回 bool 值

  使用 case when 语法:

```java
  /**
   * 根据航司代码查看是否存在（排除指定的id)
   */
  @Query("select case when count(id) > 0 then true else false end from CommonAirline  where code = :code and id <> :exceptId")
  boolean existsByCodeAndExceptId(@Param("code") String code,
                                  @Param("exceptId") Long exceptId);
```
