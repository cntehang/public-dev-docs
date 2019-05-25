# Java 高级代码规范

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

## 4. 接口与实现类命名风格

- **结论**

  - 接口名称前不加附属的字母（如接口前加 I），
  - 实现类后不加附属的无意义单词（如 Impl）
  - 如果接口表示能力，可以使用 able 作为结尾
  - 实现类名相较于接口名后面可加有意义的单词，如 Db

## 5. Pimitive Type or Wrapper Class

- **结论**：
  - 若该字段可以不初始化，使用 Wrapper Class
  - 若该字段必须被初始化，使用 Primitive Class
- **举例**：
  - Employee 类中的 corpId
    - 如果员工定义时可以不指定所属公司，则用 Long
    - 若定义员工时必须定义公司，则用 long

## 6. 实体类字段不赋默认值

- POJO 类属性没有初值是醒使用者在需要使用时，必须自己显式地进行赋值，任何 NPE 问题，或者入库检查，都由使用者来保证。

## 7. 方法（Method）风格

- **结论**：
  - 过长代码要抽成方法
  - 每个函数不超过 10 条语句
  - 一个函数的所有语句都在单一抽象层（SLA 原则）

## 8. 异常的处理

- **结论**：开发应用时，使用两种异常
  - Java 定义的标准异常，如参数检查中发现 null 参数
  - 业务相关自定义异常：如密码过长或过短
    - 自定义异常应继承自 RuntimeException，因为它是 Unchecked Exception
    - 使用异常类型而非异常信息来分辨异常来自的不同 Domain 类
- **注意**：以上两个例子即使和你的业务契合，也不一定满足和你的业务要求

## 9. java 中使用 swagger

针对`io.springfox:springfox-swagger2`的使用，我们要注意：

1. @ApiModel 注解 value 属性值不能写中文，会导致 swagger 导出 json 时会报错。建议直接不写参数。
2. 任何 swagger 注解的属性值都不要有单引号，json 不认识单引号，swagger 导出 json 会报错。比如@ApiModelProperty 注解 example 属性值我们有时候希望给复杂类型（比如"['111','222']"）。遇到这种情况，我们不写 example。

## 10. 对同一服务下的接口文档进行分类

如 [swagger-usage-guideline](https://github.com/cntehang/public-dev-docs/blob/master/backend/swagger-usage-guideline.md) 中说明的那样，我们使用 swagger 用来作为前后端交流接口约定的工具。

随着接口数量的增长，Controller 数量的变多，建议对 API 文档按照 controller 所属的业务范畴进行分类（即 swagger 中的 spec）。Swagger 会根据 Swagger 配置类中定义的 Bean 生成不同的 spec，如下 Bean 配置会扫描包路径 com.tehang.tmc.services.application.rest 下的所有 Controller 并将其归属于一类。

```java
  /**
   * swagger配置
   *
   * @return swagger相关配置
   */
  @Bean
  public Docket createRestApi() {
    return new Docket(DocumentationType.SWAGGER_2)
        .forCodeGeneration(true)// 解决泛型转json错误的问题
        .useDefaultResponseMessages(false)
        .apiInfo(this.getApiInfo())
        .globalOperationParameters(this.getParams())
        .select()
        .apis(RequestHandlerSelectors.basePackage("com.tehang.tmc.services.application.rest"))
        .paths(PathSelectors.any())
        .build();
  }
```

## 11. 更多地使用官方工具包中定义好的 API，以一种更易读的方式对空、null 进行判断

- 判断集合是否非空使用 org.apache.commons.collections4.CollectionUtils.isNotEmpty
- 判断字符串非空使用 org.apache.commons.lang3.StringUtils.isNotEmpty，若还需非空格则使用 org.apache.commons.lang3.StringUtils.isNotBlank
- 判断对象为非 null 使用 java.util.Objects.nonnull
- 判断 Boolean 类型是否为 true 使用 org.apache.commons.lang3.BooleanUtils.isTrue

## 12. 日志记录应当涵盖所有代码分支

日志记录是为追踪业务流程，排查系统 BUG 服务的，所以日志记录应该涵盖代码执行的所有分支。如：

- while 语句代码快
- if-else 语句的 if 代码块和 else 代码块
- throw exception 代码前

## 13. 所有集成测试用例使用统一的外部服务模拟器

集成测试中需要对当前服务调用到的外部服务进行模拟（使用 WireMock 等工具），如下代码定义了一个运行在 10098 端口的模拟服务器：

```java
 /**
  * wire mock, 10098是配置中指定的端口
  */
 @Shared
 WireMockRule wireMockRule = new WireMockRule(10098)
```

如果为每个 IntegrationTest 类创建一个模拟服务器，一方面会降低集成测试运行的效率（考虑模拟服务器开闭消耗的时间），另一方面会给包含异步方法调用的集成测试带来访问不到目标地址的风险（进行访问拦截的模拟服务器此时可能已经关闭）。故推荐为所有的集成测试启动一个唯一的外部服务模拟器，统一加载需要拦截的 URL，在所有集成测试运行之始，在所有集成测试运行结束后关闭。

## 14. 统一服务的错误处理

这里的统一是指形式上的统一，如同本文第三点中所述 API 返回体数据结构定义的那样，错误信息也应该在这样的数据结构中返回。
要想做到这一点，我们需要一个统一的异常拦截器对程序中抛出的异常进行包装，以兼容定义好的数据结构。要想实现这一点，可使用自定义 ExceptionResolver（Spring 3.2 以下），或者使用 ControllerAdvice（Spring 3.2 及以上）。一个可能的最佳异常处理器形式如下：

```java
/**
 * 微服务异常集中处理器
 */
@RestControllerAdvice
public class BestExceptionHandler {

  /**
   * 处理异常
   */
  @ExceptionHandler
  @ResponseBody
  @ResponseStatus(HttpStatus.OK)
  public DataContainer handle(Exception ex) {

    DataContainer container;

    if (ex instanceof ApplicationException) {
      container = new DataContainer(((ApplicationException) ex).getCode(), ex.getMessage());
    } else if (ex instanceof ParameterException || ex instanceof ConstraintViolationException || ex instanceof MethodArgumentNotValidException) {
      container = new DataContainer(CommonCode.PARAMETER_ERROR_CODE, ex.getMessage());
    } else if (ex instanceof DataAccessException) {
      container = new DataContainer(CommonCode.SQL_ERROR_CODE, CommonCode.SQL_ERROR_MESSAGE);
      //other debug message setting
    } else {
      container = new DataContainer(CommonCode.COMMON_ERROR_CODE, ex.getMessage());
    }

    return container;
  }
}
```

相较于自定义 ExceptionResolver 实现，使用 ControllerAdvice 的优点在于屏蔽了 Respon/Request 等对象，以及丑陋的写输出流的操作。需要注意的是，这里限制了返回的所有错误形式都是我们约定好的 API 响应数据结构。

目前我们项目中混用了 ExceptionResolver 和 ControllerAdvice，实际上选用一种即可。
