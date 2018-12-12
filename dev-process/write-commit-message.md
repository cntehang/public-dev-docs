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

建议标题的尽可能用英文，可以参考如下类别：

- Add = Create a capability e.g. feature, test, dependency.
- Cut = Remove a capability e.g. feature, test, dependency.
- Fix = Fix an issue e.g. bug, typo, accident, misstatement.
- Bump = Increase the version of something e.g. dependency.
- Make = Change the build process, or tooling, or infra.
- Start = Begin doing something; e.g. create a feature flag.
- Stop = End doing something; e.g. remove a feature flag.
- Refactor = A code change that MUST be just a refactoring.
- Reformat = Refactor of formatting, e.g. omit whitespace.
- Optimize = Refactor of performance, e.g. speed up code.
- Document = Refactor of documentation, e.g. help files.

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
