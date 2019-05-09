# 数据库连接池

几乎所有的商业应用都有大量数据库访问，通常这些应用会采用数据库连接池。理解问什么需要连接池，连接池的实现原理和参数设置对于写出正确、高效的程序很有帮助。理解这些概念对于理解分布式计算也很有帮助。

## 为什么需要连接池

任何数据库的访问都需要建立连接。这是一个复杂、缓慢的处理。牵涉到通信连接建立（包括 TCP 的三次握手）、认证、授权、资源的初始化和分配等一系列任务。而且数据库服务器通常和应用服务器是分开的，所有的操作都是分布式网络请求和处理。[建立数据库连接时间](https://stackoverflow.com/questions/2188611/how-long-does-it-take-to-create-a-new-database-connection-to-sql)通常在 100ms 或更长。而通常小数据的 CRUD 数据库操作是 ms 级或更短，加上网络延迟一般 10 到 50 个 ms 就可以返回多数请求处理结果。在应用启动时预先建立一些数据库连接，应用程序使用已有的连接可以极大提高相应速度。另外，Web 服务应用当客户很多时，有很多线程，连接数目过多以及频繁创建/删除连接也会影响数据库的性能。

总结起来，采用数据库连接有如下好处：

- 节省了创建数据库连接的时间，通常这个时间大大超过处理数据访问请求的时间。
- 统一管理数据库请求连接，避免了过多连接或频繁创建/删除连接带来的性能问题。

## 实现原理

如同多数分布式基础构件，连接池的原理比较简单，但是牵涉到数据库，操作系统，编程语言，运维以及应用场景的不同特点，具体实现比较复杂。从数据库诞生就有的广泛需求，半个世纪后还有不断改进提高的余地。

原理上，在应用开始时创建一组数据库的连接。也可以动态创建但是复用已有的连接。这些连接被存储到一个共享的数据结构，称为连接池。每个线程在需要访问数据库时借用（borrow）一个连接，使用完成则释放（release）连接回到连接池供其他线程使用。比较好的线程池构件会有二个参数动态控制线程池的大小：最小数量和最大数量。最小数量指即使负载很轻，也保持一个最小数目的数据库连接以备不时之需。当同时访问数据库的线程数超过最小数量时，则动态创建更多连接。最大数量则是允许的最大数据库连接数量，当最大数目的连接都在使用而有新的线程需要访问数据库时，则新的线程会被阻塞直到有连接被释放回连接池。当负载变低，池里的连接数目超过最小数目而只有低于或等于最小数目的连接被使用时，超过最小数目的连接会被关闭和删除以便节省系统资源。

具体的线程池实现需要考虑很多应用细节。

- 多余的连接不会立即关闭，而是会等待一段空闲时间（idle time）再关闭。
- 连接有最长生命时间限制，即使连接池不管，数据库也会自动关闭超过生命时间的连接。在 MySql 里面，连接最长生命时间是 8 个小时。连接池需要定期监控清理无效的连接。
- 当连接数小于最大数目而需要为新线程创建连接时，新线程应该等待池里第一个可用的连接而不必等待因它而创建的线程。HikariCP 的文档[Welcome to the Jungle](https://github.com/brettwooldridge/HikariCP/blob/dev/documents/Welcome-To-The-Jungle.md) 比较了这种实现的优点：可以避免创建很多不必要的连接并且有更好的性能。
- 数据库各种异常的处理。[Bad Behavior: Handling Database Down](https://github.com/brettwooldridge/HikariCP/wiki/Bad-Behavior:-Handling-Database-Down) 给出里不同连接池构件实现对于线程阻塞 timeout 的不同处理方式。很多不能正确处理。
- 线程池的监控监测。[HikariCP Dropwizard HealthChecks](https://github.com/brettwooldridge/HikariCP/wiki/Dropwizard-HealthChecks)是一个例子。
- 线程池的性能检测。[HikariCP Dropwizard Metrics](https://github.com/brettwooldridge/HikariCP/wiki/Dropwizard-Metrics) 给出了检测的性能指标。
- 线程阻塞的机制以及相关数据结构对连接池的性能有很大影响。[Down the Rabbit Hole](https://github.com/brettwooldridge/HikariCP/wiki/Down-the-Rabbit-Hole)给出了 Java 里的优化方法。当然，里面有很多优化过于琐碎使得代码晦涩难懂而且需要很多维护。

## 配置数据库连接池

先说句题外话，老刘没有想到这么简单的一个配置问题竟然多数人都不清楚甚至搞错，这些糊涂的人甚至包括负责连接池编码的程序员。[About Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)的作者是广泛使用的 HikariCP 的程序员。很不幸，有实现连接池的编码经验，面对很多数据，他给出里错误的结论。不过，他的糟糕的代码风格也预示着他的条理性不好。
