# 是否要用 Redis

团队采用 Redis 缓存了用户 Session 和一些其他基础数据和一些查询结果。这是个大的系统结构设计，需要仔细权衡收益和成本。

## 问题

考虑到在 MySQL 之外再采购部署一套 Redis 会产生如下的问题：

- 额外的费用：相同的内存配置，和 MySQL 数据库服务价钱差不多。
- 多了一个系统失效节点：本来只有 MySQL，现在如果 Redis 失效，则整个业务系统也无法运行。
- 额外的 API：需要学习使用多一套 API。
- 数据同步难题：如果将 MySQL 查询结果放到 Redis，数据的同步是个难题。
- 另外的部署：需要增加一套账户和配置参数。

## 答案

具体到我们的具体应用，上述问题的答案如下：

- 额外的费用：即使和数据库费用差不多，但是在 MySQL 实现那些功能更花钱而且不稳定。
- 多了一个系统失效节点：阿里云有分布式可靠措施，Redis 也比较成熟，可以信赖。
- 额外的 API：团队使用 Redis 的程序员普遍认为其 API 简单易用。而且类似 Hash， Set， List 这样的数据结构在我们应用里有许多应用场景。
- 数据同步难题：尽量避免 MySQL 和 Redis 需要同步的使用。
- 另外的部署：这是一次性的开支，可以基本忽略。

## 结论

在我们的应用里面有许多地方 Redis 带来很大的方便，尤其是很多只读基础数据和不需要长期保存的数据（比如 Session Token， 短信验证码等）， Redis 的 API 非常简单好用而且性能高，系统稳定性非常好。相同的功能如果自己实现成本更高而且稳定性不好。

所以可以把 Redis 做为我们的一个基础服务设施使用。根据使用场景，可以把使用中的陷阱和最佳实践写到文档。
