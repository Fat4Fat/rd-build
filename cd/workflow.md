# 基于gitlab的工作流

### 背景介绍

我们已经将代码版本管理有svn切换到git上，并使用gitlab进行管理。

### 版本管理的挑战

虽然git的代码管理非常出色，但是我们面对版本管理的时候，依然有非常大得挑战，我们都知道大家工作在同一个仓库上，那么彼此的代码协作必然带来很多问题和挑战，如下：

1. 如何开始一个Feature的开发，而不影响别的Feature？
2. 由于很容易创建新分支，分支多了如何管理，时间久了，如何知道每个分支是干什么的？
3. 哪些分支已经合并回了主干？
4. 线上代码出Bug了，如何快速修复？而且修复的代码要包含到开发人员的分支以及下一个Release?

随着人员的增加，而且项目周期一长就会出现各种问题。因此，**就像代码需要代码规范一样，代码管理同样需要一个清晰的流程和规范**。在参考了gitflow和gitlabfolw后我们采用了gitlabflow并进行了一些细微的调整。

### 人员划分
1. 产品
2. 研发
3. 项目负责人

### 需求管理

基于issues的需求管理。

创建需求时需要注意以下几点：

1. issue的标题应该描述出最后想要的结果来。
2. issue的标题尽量使用简洁的英文描述为开头，方便开发人员创建分支。
3. 每个issue尽可能的小，可以将多个小issue关联到一个大的issue里，通过子任务的方式体现，同时可以直接连接到相关issue上(使用#进行关联)。

#### 里程碑

附加在每一个问题上的日期分类，可用于确定哪些问题需要在下一个版本解决。 此外，由于每个问题都定有里程碑，每当一个问题解决，它会自动更新进度条。

#### 标签

有不同颜色的类别，用来帮助过滤问题，区分问题的紧急程度和类型。


### 开发流程

![workflow](/images/workflow.png)

#### 分支介绍

- Production 分支

生产分支，这个分支最近发布到生产环境的代码， 这个分支只能从其他分支合并，不能在这个分支直接修改

- Master 分支

这个分支是我们是我们的主开发分支，是所有分支分源头，这个主要合并与其他分支，比如Feature分支和production分支

- Feature 分支系列

这个系列分支主要是基于issue创建的用来开发一个新的功能，一旦开发完成，我们合并回Master。

- Hotfix分支

当我们在Production发现紧急Bug时候，我们需要创建一个Hotfix, 完成Hotfix后，我们合并回Master分支。

#### 工作方式



1. 发现bug或提出新需求时需要到issues列表中提交issue.填写issue内容;选择issue级别;指定响应的开发人员.

2. 开发人员收到issue后,创建开发分支.开发分支的命名规则为:feature - issue编号 - 分支名称,例如“feature-7-login-find-pwd”.

3. 开发人员在完成分支自测后发起到master的merge request,审核人员review代码通过后，触发CI集成测试，并反馈测试结果。 

   > 1. 从commit的message中或者是merge request的信息中可以关联相关的issue,具体语法是:“fixes #7”或者“closes #7”.这样会在issue中创建一个评论，并且merge request会显示相关联的issue.如果这个merge request被接受，这个issue会自动被关闭.
   > 2. 假如你仅仅想关联这个问题，但是不关闭这个问题，你可以这样写：#7

   当master上的代码通过CI后，会触发master上的CD，自动将代码推送到稳定测试环境中。

   ![合并流程](/images/合并流程.png)

4. 当准备发布上线的时候，首先将代码merge到production分支上触发CD，将merge后的production代码自动推送到生产代码仓库中，之后可通过人工发布的操作，完成代码的生产发布。

   ![部署](/images/部署.png)


#### gitlab使用注意事项

##### 代码什么时候提交到主分支

强调频繁地（一天多次）将代码集成到主干的目的是什么：

1. 快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易。
2. 防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。

简单一点来说就是功能开发完成，就可以提交到主分支了。但是功能开发完成的标志点是，测试通过、review 完成。

##### 使用rebase来梳理版本历史记录

在merge之前使用rebase命令来梳理版本记录

##### 使用tag标记版本

当一个里程碑完成后或者一个里程碑中一个重要的功能点完成后，最好使用tag标记project分支当前的版本。

### 相关参考

[gitlabfolw官方文档](https://docs.gitlab.com.cn/ce/workflow/gitlab_flow.html#introduction)

[gitflow提出者的原文](http://nvie.com/posts/a-successful-git-branching-model/)

[Git工作流指南：集中式工作流](http://blog.jobbole.com/76847/)

[Git工作流指南：Gitflow工作流](http://blog.jobbole.com/76867/)

[gitlab示例](https://gitlab.com/gitlab-org/gitlab-ce/issues)