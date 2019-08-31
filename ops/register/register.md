# Register 的原理&实战

## 什么是 Register

Register，中文叫注册中心

- 服务发现
  - 告诉你，你要请求的服务在哪里
- 负载均衡

## 为什么要用 Register，不用是否可行

## Register 的基本原理是什么

## 自己实现一个 Register

- 存储：存储 service 的信息
- 注册：更新 service 的信息
- 下线：删除 Service 的信息
- 通知：服务信息有更新时，如何及时通知到 Client 端

额外的问题：

- 如何保证 Register 的稳定和可靠性

## 可有的现成选择

| 选项      | 维护组织                | 稳定性             | 是否方便使用 | 支持语言     | 支持的功能 |
| --------- | ----------------------- | ------------------ | ------------ | ------------ | ---------- |
| Eureka    | Netflix（2.0 停止更新） | 老牌服务，稳定可靠 | ------------ | --------     |
| Consul    | HashiCorp （升级活跃）  | ------             | ----         | ------------ | --------   |
| Zookeeper | Hadoop 的正式子项目     | ------             | ----         | ------------ | --------   |
| Etcd      | 深受大厂喜爱的工具之一  | ------             | ----         | ------------ | --------   |

## 我们的选择

## 我们的实际应用

## 可能存在的问题

### Register 单点失效

### Register 集群的数据同步问题

服务的注册、下线一般是向其中一个服务请求，但是如何在 Register 的集群内保持数据同步呢？最终一致性？时间窗口期是多少？如何处理？hadoop 的分片机制么？

### 如何兼容其他语言的服务

## 遗留的有趣问题

- 如何跨区访问服务：不同数据区的数据访问（国内、国外；南、北）
- Register 还可以用来做什么

https://blog.csdn.net/lchq1995/article/details/87931851

## Consul 说明

```java
NewService{
  id='test-service-10061',
  name='test-service',
  tags=[secure=false],
  address='192.168.1.157',
  meta=null,
  port=10061,
  enableTagOverride=null,
  check=Check{
    script='null',
    interval='15s',
    ttl='null',
    http='http://192.168.1.157:10061/actuator/health',
    method='null',
    header={},
    tcp='null',
    timeout='null',
    deregisterCriticalServiceAfter='null',
    tlsSkipVerify=null,
    status='null'
  },
  checks=null
  }
```
