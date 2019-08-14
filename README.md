# 开发文档

---

备注： **这里只存放可以公开共享的、目前有效的文档，不要存放任何会议记录或具体项目相关的文档。包含有内部项目截屏、计划或人员信息的文档要存在内部私有库。**

持续的、高质效的软件开发能力是现代企业的核心竞争力。多年实践让我们意识到以下几点事实：

- 企业文化不是业务的一个方面，它就是业务。Culture isn't just one aspect of the game, it is the game. （郭士纳，Louis V. Gerstner, IBM 前总裁）
- 思想就是软件。 Mind is software. （老刘）
- 切忌随波逐流。 Only dead fish go with the flow.（西谚）
- 做人如果没有梦想，跟咸鱼有什么分别。 Salted fish has no dream. (星爷)
- 管理的本质是激发善意和潜能。The essence of management is to inspire goodwill and potential.（德鲁克）

用思想创造软件，进而改变世界的程序员心怀理想，责任重大，任劳任怨。可是环顾四周，多数软件团队的研发能力相对计算机的巨大潜力和广泛的业务需求有巨大鸿沟，开发效率低，软件的质量令人忧伤。稍感安慰的是半个多世纪的编程历史积累了一些基本原则和最佳实践（best practices)。遵守基于这些原则和最佳实践能大大改善程序员的工作效率和工作氛围。

## 出发点

[程序员工作原则](./principles.md)给出了我们的开发理念和基本原则。这些原则都是为了达成软件开发的二个根本目标：正确的业务逻辑与可维护性。

软件开发管理活动，如自动测试要求，代码标准，代码审核，文档标准，github 工作流，需求工作流程，设计工作流程，开发环境配置等等都是基于这些原则来展开。我们明白这些流程、标准、模版、规则不是铁一样的限制，有足够好的理由，任何规定都可以按实际情况来改变或取消。标准化和自动化流程的结果是让开发人员把精力放在最有价值的创造性工作中。

工作的高效率来自高标准自我要求和务实的合作精神。企业文化是企业提倡的团队价值观。所有的流程和做事方法都尊从我们提倡的[企业文化](./team/culture.md)。

## 文档的创建和维护

可以和外部共享的、可以标准化的流程和最佳实践都存放这里。

所有的工作都必须符合流程规则要求。如果规则或最佳实践不再适用，要先更新文档再按新流程执行。更新流程和最佳实践是所有开发人员的责任。

有很多文档或需要说明的目录应该有一个`README.md`说明文件。这个文件描述了目录下的所有文件并保持更新。当目录下的文件和文件夹超过十个时，需要按分类创建子目录。

所有文件的创建和更新需要创建 PR 和通过 Review。然后通知所有相关人员按新规则执行。不影响软件开发活动的简单的笔误更正和内容改善可以直接提交。

文档统一采用 Markdown 格式。使用 VS Code 并用 [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint)来保证 markdown 文件风格的一致性。

## 目录和文件说明

- [程序员工作原则](./principles.md)：我们的工作理念和原则
- [team](./team/README.md)： 企业文化。
- [coding-guide](./coding-guide/README.md): 编程指南，包括 Git 工作流、代码审核和如何写日志等。
- [frontend](./frontend/README.md)：前端的技术文档。
- [backend](./backend/README.md): 后端的技术文档。
- [backend-and-frontend](./backend-and-frontend/README.md)：前后端接口相关的技术文档
- [pm](./pm/README.md): 产品经理和项目管理文档。
- [learning](./learning/README.md): 程序员学习指南。
