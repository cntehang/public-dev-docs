# API设计规范

## 0. 原则

- 使用RPC风格, 拒绝REST。
- 只将HTTP作为通讯协议，正常情况下只使用HTTP状态码200。
- 定义特定的业务代码作为操作的返回。
- 使用Json格式，而非Xml。

## 1. 路由

- 路由格式为 `https://{baseUrl}/{module}/{version}/{controller}/{action}`, 其中module是可选的，例如：

```java
https://dev-tmc-services.teyixing.com/front/v1/flight/cancelApply
```

- 为每个Controller添加一个对应的路由Router文件，并为Controller的每个Action方法添加对应的常量，而不是直接硬编码路由到每个方法上; Route中的每个常量值应与Controller中的方法名完全相同。
- Controller的Action方法应该用动词或动词词组命名。

## 2. 请求参数

- 请求方法只使用HTTP POST
- 请求参数应该以json格式放在HTTP的body中

  - **例外** 对于简单情形（比如只有一两个简单类型参数）也可以使用URL Parameter来传递，但对于密码，金额等敏感数据必须放在body中传递。

- 只传递操作必须的数据，不应该为了重用而传递多余的不需要的数据。

## 3. 返回

- 返回格式的定义如下：

```javascript
{
    code: number // success 0, business exception starts with 1
    message: string // message, 'OK' for 0. The message is user-oriented that might be shown to end user.
    debugMsg?: string // optional, message used for debugging, including request/context data.
    data?: object // optinal, the biz data, empty if code is not 0
}
```

- 返回状态码  
  - 使用自定义的状态码表示业务操作的返回状态。
  - 0表示成功。
  - 1~96用来表示需要客户端关注的业务状态，这些状态码应该在API详细文档中有详细说明。
  - 97表示数据访问错误。
  - 98表示参数错误。
  - 99表示未知错误。  
  - 100 以上的系统错误代码通常由系统中间件处理，可以有一份指导性的标准错误代码。 中间件被各个子系统共享，所以容易标准化。

## 4. 文档

- 使用swagger来生成api文档, 参考 [swagger文档使用说明](./swagger-usage-guideline.md)

## 5. 测试

- API开发时应先定义出接口，包括路由，传入和传出参数。
- 在开发完成之前，应先生成模拟数据，模拟数据应定义为json为文件。
- 在开发完成之后，使用真实数据替代模拟数据。