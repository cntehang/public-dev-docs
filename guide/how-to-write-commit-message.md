# Write Good Commit Message

写出好的提交信息有助于提高代码可维护性。提交信息包含三个部分：标题(Subject)，主体(Body)和相关 Issue。各个部分之间用空行隔开。参考了[write good git commit meesage](https://juffalow.com/other/write-good-git-commit-message).

## 提交信息建议

### 标题

好的标题应该回答这个问题： 如果生效，这个提交（commit）""
在空的地方应该填入动词加名词的一个简单句子。不超过 50 个字母。

好的例子：

- 如果生效，这个提交（commit）“删除多余的文件”
- 如果生效，这个提交（commit）“可以选择多个用户地址”
- 如果生效，这个提交（commit）"fix bug when network is disconnected"

不好的例子

- 如果生效，这个提交（commit）“fix bug"
- 如果生效，这个提交（commit）"user group"

#### 标题行的基本规范
```html
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

对格式的说明如下：

type代表某次提交的类型，比如是修复一个bug还是增加一个新的feature。所有的type类型如下：
- feature： 新增feature
- fix: 修复bug
- docs: 仅仅修改了文档，比如README, CHANGELOG, CONTRIBUTE等等
- style: 仅仅修改了空格、格式缩进、都好等等，不改变代码逻辑
- refactor: 代码重构，没有加新功能或者修复bug
- performance: 优化相关，比如提升性能、体验
- test: 测试用例，包括单元测试、集成测试等
- chore: 改变构建流程、或者增加依赖库、工具等
- revert: 回滚到上一个版本

建议标题的尽可能用英文，可以参考如下类别：

- feature: Create a capability e.g. feature, test, dependency.
- fix: Fix an issue e.g. bug, typo, accident, misstatement.
- chore: Change the build process, or tooling, or infra.
- refactor: A code change that MUST be just a refactoring.
- style: Refactor of formatting, e.g. omit whitespace.
- performance: Refactor of performance, e.g. speed up code.
- docs: Refactor of documentation, e.g. help files.

示例：`feat(addition): big integer addition function`

### 主体

描述为什么做这个提交以及代码完成了哪些具体功能和做了哪些改动。主体包含多行, 行长度不要超过 72 个字符。

### Issue

列出相关的 Issue Number。

## 例子

```txt
Fix error when protocol is missing

First, it checks if the protocol is set. If not, it changes the url and
add the basic http protocol on the beginning.
Second, it does a "preflight" request and follows all redirects and
returns the last URL. The process then continues with this URL.

Resolves: #56, #78
See also: #12, #34
```
