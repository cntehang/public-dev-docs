# 20180706 代码契约 V0.1 Beta

-----

### 1. Spring 依赖注入
- **结论**：绝大部分情况下只使用 Constructor DI，不使用 Field DI
- **注意事项**：
    - 使用 Constructor DI时，若只有一个构造方法，则不用在构造方法上加 @Autowired 注解，且该构造方法形参应只包含要注入的那些依赖
    -  It’s fine to use field based injection in tests when you’re using the SpringJUnit4ClassRunner.
- **主要原因**：
    - 如果使用 Field DI，对于IOC容器以外的环境，除了使用反射来提供它需要的依赖之外，无法复用该实现类。而且将一直是个潜在的隐患，因为你不调用将一直无法发现 NullPointerException 的存在。 
    - 上文注意事项中提到的 SpringJUnit4ClassRunner 类配合 @Runwith 注解可以让测试在 Spring 容器环境下执行，不会出现标注了 Field DI 的字段未注入的情况。
- **支撑文章**：[Why field injection is evil](http://olivergierke.de/2013/11/why-field-injection-is-evil/) 

### 2. 路由 Router文件
- **结论**：为保证对外服务的统一管理，将具体的路由信息写在一个统一的文件中，而不是将其拆散写在控制器的每个方法声明头上

### 3. Restful Or Not
- **结论**：
    - 拒绝Restful
    - 应用运行的状态码由开发人员自己制定和维护
    - HTTP 动词使用 GET、POST、DELETE
- **主要思路**：HTTP Code 用于区分网络问题和业务问题，自定义 Code 用于处理业务问题，不要使用 HTTP Code 去反映业务的运行情况，一方面其容量有限，另一方面使用其的学习成本也不低。Restful 风格的 API 虽然流行，但是要有自己的思考，其更适用于简单的场景。而商业系统开发往往是复杂的，不要开始就想着去简化，而应该从后期的管理上深入、想办法。

### 4. 接口与实现类命名风格
- **结论**
    - 接口名称前不加附属的字母（如接口前加 I），
    - 实现类后不加附属的无意义单词（如 Impl）
    - 实现类名相较于接口名后面可加有意义的单词，如Db

### 5. Pimitive Type or Wrapper Class 
- **结论**：
    - 若该字段可以不初始化，使用 Wrapper Class
    - 若该字段必须被初始化，使用 Primitive Class
- **举例**：
    - Employee类中的corpId
        - 如果员工定义时可以不指定所属公司，则用Integer
        - 若定义员工时必须定义公司，则用int

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

### 9. LOG的等级划分 => 后面提供要求文档（刘博）
- **结论**：
    - 不要求进出都写，非常短小的的方法不用写，只要能追踪主要参数即可
    - 数据量太大时打印变量用Trace，数据量小用Info

