# 发布流程

![OSS](https://static.cnbetacdn.com/article/2018/0821/6516ac17b422c31.png)

在 test/prod 环境，需要进行代码与运行程序的版本化控制，目前项目使用 github 的 release 功能来做版本发布与管理，结合项目工程，制定该流程。

## 1. 发版流程

发布新版本的流程如下：

1. 发版

登录 github，在工程页面，点击 release -> Draft a new release -> 填写对应的 version tag(来自于 build_scripts/application_version.gradle 中的版本号)与 release title(与 version 一致)，从 github Wiki 中把该版本的 Page 文档修改历史拷贝到 describe 中 -> Publish release，如下：

![发布示例](../resources/release_example.png)

2. 通知运维发布新版，并确定发版成功
3. 版本迭代

发布之后，发布人需要把当前版本号更新迭代，如 v2.0.1 -> v2.0.2，迭代涉及以下工作：

- 修改 build_scripts/application_version.gradle 中的版本号
- 新建该版本对应的 sql 目录
- github 上，点击 Wiki -> New Page -> title 用版本号（版本号为最新的），初始化内容
- 把上述修改 push 并 merge 到 master 中

### 特别注意：

- release Tag version 是打包时实际上采用的版本号，应与 application_version.gradle 中的版本号一致，且与 Wiki 中 title 的版本字段一致
- release Title 与 release Tag version 保持一致
- release 描述文字，复制 Wiki 中的**更新内容**和**回滚操作**中的条目对齐即可

## 2. 版本相关文件

在工程中，与版本相关的几个文件，特此说明：

### 2.1 build_scripts/application_version.gradle

打包相关的版本信息，内部记录了对应的版本号，发版之后需要按照流程迭代版本号

```text
version = 'v2.0.1'
jar {
  archivesBaseName = 'tmc-services'
}
```

### 2.2 sql/vX_X_X

    对应版本需要执行的sql语句存放的目录，vX_X_X与版本号对应，由发版人员新建，开发人员在该版本使用的sql都存放在对应的目录下

### 2.3 Wiki Page

对应版本的修改历史，具体内容如下：

```text
# 版本升级基本内容
- 当前版本 v2.0.2
- 上一版本 v2.0.1
- 更新内容:
    - 登录流程中加入了验证码元素，当登录失败超过三次需要输入验证码，最后一次输入错误15分钟后可不用输入验证码
- 回滚操作
```

以上所写内容应概括本次提交中的要点，需注意以下几点

- 版本号由发版人在发布新版后迭代更新，与 application_version 中的版本号一致，在发布该版本之前不允许修改
- 开发过程中，每个 pr 都需要如实记录提交的内容，并且带上 pr 链接与 issue 链接
- 发布成功后，由发布人更新 Wiki Page 文档
