# 后端周总结内容

## 2018-12-07

### 建议：

- 按照当前的项目结构划分，不应将 Repository 作为参数传入 Entity/Bo 的创建方法中
- 开展团队 CODE REVIEW，促进成员代码风格统一
- 多多关注月度研发计划看板，跟紧测试验收de步调

### 标准：

- 关于LOG
  - 重要节点打上 INFO 级别的 LOG
    - 如 Controller、服务间调用方法、消息生产和消费处等，以及业务处理流程中需要标识的点
    - 如果数据量大，不应使用 INFO，应搭配使用 TRACE 记录数据
  - 其它地方使用 DEBUG 级别的 LOG
- 避免在循环中访问数据库，会造成大量的开销
- 严格执行测试及正式环境打包流程，详见[打包流程](https://github.com/cntehang/dev-docs/blob/master/backend/release-guideline.md)