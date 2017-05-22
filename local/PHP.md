# 搭建PHP本地开发环境

## 本地IDE 

采用[phpstrom](http://www.jetbrains.com/phpstorm/)进行编码开发

## PHP框架

> 项目介绍：https://github.com/ifintech/phplib

## PHP镜像

> 项目介绍：https://github.com/ifintech/dockerhub-php

## 新建工程

> 本地phplib路径 {phplib}
> 本地项目路径 {project}
> 项目名称{appname}


1. 拉取公司的php镜像

   ```shell
   docker pull gitlab.yilumofang.com:4567/dockerhub/php
   ```

2. 运行容器

   ```shell
   docker run -itd --name test -p 80:80 -v {phplib}:/data1/htdocs/phplib -v {project}:/data1/htdocs/[项目名] gitlab.yilumofang.com:4567/dockerhub/php
   ```

3. 登录到容器中 执行项目构建脚本

   ```shell
   docker exec -it test /bin/bash
   /usr/local/php/bin/php /data1/htdocs/phplib/Build/cg.php {appname} {appname}.com admin,wechat
   ```
   > cg 说明
   ```text
    *******************************************************************************************
    Code Generetor Version 3.0.0
    *******************************************************************************************
    使用说明:
    1. 执行当前脚本需要root权限
    2. 执行命令示例
       /usr/bin/php /phplib/Build/cg.php 项目名称 域名    模块列表(多个模块用英文逗号分割)
       /usr/bin/php /phplib/Build/cg.php demo    demo.com admin,wechat
       
    3. 生成的项目代码目录位置 - 当前目录
       ./{项目名称}
    *******************************************************************************************
    ```

4. 访问后台`http://localhost/admin/login/index`  见到如下页面则代表开发环境构建成功![WX20170514-210300@2x](http://ww1.sinaimg.cn/large/006tNbRwly1ffm9wija05j31kw0s7h8k.jpg)

5. 如何开发

   直接修改本地的项目文件, 即可在容器中见到变化
   
   
## 导入工程

> 本地phplib路径 {phplib}
> 本地项目路径 {project}
> 项目名称{appname}

1. 拉取公司的php镜像

   ```shell
   docker pull gitlab.yilumofang.com:4567/dockerhub/php
   ```

2. 运行容器

   ```shell
   docker run -itd --name test -p 80:80 -v {phplib}:/data1/htdocs/phplib -v {project}:/data1/htdocs/[项目名] gitlab.yilumofang.com:4567/dockerhub/php
   ```

3. 登录到容器中 执行项目构建脚本

```shell
    docker exec -it test /bin/bash
    sh /data1/hodocs/{appname}/build/bulid.sh
```
   
4. 登录到容器中 启动服务

```shell
    docker exec -it test /bin/bash sh /data1/hodocs/{appname}/bin/service.sh reload
```

5. 访问后台`http://localhost/admin/login/index`  见到如下页面则代表开发环境构建成功![WX20170514-210300@2x](http://ww1.sinaimg.cn/large/006tNbRwly1ffm9wija05j31kw0s7h8k.jpg)

6. 如何开发

   直接修改本地的项目文件, 即可在容器中见到变化