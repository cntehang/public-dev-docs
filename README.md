# 开发规则与文档

软件开发有二个根本性原则：正确的业务逻辑与可维护性。

软件开发管理活动，如自动测试要求，代码标准，代码审核，文档标准，github 工作流，需求工作流程，设计工作流程，开发环境配置等等都是基于这二个原则来展开。我们明白这些流程、标准、模版不是限制，任何规则都可以定制，目的是让整个开发过程自动化和标准化，从而让开发人员把精力放在最关键的创造性工作中。

高效工作来自我高标准驱动与合作精神。

## 文档的创建和维护

所有开发相关的，可以标准化的流程和最佳实践都存放这里。所有的工作都必须符合流程规则要求。如果规则或最佳实践不再适用，要先更新文档再按新流程执行。更新流程和最佳实践是所有开发人员的责任。这里只存放目前有效的技术文档，不要存放任何会议记录。

所有目录都有一个`README.md`说明文件。这个文件描述了目录下的所有文件并保持更新。当目录下的文件和文件夹超过十个时，需要按分类创建子目录。

所有文件的创建和更新需要创建 PR 和通过 Review。然后通知所有相关人员按新规则执行。不影响软件开发活动的简单的笔误更正和内容改善可以直接提交。

文档统一采用 Markdown 格式。建议采用 VS Code 并用 [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint)来保证 markdown 文件风格的一致性。

## 文档说明

- [程序员工作指南](./developer-working-guide.md)：我们的开发理念
- [backend](./backend/README.md): 后端的技术文档
- [backend-and-frontend](./backend-and-frontend/README.md)：前后端接口相关的技术文档
- [dev-process](./dev-process/README.md)：开发流程规则
- [frontend](./frontend/README.md)：前端的技术文档
- [test](./test/README.md)：测试相关技术文档
