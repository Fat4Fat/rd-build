## 测试

整个测试环节，大概分为三个步骤：

* 人工测试：确保开发出来的功能满足需求的要求。
* 自动化测试：确保整个系统对外提供的功能没有问题。

其中人工测试在开发分支上进行，自动化测试在合并master时通过CI自动完成。

### CI介绍

如果说workflow是串起整个交付流程的一条线的话，那么CI/CD便是这条线上最重要的一点，可以说没有CI/CD就没有持续交付。CI/CD就是持续交付周期中的部署流水线。

![cicd_pipeline_infograph](/images/cicd_pipeline_infograph.png)

持续集成(Continuous integration，简称CI)CI是软件（产品）研发生命周期中对代码质量、系统集成的一个持续构进的过程，当作为一个团队开发产品时，每个人都要开发自己的功能模块，最终都需要集成在一起，代码也需要集中托管到同一个地方，通过使用一些自动化的代码打包、测试工具，能够在开发人员每提交一次代码的时候，系统自动对程序进行打包和单元测试，如果出现问题，及时通过邮件等方式通知相关的开发人员。**持续集成的目的，就是让产品可以快速迭代，同时还能保持高质量**

> Continuous Integration (CI) is a development practice that requires developers to integrate code into a shared repository several times a day. Each check-in is then verified by an automated build, allowing teams to detect problems early.
>
> By integrating regularly, you can detect errors quickly, and locate them more easily.

> 持续集成是指软件个人研发的部分向软件整体部分交付，频繁进行集成以便更快地发现其中的错误。“持续集成”源自于极限编程（XP），是 XP 最初的 12 种实践之一。

### 为什么要做CI/CD

持续集成与持续部署的最大好处在于**提高交付速度、降低风险**。能够在代码提交到资源库之前，进行构建、自动化测试和项目部署等等，我们每天需要提交上线大量的迭代需求，持续集成可以有效的帮助我们发现代码中的 Bug，并且减少一些反复的工作等等，使团队更加有效的开发协作，持续部署可以降低我们在代码发布过程中的风险，是每次代码发布变得简洁、方便。

#### 持续集成的起因

随着需求的变化越来越快、系统复杂度越来越高，软件集成的工作一般会比较频繁琐碎，为了不影响开发效率，以前软件集成这个环节一般不会经常进行或者只会等到项目后期再进行。但是有些问题，如果等到后期才发现，解决问题的代价很大，有可能导致项目延期或者失败。因此，为了尽早发现软件集成错误，鼓励团队成员应该经常集成他们的工作，这就是所说的持续集成。所以说，持续集成是一种软件开发实践。

#### 持续集成的优点

- “快速失败”，在对产品没有风险的情况下进行测试，并快速响应；
- 最大限度地减少风险，降低修复错误代码的成本；
- 将重复性的手工流程自动化，让工程师更加专注于代码；
- 保持频繁部署，快速生成可部署的软件；
- 提高项目的能见度，方便团队成员了解项目的进度和成熟度；
- 增强开发人员对软件产品的信心，帮助建立更好的工程师文化。


### 如何实现CI/CD

#### 持续集成的实现

要实现持续集成我们首先要做到什么：

1. 使用版本控制工具来管理代码。
2. 实现构建自动化。
3. 实现测试自动化并不断提高测试覆盖率。
4. 每人每天都向主线提交代码，通过频繁的提交代码来降低团队开发可能导致的代码冲突。
5. 每次提交都应在集成机上进行构建测试。
6. 快速构建，持续集成的关键在于快速反馈，需要长时间构建的CI是极其糟糕的。构建的时间尽量满足10分钟原则即将构建的时间控制在10分钟之内。

使用 GitLab CI + Docker进行持续集成：

#### Pipeline

一次 Pipeline 其实相当于一次构建任务，里面可以包含多个流程，如安装依赖、运行测试、编译、部署测试服务器、部署生产服务器等流程。

任何提交或者 Merge Request 的合并都可以触发 Pipeline，如下图所示：

```
+------------------+           +----------------+
|                  |  trigger  |                |
|   Commit / MR    +---------->+    Pipeline    |
|                  |           |                |
+------------------+           +----------------+
```

#### Stages

Stages 表示构建阶段，说白了就是上面提到的流程。我们可以在一次 Pipeline 中定义多个 Stages，这些 Stages 会有以下特点：

- 所有 Stages 会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始
- 只有当所有 Stages 完成后，该构建任务 (Pipeline) 才会成功如果任何一个 Stage 失败，那么后面的 Stages 不会执行，该构建任务 (Pipeline) 失败

因此，Stages 和 Pipeline 的关系就是：

```
+--------------------------------------------------------+
|                                                        |
|  Pipeline                                              |
|                                                        |
|  +-----------+     +------------+      +------------+  |
|  |  Stage 1  |---->|   Stage 2  |----->|   Stage 3  |  |
|  +-----------+     +------------+      +------------+  |
|                                                        |
+--------------------------------------------------------+
```

#### Jobs

Jobs 表示构建工作，表示某个 Stage 里面执行的工作。我们可以在 Stages 里面定义多个 Jobs，这些 Jobs 会有以下特点：

- 相同 Stage 中的 Jobs 会并行执行
- 相同 Stage 中的 Jobs 都执行成功时，该 Stage 才会成功
- 如果任何一个 Job 失败，那么该 Stage 失败，即该构建任务 (Pipeline) 失败

所以，Jobs 和 Stage 的关系图就是：

