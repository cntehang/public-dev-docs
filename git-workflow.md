# Git 工作流程

基于一个流行的 [项目开发指南](https://github.com/elsewhencode/project-guidelines)，本文描述了一个建议的 github 工作流程。具体项目可以定制工作流程。项目初期可以采用简化的工作流。比如，一个项目在初期只有一、二个人的时候，所有操作都在 master 分支 -- 但是上线前必须有 develop 分支。

## 1. 一些原则

这里有一套规则要牢记：

* 在功能分支中执行开发工作。

  _为什么：_

  > 因为这样，所有的工作都是在专用的分支而不是在主分支上隔离完成的。它允许您提交多个 pull request 而不会导致混乱。您可以持续迭代提交，而不会使得那些很可能还不稳定而且还未完成的代码污染 master 分支。

* 从 `develop` 独立出分支。

  _为什么：_

  > 这样，您可以保持 `master` 分支中的代码稳定性，这样就不会导致构建问题，并且几乎可以直接用于发布（当然，这可能对某些项目来说要求会比较高）。

* 永远也不要将分支（直接）推送到 `develop` 或者 `master` ，请使用合并请求（Pull Request）。

  _为什么：_

  > 通过这种方式，它可以通知整个团队他们已经完成了某个功能的开发。这样开发伙伴就可以更容易对代码进行 code review，同时还可以互相讨论所提交的需求功能。

* 在推送所开发的功能并且发起合并请求前，请更新您本地的`develop`分支并且完成交互式变基操作（interactive rebase）。

  _为什么：_

  > rebase 操作会将（本地开发分支）合并到被请求合并的分支（ `master` 或 `develop` ）中，并将您本地进行的提交应用于所有历史提交的最顶端，而不会去创建额外的合并提交（假设没有冲突的话），从而可以保持一个漂亮而干净的历史提交记录。 [合并（merge）和变基（rebase）的比较](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

* 请确保在变基(rebase)并发起合并请求之前解决完潜在的冲突。

* 合并分支后删除本地和远程功能分支。

  _为什么：_

  > 如果不删除需求分支，大量僵尸分支的存在会导致分支列表的混乱。而且该操作还能确保有且仅有一次合并到`master` 或  `develop`。只有当这个功能还在开发中时对应的功能分支才存在。

* 在进行合并请求之前，请确保您的功能分支可以成功构建，并已经通过了所有的测试（包括代码规则检查）。

  _为什么：_

  > 因为您即将将代码提交到这个稳定的分支。而如果您的功能分支测试未通过，那您的目标分支的构建有很大的概率也会失败。此外，确保在进行合并请求之前应用代码规则检查。因为它有助于我们代码的可读性，并减少格式化的代码与实际业务代码更改混合在一起导致的混乱问题。

* 保护您的 `develop` 和 `master` 分支。

  _为什么：_

  > 这样可以保护您的生产分支免受意外情况和不可回退的变更。

## 2. 建议的工作流

基于以上原因, 我们将 [功能分支工作流](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow) ， [交互式变基的使用方法](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing) 结合一些 [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows#gitflow-workflow)中的基础 (比如，命名和使用一个 develop branch)一起使用。 主要步骤如下:

* 针对一个新项目, 在您的项目目录初始化您的项目。 **如果是（已有项目）随后的功能开发/代码变动，这一步请忽略**。

  ```sh
  cd <项目目录>
  git init
  ```

* 检出（Checkout） 一个新的功能或故障修复（feature/bug-fix）分支。

  ```sh
  git checkout -b <分支名称>
  ```

* 新增代码变更。

  ```sh
  git add
  git commit -a
  ```

  _为什么：_

  > `git commit -a` 会独立启动一个编辑器用来编辑您的说明信息，这样的好处是可以专注于写这些注释说明。请参考下面关于说明信息的要求。

* 保持与远程（develop 分支）的同步，以便（使得本地 develop 分支）拿到最新变更。

  ```sh
  git checkout develop
  git pull
  ```

  _为什么：_

  > 当您进行（稍后）变基操作的时候，保持更新会给您一个在您的机器上解决冲突的机会。这比（不同步更新就进行下一步的变基操作并且）发起一个与远程仓库冲突的合并请求要好。

* （切换至功能分支并且）通过交互式变基从您的 develop 分支中获取最新的代码提交，以更新您的功能分支。

  ```sh
  git checkout <branchname>
  git rebase -i --autosquash develop
  ```

  _为什么：_

  > 您可以使用 `--autosquash` 将所有提交压缩到单个提交。没有人会愿意（看到） `develop` 分支中的单个功能开发就占据如此多的提交历史。 [更多请阅读...](https://robots.thoughtbot.com/autosquashing-git-commits)

* 如果没有冲突请跳过此步骤，如果您有冲突, 就需要[解决它们](https://help.github.com/articles/resolving-a-merge-conflict-using-the-command-line/)并且继续变基操作。

  ```sh
  git add <file1> <file2> ...
  git rebase --continue
  ```

* 推送您的（功能）分支。变基操作会改变提交历史, 所以您必须使用 `-f` 强制推送到远程（功能）分支。 如果其他人与您在该分支上进行协同开发，请使用破坏性没那么强的 `--force-with-lease` 参数。

  ```sh
  git push -f
  ```

  _为什么:_

  > 当您进行 rebase 操作时，您会改变功能分支的提交历史。这会导致 Git 拒绝正常的 `git push` 。那么，您只能使用 `-f` 或 `--force` 参数了。[更多请阅读...](https://developer.atlassian.com/blog/2015/04/force-with-lease/)

* 提交一个合并请求（Pull Request）。
* Pull Request 会被负责代码审查的同事接受，合并和关闭。
* 如果您完成了开发，请记得删除您的本地分支。

  ```sh
  git branch -d <分支>
  ```

  （使用以下代码）删除所有已经不在远程仓库维护的分支。

  ```sh
  git fetch -p && for branch in `git branch -vv | grep ': gone]' | awk '{print $1}'`; do git branch -D $branch; done
  ```

## 3 如何写好 Commit Message

坚持遵循关于提交的标准指南，会让在与他人合作使用 Git 时更容易。这里有一些经验法则 ([来源](https://chris.beams.io/posts/git-commit/#seven-rules)):

* 用新的空行将标题和主体两者隔开。

  _为什么：_

  > Git 非常聪明，它可将您提交消息的第一行识别为摘要。实际上，如果您尝试使用 `git shortlog` ，而不是 `git log` ，您会看到一个很长的提交消息列表，只会包含提交的 id 以及摘要（，而不会包含主体部分）。

* 将标题行限制为 50 个字符，并将主体中一行超过 72 个字符的部分折行显示。

  _为什么：_

  > 提交应尽可能简洁明了，而不是写一堆冗余的描述。 [更多请阅读...](https://medium.com/@preslavrachev/what-s-with-the-50-72-rule-8a906f61f09c)

* 标题首字母大写。
* 不要用句号结束标题。
* 在标题中使用 [祈使句](https://en.wikipedia.org/wiki/Imperative_mood) 。

  _为什么：_

  > 与其在写下的信息中描述提交者做了什么，不如将这些描述信息作为在这些提交被应用于该仓库后将要完成的操作的一个说明。[更多请阅读...](https://news.ycombinator.com/item?id=2079612)

* 使用主体部分去解释 **是什么** 和 **为什么** 而不是 **怎么做**。
