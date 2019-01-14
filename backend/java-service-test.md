# Java 微服务测试框架以及方法

## 单元测试

当前 Java 服务采用的是 Spock 进行测试
具体文档可以参考： [SPOCK官方文档](http://spockframework.org/)

### API测试用例

- 如果API的条件是由严变松（如：not null --> nullable ），并增加null的测试（测试用例、测试脚本都可以）
- 如果API的条件是由松变严(如：nullable --> not null)，那么可以沿用之前的测试用例

### 其他原则

- 测试数据以json文件存放，且json文件必须格式化

## 集成测试

当前 Java 服务集成测试采用的是 WireMock 框架，具体文档可以参考：[WireMock官方文档](http://wiremock.org/docs/)

### 