```
+------------------------------------------+
|                                          |
|  Stage 1                                 |
|                                          |
|  +---------+  +---------+  +---------+   |
|  |  Job 1  |  |  Job 2  |  |  Job 3  |   |
|  +---------+  +---------+  +---------+   |
|                                          |
+------------------------------------------+
```

### CI/CD操作手册

#### 管理员手册

##### 1.安装gitlab-ci-multi-runner

gitlab-ci-multi-runner用于运行作业。

```shell
## 添加yum源：
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
## 安装：
yum install gitlab-ci-multi-runner
## 注册runner(使用共享的token)执行器采用shell：
gitlab-runner register
## 安装服务：
gitlab-runner install
## 启动服务：
gitlab-runner start
```

> 设置runner并行执行数量，修改配置文件: /etc/gitlab-runner/config.toml
>
> ```
> concurrent = 4
> ```
>
> 修改gitlab-runner的执行用户: gitlab-runner install --user "vagrant"
>
> 删除runner
>
> gitlab-runner unregister --url  --token

##### 2.配置邮件通知

修改`/etc/gitlab/gitlab.rb`文件添加邮件配置：

```ruby
# 开启邮件配置
gitlab_rails['gitlab_email_enabled'] = true

# 为了防止邮件被当做垃圾邮件拦截，使用smtp
# https://docs.gitlab.com/omnibus/settings/smtp.html#testing-the-smtp-configuration
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "gitlab@jrmf360.com"
gitlab_rails['smtp_password'] = "smtp password"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
gitlab_rails['gitlab_email_from'] = 'gitlab@jrmf360.com'
```

加载配置，重启服务:

```shell
# 停止服务
gitlab-ctl stop
# 重新加载配置
gitlab-ctl reconfigure
# 开启服务
gitlab-ctl start
```

检查是否成功,

```shell
# 进入cosole
gitlab-rails console
# 执行邮件发送命令
irb(main):001:0>Notify.test_email('destination_email@address.com', 'Message Subject', 'Message Body').deliver_now
```

如果出现错误可以通过`/var/log/maillog`邮件日志查看。

#### 项目人员手册

##### 1.添加dockerfile

dockerfile用于填补项目与基本镜像的差异点，实现环境的定制化。

```dockerfile
############################################################
# 创建contract的dockerfile
# Based on php
############################################################
# Set the base image to centos6.8
FROM gitlab.yilumofang.com:4567/dockerhub/php/image

# File Author / Maintainer
MAINTAINER lvyalin lvyalin.yl@gmail.com

# 安装wkhtmltopdf依赖
RUN yum install -y zlib fontconfig freetype libX11 libXext libXrender

# 安装wkhtmltopdf
RUN wget https://downloads.wkhtmltopdf.org/0.12/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz && \
tar -vxf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz && \
mv wkhtmltox /usr/local/ && \
chmod +x /usr/local/wkhtmltox/bin/wkhtmltopdf && \
cp /usr/local/wkhtmltox/bin/* /usr/bin/ && \
rm -rf wkhtmltox-0.12.4_linux-generic-amd64.tar.xz

# 添加中文字体
ADD ./build/font /usr/share/fonts/

CMD ["/etc/rc.local"]
```

##### 2.配置CI作业

.gitlab-ci.yml文件配置主要是定义CI流程。[配置文件手册](https://docs.gitlab.com.cn/ce/ci/yaml/README.html#variables)

```yaml
# 定义 stages
stages:
  - test
  - push
  - deploy
  - cleanup
  
variables:
  # 项目唯一标识
  PROJECT_INDEX: ${CI_PROJECT_NAME}${CI_COMMIT_SHA}
  DEV_IMAGE_NAME: gitlab.yilumofang.com:4567/${CI_PROJECT_PATH}/image

# 单元测试
test_job:
  stage: test
  script:
    - docker-compose -p ${PROJECT_INDEX} up php -d
    - sleep 120
    - docker-compose -p ${PROJECT_INDEX} run --rm -w /data1/htdocs/${CI_PROJECT_NAME}/test/ php phpunit
  only:
    - master
  tags:
    - CI

# 推送镜像到线下代码仓库
push_image:
  stage: push
  script:
    - docker build -t ${DEV_IMAGE_NAME}:${CI_COMMIT_SHA} .
    - sh /data/gitlab/bin/push.sh ${DEV_IMAGE_NAME}:${CI_COMMIT_SHA} dev
  only:
    - master
  tags:
    - CI

# 清理ci过程中产生的环境
cleanup_job:
  stage: cleanup
  script:
    - docker-compose -p ${PROJECT_INDEX} down
    - docker rmi ${PROJECT_INDEX}_php
  when: always
  only:
    - master
  tags:
    - CI
```

### 相关文章

#### 精华文章

1. [重温大师经典：Martin Fowler 的持续集成](http://www.cnblogs.com/davenkin/archive/2012/02/25/continuous-integration-from-martin-fowler.html)
2. [持续集成是什么？](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)

#### 参考资料

1. [配置 gitlab-ci 进行持续集成](https://ruby-china.org/topics/28726)
2. [基于gitlab搭建CI环境](https://www.zybuluo.com/bornkiller/note/314902)
3. [使用 GitLab-CI 来自动创建 Docker 镜像](http://answ.me/post/build-docker-images-automatically-via-gitlab-ci/)

#### 官方文档

1. [gitlab-ci-multi-runner官方地址](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner)
2. [gitlab-ci手册](https://docs.gitlab.com.cn/ce/ci)