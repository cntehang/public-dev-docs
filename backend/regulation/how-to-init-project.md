# Java 服务开发

---

## 基本环境与开发工具

- JDK： 1.8
- Gradle： 4.8
- IDE： IntelliJ IDEA
- Mysql： 5.7
- Redis： 4.0
- Springboot：2.0.4

## 初始化项目

- 在 github 上创建项目：项目名称采用小写单词加中划线命名
  - 创建时选择 private
  - 自动生成 readme.
- 将 github 上的项目 clone 到本地：此时得到本地的一个空项目

- 创建 gradle 项目，勾选 java、Groovy
  - GroupId：com.tehang.xxx.xxx
  - ArtifactId：xxx-xxx-xxx
  - 然后 IDE 会将项目会将项目初始化好
- 添加 git ignore 文件：复制[git ignore](https://github.com/cntehang/public-dev-docs/blob/master/.gitignore)文件内容即可
- 提交项目到 github

`至此，项目就初始化完毕了.`

## 初始化项目结构

### 添加构建脚本

将之前构建脚本 copy（build_scripts 文件夹，与根目录下的 build.gradle）过来，并修改以下内容：

- 依赖项(application_dependencies.gradle)，保证只使用了本项目需要的依赖

- 应用版本和名称：application_version.gradle

- maven 仓库(build_setups.gradle)，原有阿里云 maven 仓库地址(<http://maven.aliyun.com/nexus/content/groups/public/>)随时可能失效，请使用最新的地址(<https://maven.aliyun.com/repository/public>)

### 创建代码基本包目录

- 基本包目录

```bash
-main
    - java
        - com
            - tehang
                - <项目名>
                    - application ： 对外的服务
                        - builder ： dto的构建方法
                        - dto ： 对外提供的model
                        - rest ： rest接口
                        - service ： 跨服务、跨domain的服务
                    - domain ： 业务逻辑
                        - model ： 数据model
                        - repository ： 数据库访问的入口
                        - service ： 业务逻辑
                    - infrastructure
                        - config ： 项目配置项
                        - exceptions ： 异常定义与处理
                        - filters ： 一些过滤器，用于请求前、后的一些处理
                        - routers ： 路由定义
                        - ... ： 其他一些可能用到的包
                    - utility : 常用工具类
                    - Application.Java : springboot 启动入口
    - resources : 配置目录
        - application.yml ： 公共配置
        - application-dev.yml : 针对开发环境的配置
        - application-test.yml : 针对测试环境的配置
        - application-pro.yml : 针对生产环境的配置
-test
    - groovy
        - com.tehang.xxx.xxx : 单元测试目录
    - integration
        - groovy
            - com.tehang.xxx.xxx : 集成测试目录
        - resources : 集成测试资源文件目录
    - resources : 单元测试资源文件目录
```

创建好项目目录之后，基本完成了项目的搭建，然后就可以开始着手项目的实际业务开发。

### 项目 build 基本方式

项目采用 gradle 打包、编译，所以再开始编译前，需要大体了解 gradle 的几个关键命令：`gralde wrapper`,`gradlw clean build`

- gradle wrapper ： 打包编译用的 gradle 包
- ./gradlew clean build ： 编译项目

为规范代码标准，项目引入了 pmd、checkstyle 等代码规范检查插件(在 build_scripts/quality_assurance 目录)，项目初始化时如已引入这些插件，建议更新这些插件。

更新方式：在项目的根目录执行下面的命令

```bash
git submodule update --init --recursive
```

如没有引入或遇到问题，请按文档进行操作 [引入代码检查规则](./quality_assurance.md)

### 项目开发

#### 项目的配置管理

我们的项目中，配置采用`yml`文件进行配置，并使用 springboot 的配置加载自动加载配置。后期可能会考虑使用 config center，目前而言，没有必要

```java
    @Value("${pkfare.timeout:60}")
    private long pkfareTimeout;
```

配置部分约定：

- 根据不同运行环境，配置不同的配置：除所有运行环境均使用的配置需要放到`application.yml`文件里面外，其他的配置都放置到 application 里面去
- 代码中，配置统一放到 config 目录的类中，进行统一管理，不允许单独注入到使用类当中
- 不同类型的配置，放到不同的配置类中：如`IBEConfig.java`，`PkfareConfig.java`

#### 日志

日志采用 sl4j + logback 记录日志。除了工具之外，还有几个比较中的：日志的格式、日志的记录地点、日志记录的内容、日志记录的级别、日志跟踪。

- 日志格式：`%date [%thread] [%X{TraceId}] %-5level %logger{80}.%M - %msg%n`
- 日志记录的地点： 记录到本地文件，不同类型日志，记录到不同文件，每日分割，最后日志由日志分析平台收集
- 日志记录的内容：能够清晰的描述日志记录点的数据、动作、结果
- 日志记录级别：线上只开通 info 级别，测试环境可以开通其他级别，后期开启动态调级的设置
- 日志跟踪：每个请求进入系统时，都会在其处理线程中添加一个线程本地变量`TraceId`，记录日志时，自动采用本变量，如果需要创建线程做一些异步操作，或者需要调用其他服务，都请将 traceId 带上

#### 异常处理

项目中，所产生的异常，我们都采用统一处理的方式进行处理，不能够在业务逻辑中把代码处理掉，但可以转换异常类型，抛出新的异常。
项目中，按照异常产生点，大体会分为两种类型的异常：业务前置异常、业务异常：

- 业务前置异常：此类异常大体时因为参数验证失败、权限验证失败等因素导致，此时还没有进入到 controller 中去
- 业务异常：这种类型的异常一般是因为业务处理失败，数据异常等因素导致

不同异常的处理方式：

- 业务前置异常：不用封装，提供统一处理
- 业务异常：项目中自定义异常类型，将业务中的异常情况转换成自定义异常，并统一处理

```bash
统一处理的最大的因素，是因为能够根据异常，返回不同的code到前端，而不用每个API自己处理
```

#### 接口定义

所有后端项目采用 REST API 的方式对外提供服务，数据通过 body 返回，采用 json 格式，错误码以 Http Code 为准，针对特殊情况，再采用自定义 code.
自定义 code 以 http code 为基础，后面补两位座位自定义 code :

例如：403 的意思是`拒绝访问`的意思，但是拒绝的原因却可能有多种，例如没有权限，token 失效等等。针对不同原因，可以定义自定义 code：`40301`,`40302`等错误类型，用于定位不同的错误类型。最多可在一种 http code 下定义 99 种错误类型

#### 服务内的执行流程

filters -> controller -> applicationService -> domainService -> domainRepository
`不允许逆向调用`

#### 内部服务服务调用

因所有的内部服务都通过 Rest API 对外暴露，所以内部服务的调用都采用 Rest call：`RestTemplate`

#### 外部服务调用

目前而言，项目中有不少的外部项目调用，统一采用`HttpClient`进行调用

## 单元测试与集成测试

初始化项目时需要引入单元测试、集成测试框架，具体步骤如下:

1. 在 build_scripts 目录下新建 test.gradle 文件.

````gradle
// 单元测试与集成测试的相关配置
apply plugin: 'org.unbroken-dome.test-sets'

testSets {
    // 指定集成测试的目录
    integrationTest { dirName = 'test/integration' }
}

check.dependsOn integrationTest
integrationTest.mustRunAfter test

// 单元测试的配置，必须使用test task，指定了单元测试的category
test {
    useJUnit {
        includeCategories 'com.tehang.tmc.services.UnitTest'            //请根据项目实际的GroupId进行适当更改
    }
    testLogging {
        showStandardStreams = true
    }
}

// 自定义的集成测试task，指定了集成测试的category
integrationTest {
    useJUnit {
        includeCategories 'com.tehang.tmc.services.IntegrationTest'    //请根据项目实际的GroupId进行适当更改
    }
    testLogging {
        showStandardStreams = true
    }
}
``` .

2. 修改build.gradle，加入以下内容:

apply from: 'build_scripts/test.gradle'

这样在build项目的时候会自动进行单元测试、集成测试.



3. 加入接口与基类

- 在test\groovy\com.tehang.xxx.xxx\目录下加入UnitTest接口文件及UnitTestSpecification基类文件.

- 在test\integration\groovy\com.tehang.xxx.xxx\目录下加入IntegrationTest接口文件及IntegrationTestSpecification基类文件.

4. 单元测试

所有单元测试代码位于 test/groovy 目录下，按照要测试的类或方法，放在对应的类中，并且必须继承UnitTestSpecification类.

5. 集成测试

所有集成测试代码位于 test/integration/groovy 目录下，并且必须继承IntegrationTestSpecification类.

6. 资源文件

单元测试、集成测试可能会引入资源文件，请在相应的resource目录下添加对应的application.yml文件.
````
