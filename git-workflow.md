# Git 工作流程

基于一个流行的 [项目开发指南](https://github.com/elsewhencode/project-guidelines)，本文描述了一个建议的 github 工作流程。和上述指南最大的不同是我们采用 master 分支作为主开发分支。另外创建专门的上线发布 release 分支。这样的好处是 master 在 github 是缺省的分支，变基，PR 和合并都比较自然。而且上线发布分支只有少数人关注，对开发人员越隐蔽越好。

具体项目可以定制工作流程。项目初期可以采用简化的工作流。比如，一个项目在初期只有一、二个人的时候，所有操作都在 master 分支 -- 但是上线前必须有 release 分支。

分支类型：

- release 分支：该分支在第一次上线之后，永远保持可随时上线状态，不允许直接在这个上面进行更改或者提交代码，只能从其他分支 merge
- master 分支：master 分支主要用于新版本的开发，代码第一次上线时，从 master 创建出 release 分支用于上线发布。所有后期版本开发都在 master 分支上进行。版本开发完毕之后，上线前，将 master 的修改合并到 release，并做上线测试
- 功能分支：每个版本的开发，会有很多独立功能，每个独立功能的开发，从 master 创建一个功能分支，并在功能分支上进行功能的开发，最后变基合并到 master。
- hotfix 分支：这种类型的分支是针对线上 bug 的紧急 fix，直接从 release 创建分支。修复，测试完毕之后，合并到 release 上线。然后相应应该也并入 master 分支。

一些原则：

- PR 每个 PR 都只是针对一个功能，不能够太大, 原则上不超过 10 个文件

## 1. 基本规则

这里有一套规则要牢记：

- 在功能分支中执行开发工作。

为什么

> 因为这样，所有的工作都是在专用的分支而不是在主分支上隔离完成的。它允许您提交多个 合并请求 pull request（PR）而不会导致混乱。您可以持续迭代提交，而不会使得那些很可能还不稳定而且还未完成的代码污染 master 分支。

- 从 `master` 独立出分支 `release` 分支用于上线发布。

为什么

> 这样，您可以保持 `release` 分支中的代码稳定性，随时可以直接用于发布。

- 永远也不要将分支（直接）推送到 `master` ，请使用合并请求（Pull Request）。

为什么

> 通过这种方式，它可以通知整个团队他们已经完成了某个功能的开发。这样开发伙伴就可以更容易对代码进行 code review，同时还可以互相讨论所提交的需求功能。

- 在推送所开发的功能并且发起合并请求前，请更新您本地的`master`分支并且完成交互式变基操作（interactive rebase）。

为什么

> rebase 操作会将功能合并到被请求合并的 master 分支，并将您本地进行的提交应用于所有历史提交的最顶端，而不会去创建额外的合并提交（假设没有冲突的话），从而可以保持一个漂亮而干净的历史提交记录。 [合并（merge）和变基（rebase）的比较](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

- 请确保在变基(rebase)并发起合并请求之前解决完潜在的冲突，合并分支后删除本地和远程功能分支。

为什么

> 如果不删除需求分支，大量僵尸分支的存在会导致分支列表的混乱。而且该操作还能确保有且仅有一次合并到`master`。只有当这个功能还在开发中时对应的功能分支才存在。

- 在进行合并请求之前，请确保您的功能分支可以成功构建，并已经通过了所有的测试（包括代码规则检查）。

为什么

> 因为您即将将代码提交到这个稳定的分支。而如果您的功能分支测试未通过，那您的目标分支的构建有很大的概率也会失败。此外，确保在进行合并请求之前应用代码规则检查。因为它有助于我们代码的可读性，并减少格式化的代码与实际业务代码更改混合在一起导致的混乱问题。

- 保护您的 `master`，尤其是 `release` 分支改动需要特别授权。

为什么

> 这样可以保护您的生产分支免受意外情况和不可回退的变更。

## 2. 建议的工作流

基于以上原因, 我们将 [功能分支工作流](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow) ， [交互式变基的使用方法](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing) 结合一些 [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows#gitflow-workflow)中的基础功能一起使用。 主要步骤如下:

### 2.1 项目初始建立`master` 和 `release`二个分支

- 针对一个新项目, 在 github 上创建项目的 repo，同时用 github 界面创建 `release` 发行分支仅用于新版本发布上线。 `clone`到本地后，只基于 `master` 分支做开发：

```sh
git clone <项目地址> # clone the remote repository
```

### 2.2 创建功能分支做具体开发

- 第一步：检出（Checkout）一个新的功能或故障修复（feature/bug-fix）分支或用 github 界面基于`master`创建功能分支。下面是命令行创建功能分支并同步到服务器。随时用 `git branch -a`检查当前分支状态。

```sh
git checkout -b my-feature # create a new branch my-feature from the current branch
git push -u origin my-feature-branch # sync to remote server
git branch -a # display branch status
```

- 第二步：在功能分枝上进行新功能的开发，提交代码并随时同步到远程 Git 服务器做备份。

```sh
git add . # Add all local changes
git commit -a # commit all changes
git push # push to remote frequently to bakcup changes
```

为什么

> `git commit -a` 会独立启动一个编辑器用来编辑您的说明信息，这样的好处是可以专注于写这些注释说明。请参考下面关于说明信息的要求。经常同步到远程库做备份。通常在 IDE 里执行上面三个操作也可以，注意当前分支为功能分支就好。

### 2.3 合并功能分支到 `master` 分支

当功能开发完成时，要将所做工作变基并入 `master` 分支。

- 第一步：在准备提交合并前， 先将需要 `master` 分支更新到最新。下面的步骤建议手工运行。

```sh
git checkout master
git pull
```

为什么

> 更新 `master` 代码版本有助于发现可能的冲突。当您进行（稍后）变基操作的时候，保持更新会给您一个在您的机器上解决冲突的机会。这比（不同步更新就进行下一步的变基操作并且）发起一个与远程仓库冲突的合并请求要好。

- 第二步：切换至功能分支，把功能分支变基到`master`分支，建议采用`rebase -i --autosquash`的交互方式

```sh
git checkout my-feature
git rebase -i --autosquash master
```

为什么

> 您可以使用 `--autosquash` 将所有提交压缩到单个提交。没有人会愿意（看到） `master` 分支中的单个功能开发就占据如此多的提交历史。 [更多请阅读...](https://robots.thoughtbot.com/autosquashing-git-commits)

- 第三步（可能需要）：这一步在没有代码冲突可以跳过。如果您有代码合并冲突, 就需要[解决它们](https://help.github.com/articles/resolving-a-merge-conflict-using-the-command-line/)。

```sh
git add <file1> <file2> ... # 任何必要的增删改
git commit -a # 提交所有修改
git rebase --continue # 继续刚才的变基操作
```

- 第四步：推送您的功能分支到 github。变基操作会改变提交历史, 所以您必须使用 `-f` 强制推送到远程（功能）分支。如果其他人与您在该分支上进行协同开发，请使用破坏性没那么强的 `--force-with-lease`。

```sh
git push -f
```

为什么

> 当您进行 rebase 操作时，您会改变功能分支的提交历史。下一步的合并请求（PR）是基于远程库进行的。这一步把本地的变基操作及修改同步到远程库。由于变基会导致 Git 拒绝正常的 `git push` 。能使用 `-f` 或 `--force` 或当多人在同一分支合作时用 `--force-with-lease` 参数了。[更多请阅读...](https://developer.atlassian.com/blog/2015/04/force-with-lease/)

- 第五步：提交一个合并请求（Pull Request）。Pull Request 会被负责代码审查的同事接受，合并和关闭。合并请求完成同时需要删除远程的功能分支。这些操作都利用 github 的用户界面进行。如果代码需要进一步的修改完善，请回到第一步。

为什么

> 只有经过授权的人做代码的合并维护可以保护代码库的稳定可靠。代码是开发团队最宝贵的资源。

- 第六步：合并完成后，记得删除您的本地分支。

```sh
git checkout master
git branch -d <分支>
```

（使用以下代码）删除所有已经不在远程仓库维护的分支。

```sh
git checkout master
git fetch -p
git branch -D featureBranch
```

### 2.4 版本发布到`release`分支

由程序员、运维人员和项目经理共同决定发布的时机。具体流程另外描述。

## 3 如何写好 Commit Message

坚持遵循关于提交的标准指南，会让在与他人合作使用 Git 时更容易。这里有一些经验法则 ([来源](https://chris.beams.io/posts/git-commit/#seven-rules)):

- 用新的空行将标题和主体两者隔开。

为什么

> Git 非常聪明，它可将您提交消息的第一行识别为摘要。实际上，如果您尝试使用 `git shortlog` ，而不是 `git log` ，您会看到一个很长的提交消息列表，只会包含提交的 id 以及摘要（，而不会包含主体部分）。

- 将标题行限制为 50 个字符，并将主体中一行超过 72 个字符的部分折行显示。

为什么

> 提交应尽可能简洁明了，而不是写一堆冗余的描述。 [更多请阅读...](https://medium.com/@preslavrachev/what-s-with-the-50-72-rule-8a906f61f09c)

- 标题首字母大写。
- 不要用句号结束标题。
- 在标题中使用 [祈使句](https://en.wikipedia.org/wiki/Imperative_mood) 。

为什么

> 与其在写下的信息中描述提交者做了什么，不如将这些描述信息作为在这些提交被应用于该仓库后将要完成的操作的一个说明。[更多请阅读...](https://news.ycombinator.com/item?id=2079612)

- 使用主体部分去解释 **是什么** 和 **为什么** 而不是 **怎么做**。
