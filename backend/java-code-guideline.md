# java-code-guideline


## 1. Spring 依赖注入

- **结论**：只使用 Constructor DI，不使用 Field DI
- **注意事项**：
  - 使用 Constructor DI 时，若只有一个构造方法，则不用在构造方法上加 @Autowired 注解，且该构造方法形参应只包含要注入的那些依赖
  - It’s fine to use field based injection in tests when you’re using the SpringJUnit4ClassRunner.
- **主要原因**：
  - 如果使用 Field DI，对于 IOC 容器以外的环境，除了使用反射来提供它需要的依赖之外，无法复用该实现类。而且将一直是个潜在的隐患，因为你不调用将一直无法发现 NullPointerException 的存在。
  - 上文注意事项中提到的 SpringJUnit4ClassRunner 类配合 @Runwith 注解可以让测试在 Spring 容器环境下执行，不会出现标注了 Field DI 的字段未注入的情况。
- **支撑文章**：[Why field injection is evil](http://olivergierke.de/2013/11/why-field-injection-is-evil/)

## 2. 路由 Router 文件

- **结论**：为保证对外服务的统一管理，将具体的路由信息写在一个统一的文件中，而不是将其拆散写在控制器的每个方法声明头上

## 3. API 返回数据结构及错误代码

应用层与网络层独立，尽量减少全局依赖是 API 设计的一些出发点。最高原则则是用户体验：用户看到最有用的信息。

HTTP Code 是网络层的代码，不要用于反映业务的运行情况。一方面其数量有限，另一方面及其模糊。比如对于错误数据请求： 400 (bad request)、409 (409 Conflict)、 412 (Precondition Failed)、 422(Unprocessable Entity)都在各处使用而且意义不清。Restful 风格的 API 虽然流行，但是要有自己的思考，其更适用于简单的资源 CRUD 场景。而商业系统开发往往是复杂的，不要开始就想着去简化，而应该以系统思维方式处理复杂性。

### 3.1 建议

- HTTP 或任何通讯层协议与业务逻辑/错误处理完全分离。
- 不用 Restful 错误代码代表业务错误，HTTP code 只用于 HTTP 通讯。
- 只使用 HTTP POST，建议所有参数放在 Body，对于简单情形（比如只有一两个简单类型参数）也可以使用 URL Parameter。
- 应用运行的状态码由开发人员自己制定和维护。
- 业务请求参数错误也应该放到业务层而不是网络层（如下面 bing 的例子）。
- 错误代码/信息的目的是为了给出错误来源，最好能给出用户解决问题的方法（如下面 Bing 的例子）

### 3.2 建议的 API 返回数据结构

```javascript
{
    code: number // success 0, business exception starts with 1, system exception starts with 100
    message: string // message, 'OK' for 0. The message is user-oriented that might be shown to end user.
    debugMessage?: string // optional, message used for debugging, including request/context data.
    data?: object // optinal, the biz data, empty if code is not 0
}
```

### 3.3 错误代码

- 基本原则: 成功 0, 用户关心的业务错误代码 1 - 99, 其他业务与系统错误代码 >100。
- 每个 API 方法都有自己定义的 1-99 的业务错误代码，在 API 文档有详细说明。
- 尽可能用有意义的业务错误代码和信息。常见的例子有：
  - 某个 API 的一个参数是电话号码， 如果电话号码不合法（比如包含了字符），返回的错误代码应该在 1-99，同时给出 `错误电话号码`信息. `debugMessage`可以给出用户的输入参数和系统的检查信息，比如正则表达式。采用中间件返回 400 (bad request）对用户和 API 使用者没有太大帮助。
  - 同理，如果用户没有给出一个必须的参数，最好明确指出那个参数缺少 （如下面的 Bing 的例子）。
- 100 以上的系统错误代码通常由系统中间件处理，可以有一份指导性的标准错误代码。 中间件被各个子系统共享，所以容易标准话。对不关心的系统错误，建议用统一错误代码 500。

### 3.4 他山之石

- **Twitter**：用 HTTP 代码 400 表示所有的商业逻辑错误，同时提供错误细节

```javascript
{
    "errors":
            [{
                "code":215,
                "message":"Bad Authentication data."
            }]
}
```

- **Facebook**：用的 GraphQL 风格，HTTP 仅仅是通讯层。错误信息包含了 Exception 的类型和内部调试索引

```javascript
{
    "error":{
        "message": "Syntax error \"Field picture specified more than once. This is only possible before version 2.1\" at character 23: id,name,picture,picture",
        "type": "OAuthException",
        "code": 2500,
        "fbtrace_id": "xxxxxxxxxxx"
    }
}
```

- **Bing**：HTTP 仅仅是通讯层。商业逻辑错误还是用 HTTP 200 状态码。特点是给出了错误的解决答案（哪个参数不对）和请求的后台 URL

```javascript
HTTP/1.1 200
{
    "SearchResponse": {
        "Version":"2.2",
        "Query":{
            "SearchTerms":"api error codes"
        },
        "Errors":[{
            "Code":1001,
            "Message":"Required parameter is missing.",
            "Parameter":"SearchRequest.AppId",
            "HelpUrl":"http\u003a\u002f\u002fmsdn.microsoft.com\u002fen-us\u002flibrary\u002fdd251042.aspx"
        }]
    }
}
```

### 4. 接口与实现类命名风格

- **结论**

  - 接口名称前不加附属的字母（如接口前加 I），
  - 实现类后不加附属的无意义单词（如 Impl）
  - 实现类名相较于接口名后面可加有意义的单词，如 Db

### 5. Pimitive Type or Wrapper Class

- **结论**：
  - 若该字段可以不初始化，使用 Wrapper Class
  - 若该字段必须被初始化，使用 Primitive Class
- **举例**：
  - Employee 类中的 corpId
    - 如果员工定义时可以不指定所属公司，则用 Integer
    - 若定义员工时必须定义公司，则用 int

### 6. 实体类字段不赋默认值

### 7. 方法（Method）风格

- **结论**：过长代码要抽成方法

### 8. 异常的处理

- **结论**：开发应用时，使用两种异常
  - Java 定义的标准异常，如参数检查中发现 null 参数
  - 业务相关自定义异常：如密码过长或过短
    - 自定义异常应继承自 RuntimeException，因为它是 Unchecked Exception
    - 使用异常类型而非异常信息来分辨异常来自的不同 Domain 类
- **注意**：以上两个例子即使和你的业务契合，也不一定满足和你的业务要求

### 9. 数据库事物处理（Transaction）

读数据错误与丢失更新

- [事务隔离级中](https://juejin.im/post/5b90cbf4e51d450e84776d27)，脏读、不可重复读、幻读三个问题都是由事务 A 对数据进行修改、增加，事务 B 总是在做读操作。

如果两事务都在对数据进行修改则会导致另外的问题：丢失更新。为什么出现丢失更新：

- 多个 session 对数据库同一张表的同一行数据进行修改，时间线上有所重复，可能会出现各种写覆盖的情况。
- 示例可见：[并发事务的丢失更新及其处理方式](https://blog.csdn.net/u014590757/article/details/79612858)

解决方案：

- 根据具体的业务场景，尽量缩小事物范围并采用正确的[事物隔离级别](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)。
- 使用数据库行级锁（如乐观锁）。完全避免对行级数据的脏操作，但是使得对该行数据的访问串行化，对于比较大的表对象而言，这样的设置往往不是我们想要的结果。
- 缩小事务管辖的范围。控制事务所辖代码执行时间的长度，不能将很耗时的操作（如外部服务调用）与数据修改置于同一个事务中。此方案只是尽量减少两个事务中的写操作互相影响的可能，无法完全避免。
- 使用 ORM save 方法实现数据持久化的情况下，开启 Dynamic update，使得保存更改时影响的字段仅限于被改动了字段。此方案通过控制更新字段的范围，尽量减少脏操作可能，但也无法完全避免。

### 10. 关于 Hibernate Dynamic update

主要缺陷

- 语义错位。本意是直接修改部分属性，现在变成取整个 Object，改部分属性，存整个 Object。中间不可控因素太多。
- 每次根据改动了的字段，动态生成 SQL 语句，性能上相比全更操作有所降低
- 需要从数据库拿到整个 Object 所有数据才能修改，大多数时候不必要，
- 当两个 session 同时对同一字段进行更新操作，极端情况下会因为 ORM 缓存出现莫名其妙的情况，示例见：[Stackexchange Q: What's the overhead of updating all columns, even the ones that haven't changed](https://dba.stackexchange.com/questions/176582/whats-the-overhead-of-updating-all-columns-even-the-ones-that-havent-changed)

### 11. 如何更新数据库字段

- 拒绝使用 Spring Data JPA 的 save 方法
  - 默认配置且未使用锁的情况下，save 方法会更新实体类的所有字段，一方面增加了开销，另一方面歪曲了更新特定字段的语义，多线程并发访问更新下的情况下易出现问题。
  - 配置动态更新且未使用锁的情况下，save 方法会监测改动了的字段并进行更新，但是可能会出现 11 点中提到的古怪情形。
  - 总的来看，使用 ORM save 方法进行实体类更新陷入了 “You wanted a banana but you got a gorilla holding the banana” 的怪圈，导致做的事情不精确、或者有其它的风险。[参考文章](https://www.johndcook.com/blog/2011/07/19/you-wanted-banana/)
- 使用自定义 SQL 进行字段更新
  - 使用 JPA 提供的 @Query/@Modifying 书写 JPQL 进行精确控制的字段更新操作。
