# 开发工作流

一套基于gitlab + docker的开发工作流，gitlab用于代码的版本管理，docker用于环境的构建，最终的交付产物是镜像（代码+环境）。

### 版本管理的挑战

虽然git的代码管理非常出色，但是我们面对版本管理的时候，依然有非常大得挑战，我们都知道大家工作在同一个仓库上，那么彼此的代码协作必然带来很多问题和挑战，如下：

1. 如何开始一个Feature的开发，而不影响别的Feature？
2. 由于很容易创建新分支，分支多了如何管理，时间久了，如何知道每个分支是干什么的？
3. 哪些分支已经合并回了主干？
4. 线上代码出Bug了，如何快速修复？而且修复的代码要包含到开发人员的分支以及下一个Release?

随着人员的增加，而且项目周期一长就会出现各种问题。因此，**就像代码需要代码规范一样，代码管理同样需要一个清晰的流程和规范**。在参考了gitflow和gitlabfolw后我们采用了gitlabflow并进行了一些细微的调整。

### 分支开发流程

![workflow](/images/workflow.png)

#### 分支介绍

* Master 分支

  master分支是一个常驻分支，使我们的默认分支，是所有分支的源头，所有的其他分支都是源于master，master也是CI/CD的触发分支。


* Feature 分支系列

  这个系列的分支主要是基于一个需求（issues）或者一系列需求创建的**开发分支**，一旦开发完成将会合并回master。一般合并上线后，会删除掉这个分支。

  > 需要注意的是，开发分支可以以issue或者feature开头，例如：issue-test、feature-test。
  >
  > 通过gitlab ui 拉出来的分支则是以issue的编号开头的，例如：1-test。

#### 工作方式

1. 发现bug或提出新需求时需要到issues列表中提交issue，填写issue内容，选择issue级别，指定响应的开发人员。

2. 开发人员收到issue后，创建开发分支。开发分支的命名规则为`feature(或者issue) - issue编号 - 分支名称`，例如：feature-7-login-find-pwd、issue-7-login-find-pwd。

3. [构建本地开发环境](/dev/develop.md)进行开发，在开发完成后首先进行自测，如果需要进行联调，参考[联调流程]()。

4. 在完成开发分支的开发，联调，测试后，需要发起到master的merge request，审核人员review代码通过后，触发CI集成测试，并反馈测试结果。

   > 1. 从commit的message中或者是merge request的信息中可以关联相关的issue,具体语法是:“fixes \#7”或者“closes \#7”.这样会在issue中创建一个评论，并且merge request会显示相关联的issue.如果这个merge request被接受，这个issue会自动被关闭.
   > 2. 假如你仅仅想关联这个问题，但是不关闭这个问题，你可以这样写：\#7

   当master上的代码通过CI后，首先会构建当前版本的镜像，将镜像推送到线下镜像仓库中，之后自动更新稳定测试环境。

   ![合并流程](/images/merge.png)

5. 当准备发布上线的时候，可通过上一步合并触发的`pipelines`中人工发布的操作按钮，完成项目的生产发布。

#### gitlab使用注意事项

##### 1.代码什么时候提交到主分支

强调频繁地（一天多次）将代码集成到主干的目的是什么：

1. 快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易。
2. 防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。简单一点来说就是功能开发完成，就可以提交到主分支了。但是功能开发完成的标志点是，测试通过、review 完成。

##### 2.使用rebase来梳理版本历史记录

在merge之前使用rebase命令来梳理版本记录

##### 3.使用tag标记版本

当一个里程碑完成后或者一个里程碑中一个重要的功能点完成后，最好使用tag标记master分支当前的版本。

### 相关参考

[gitlabfolw官方文档](https://docs.gitlab.com.cn/ce/workflow/gitlab_flow.html#introduction)

[gitflow提出者的原文](http://nvie.com/posts/a-successful-git-branching-model/)

[Git工作流指南：集中式工作流](http://blog.jobbole.com/76847/)

[Git工作流指南：Gitflow工作流](http://blog.jobbole.com/76867/)

[gitlab示例](https://gitlab.com/gitlab-org/gitlab-ce/issues)
