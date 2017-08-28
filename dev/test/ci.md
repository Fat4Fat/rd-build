# 自动测试

### 简介

自动测试是CI中的一个环节，由代码合并master时自动触发，主要的目的是为了这次的版本变更不会影响到系统中的其他功能，保证整个系统对外提供的功能的正确性。详细的CI/CD的介绍请参考[附录二：CI/CD](/dev/cicd.md)。

### GitLab CI

使用 GitLab CI + Docker Compose实现整个整个CI的流程。GitLab是通过项目根路径下的.gitlab-ci.yml文件配置定义CI流程，一般情况下使用标准CI配置文件即可，但在一些有特殊要求的项目下，开发人员需要能够在标准配置文件上修改自己项目的特殊需求。下面是php版本的标准CI配置文件及解释，详细请参考[配置文件手册](https://docs.gitlab.com.cn/ce/ci/yaml/README.html#variables)。

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
  # 线下镜像仓库地址
  DEV_REGISTRY_ADDRESS: gitlab.nw.com
  # 生产镜像仓库地址
  PRO_REGISTRY_ADDRESS: dockerhub.nw.com
  # 镜像仓库名称
  REPOSITORY: ${CI_PROJECT_PATH}

before_script:
  - docker login -u gitlab-ci-token -p ${CI_JOB_TOKEN} ${DEV_REGISTRY_ADDRESS}

# 集成测试
test_job:
  stage: test
  script:
    - docker-compose -p ${PROJECT_INDEX} up -d php
    - sleep 120
    - docker-compose -p ${PROJECT_INDEX} run --rm -w /data1/htdocs/${CI_PROJECT_NAME}/test/ php phpunit
  after_script:
    # 销毁测试中生产的容器和镜像
    - docker-compose -p ${PROJECT_INDEX} down
    - docker rmi ${PROJECT_INDEX}_php
  only:
    - master
  tags:
    - CI

# 推送镜像到线下代码仓库
push_offline:
  stage: push
  script:
    - docker build -t ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA} .
    - docker push ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
    # 打上latest标签，以备构建稳定测试环境使用
    - docker tag ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA} ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}
  after_script:
    # 清理镜像
    - docker rmi ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
  only:
    - master
  tags:
    - CI

# 将镜像发布到生产仓库
push_online:
  stage: push
  script:
    - git archive --format=zip --remote=git@gitlab.nw.com:security/configure.git master -o /tmp/configure.zip
    - unzip -o /tmp/configure.zip -d /tmp/configure/
    - cp -r /tmp/configure/${CI_PROJECT_NAME}/* ./
    - docker build -t ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA} .
    - sh /data/gitlab/bin/push.sh ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
    - docker tag ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA} ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}
    - sh /data/gitlab/bin/push.sh ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}
  after_script:
    - docker rmi ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
  environment:
    name: production
  when: manual
  only:
    - master
  tags:
    - CD

# 更新测试环境的代码
deploy_test:
  stage: deploy
  script:
    - docker-compose -f /data/test/${CI_PROJECT_PATH}/docker-compose.yml stop php
    - docker-compose -f /data/test/${CI_PROJECT_PATH}/docker-compose.yml rm -f php
    - docker-compose -f /data/test/${CI_PROJECT_PATH}/docker-compose.yml up -d php
  only:
    - master
  tags:
    - CI

# 清理ci过程中产生的环境
cleanup_job:
  stage: cleanup
  script:
    # 对于稳定测试环境更新时会产生一些tag为none的镜像，此步骤是为了清理这些虚悬镜像
    - docker images|grep none|awk '{print $3}'|xargs docker rmi -f
  when: always
  only:
    - master
  tags:
    - CI
```

java版本CI文件如下：

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
  # 线下镜像仓库地址
  DEV_REGISTRY_ADDRESS: gitlab.nw.com
  # 生产镜像仓库地址
  PRO_REGISTRY_ADDRESS: dockerhub.nw.com
  # 镜像仓库名称
  REPOSITORY: ${CI_PROJECT_PATH}

before_script:
  - docker login -u gitlab-ci-token -p ${CI_JOB_TOKEN} ${DEV_REGISTRY_ADDRESS}

# 集成测试
test_job:
  stage: test
  script:
    - echo "skip test"
  only:
    - master
  tags:
    - CI

# 推送镜像到线下代码仓库
push_offline:
  stage: push
  script:
    # 编译
    - mvn package -DskipTests && mv target/$CI_PROJECT_NAME-SNAPSHOT.jar bin/$CI_PROJECT_NAME.jar
    - docker build -t ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA} .
    - docker push ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
    # 打上latest标签，以备构建稳定测试环境使用
    - docker tag ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA} ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}
  after_script:
    # 清理镜像
    - docker rmi ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
  only:
    - master
  tags:
    - CI

# 将镜像发布到生产仓库
push_online:
  stage: push
  script:
    # 编译(构建编译环境，进行编译，可以通过编译环境解决不同版本jdk的问题)
    - git archive --format=zip --format zip -o /tmp/${CI_PROJECT_NAME}.zip HEAD
    - unzip -o /tmp/${CI_PROJECT_NAME}.zip -d /tmp/${CI_PROJECT_NAME}/
    - docker run -i --rm --name mvn-jdk7 -v /path/to/local/repo:/path/to/local/repo -v /usr/share/maven/conf/settings.xml:/usr/share/maven/conf/settings.xml -v /tmp/xw-parent:/data/xw-parent -w /data/${CI_PROJECT_NAME} maven:3-jdk-7 mvn package -DskipTests
    - cp -r /tmp/${CI_PROJECT_NAME}/* ./

    # 添加生产配置
    - git archive --format=zip --remote=git@gitlab.nw.com:security/configure.git master -o /tmp/configure.zip
    - unzip -o /tmp/configure.zip -d /tmp/configure/
    - cp -r /tmp/configure/${CI_PROJECT_NAME}/* ./

    - docker build -t ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA} .
    - sh /data/gitlab/bin/push.sh ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
    - docker tag ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA} ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}
    - sh /data/gitlab/bin/push.sh ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}
  after_script:
    - docker rmi ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
  environment:
    name: production
  when: manual
  only:
    - master
  tags:
    - CD

# 更新稳定环境的代码
deploy_test:
  stage: deploy
  script:
    - echo "skip deploy test"
  only:
    - master
  tags:
    - CI

# 清理ci过程中产生的环境
cleanup_job:
  stage: cleanup
  script:
    - echo "skip cleanup"
  when: always
  only:
    - master
  tags:
    - CI
```



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

### 环境构建

1. 采用docker服务编排进行构建，执行完单元测试后即销毁，编排文件与本地环境编排文件一致，构建命令。

   ```shell
   docker-compose -p PROJECT_INDEX up php -d
   ```

   ​

2. 执行单元测试用例：

   ```shell
   docker-compose -p $PROJECT_INDEX run --rm -w /data1/htdocs/$CI_PROJECT_NAME/test/ php phpunit
   ```


### 注意事项

1. 开发人员需要能够严格的按照要求去编写测试用例。
2. 当CI构建失败时，一定要及时的去查找CI的失败原因并修改。