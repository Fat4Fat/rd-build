## 发布生产

发布生产的操作通过GitLab CI/CD实现，操作流程配置同CI配置一起放到了`.gitlai-ci.yml`文件中：

```yaml
# 将镜像发布到生产仓库
push_online:
  stage: push
  script:
    - docker pull ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
    - docker tag ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA} ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
    - sh /data/gitlab/bin/push.sh ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
    - docker tag ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA} ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}
    - sh /data/gitlab/bin/push.sh ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}
  after_script:
    - docker rmi ${DEV_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
    - docker rmi ${PRO_REGISTRY_ADDRESS}/${REPOSITORY}:${CI_COMMIT_SHA}
  environment:
    name: production
  when: manual
  only:
    - master
  tags:
    - CD
```



通过GitLab UI实现人工手动操作发布上线。在项目下的pipelines目录下能够看到如下的操作页面：

![发布操作](/images/gitlab/6.发布操作.png)

点击push按钮，首先将指定版本的镜像发布到生产，之后触发生产的版本更新。