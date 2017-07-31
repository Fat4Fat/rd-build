# 单元测试

### 简介

单元测试是CI中的一个环节，由代码合并master时自动触发，主要的目的是为了解决单个项目代码质量的问题。

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

