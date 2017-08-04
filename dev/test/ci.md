# 自动化测试

### 简介

自动化测试是CI中的一个环节，由代码合并master时自动触发，主要的目的是为了这次的版本变更不会影响到系统中的其他功能，保证整个系统对外提供的功能的正确性。

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

