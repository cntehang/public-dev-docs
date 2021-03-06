# Spring Boot 配置

## 展示/隐藏 Hibernate 生成的 SQL

此日志输出无法通过 logging.level.xx 来进行控制，需要在 `application.yml` 中配置如下项为 true（展示）或 false（隐藏）:

- spring.jpa.show_sql
