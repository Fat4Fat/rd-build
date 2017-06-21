## 示例

下面的示例演示一次需求迭代的流程。假设你已经创建了一个项目demo,并创建了一个master分支。

### 项目准备
![创建production](/images/branch/创建production分支.png)


首先为master分支创建一个配套的production生产分支。简单来做可以通过gitlab的UI来创建，或者通过本地命令行创建分支并push到服务器上：

```shell
# 当前分支为master
git branch production
git push -u origin production
```

在gitlab中将production分支设置为默认、保护分支。

### 管理员创建第一个阶段的里程碑

在项目创建后，我们需要给项目创建一个大的目标即gitlab的里程碑。里程碑的创建如下图所示：

![创建里程碑](/images/gitlab/1.创建里程碑.png)

### 产品小明提出了一个新的需求

再通过需求评审后，小明将需求以gitlab的issue的方式来体现。创建issue如下图所示：

![创建里程碑](/images/gitlab/2.新建问题.png)
小明在新建问题的时候可以设置问题的里程碑、标签和截止时间，同时可以指定将问题指派给谁。

### 研发人员小黑收到需求后开始开发

小黑接到新的功能需求后，开始创建相关的功能分支进行开发。可以通过gitlab问题页面进行创建，如下图所示：

![需求分支创建](/images/gitlab/3.新建分支.png)

新建的分支名称为 ：2-credit-task 由“问题编号-问题标题”组成。也可以通过git命令行创建分支：
```shell
git checkout -b 2-credit-task master
```
![创建feature分支.png](/images/branch/创建feature分支.png)
所有的需求分支都是基于`master`分支的。创建分支后，他们用老套路添加提交到各自功能分支上：编辑、暂存、提交：
```shell
git status
git add
git commit
```

### 小黑完成功能开发

小黑完成开发后，自己进行自测都OK了。如果团队使用`Pull Requests`，这时候可以发起一个用于合并到`master`分支。如下图所示：

![合并分支](/images/gitlab/4.新建合并请求.png)

否则他可以直接合并到他本地的`master`分支后`push`到中央仓库：

```shell
## 合并分支
git pull origin master
git checkout master
git merge 2-credit-task
git push
## 合并成功并验收完成后删除分支
git branch -d 2-credit-task
```

![img](/images/branch/合并分支.png)

小黑在发起合并请求或者push请求时可以添加 `Closes #2`来关闭相关问题。小黑在将代码push到`master`后就开始触发了集成测试（CI）,如果测试不通过小黑需要在需求分支上修改代码后，继续发起合并请求。当集成测试通过后，自动触发CD操作将代码部署到稳定测试环境上，小黑的当前功能已经开发完成了。

### 版本发布

当大家都开发完成自己的需求后，开始发布准备，首先需要先部署代码。先将代码合并到production分支。
![合并到production分支](/images/branch/合并到production分支.png)
gitlab操作请参考合并分支请求操作，git命令操作如下：
```shell
## 合并到production分支
git pull origin production
git checkout production
git merge master
git push
```
合并完代码后需要手动确认将代码推送到生产代码仓库中，之后自动切换生产代码，完成发布。如下图所示：

![发布操作](/images/gitlab/6.发布操作.png)

### 添加tag
添加tag的操作可以选择版本发布前后皆可。主要在一个里程碑的完成代表一个大的版本迭代，一个里程碑中主要细节点也可以添加小版本的tag。UI操作如下图：

![添加tag](/images/gitlab/5.添加tag.png)

也可以进行命令行操作：

```shell
## 给production分支添加tag
git pull origin production
git checkout production
## 如果是补标签则在后面加上版本号即可，例如：git tag -a v1.0 9fceb02
git tag -a v1.0 -m '版本1.0发布'
## 此处注意git push并不会把标签传送到远端服务器上，只有通过显式命令才能分享标签到远端仓库。
git push origin v1.0
```

