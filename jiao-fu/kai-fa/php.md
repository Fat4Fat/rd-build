# PHP本地开发

### 简介

#### 本地IDE

采用[phpstrom](http://www.jetbrains.com/phpstorm/)进行编码开发

#### PHP框架

> 项目介绍：[https://github.com/ifintech/phplib](https://github.com/ifintech/phplib)

### 获取项目代码

#### ①新建工程

1. 获取phplib-templete到本地

   ```shell
   git clone https://github.com/ifintech/phplib-template.git
   ```

2. 进入项目代码根路径，执行脚手架脚本

   ```shell
   cd phplib-template
   # 查看脚手架使用说明
   php cg.php
   # 根据参数提示进行构建，参数依次为：项目名称、域名、项目目标地址、模块列表(多个模块用英文逗号分割)
   php cg.php demo demo.com ~/work/demo admin
   ```

   > 参数介绍：
   >
   > * 项目名称：项目的唯一标识
   > * 域名：本地开发时的访问域名
   > * 项目目标地址：本地生成项目代码的地址
   > * 模块列表：添加项目依赖的模块，现支持模块有：admin（后台模块）、wechat（微信模块）

3. 此时`~/work/demo`下已经有了我们项目的初始代码，代码结构如下。

#### ②导入工程

1. 直接通过`git clone`项目代码即可。

   ```shell
   git clone git@gitlab.yilumofang.com:demo/demo.git
   ```

   > 需要注意是否拥有项目代码检出的权限

### 构建本地开发环境

通过上个获取项目代码的流程，我们已经获取到了项目的代码，现在来构建本地的开发环境，本地开发环境基于docker服务编排来构建，在项目文件的根目录下我们可以看到编排文件`docker-compose.yml`和环境文件`.env`。

1. 在项目的根目录下执行构建命令，生成项目环境。

   ```shell
   docker-compose -p demo up -d
   ```
   编排文件`docker-compose.yml`：

   ```yaml
   version: "3"
   services:
     nginx:
       image: nginx
       restart: always
       ports:
         - "80:80"
       volumes:
         - ./build/nginx/vhosts/${PROJECT_NAME}.conf:/etc/nginx/conf.d/${PROJECT_NAME}.conf
         - ./build/nginx/fastcgi_params:/etc/nginx/fastcgi_params
       links:
         - php

     php:
       build: .
       command: php-fpm
       restart: always
       volumes:
         - .:/data1/htdocs/${PROJECT_NAME}
         - /tmp/logs:/data1/logs
       links:
         - mysql
         - redis

     mysql:
       image: mysql:5.7
       restart: always
       command:
         - --character-set-server=utf8mb4
         - --collation-server=utf8mb4_unicode_ci
         - --skip-character-set-client-handshake
       environment:
         MYSQL_ROOT_PASSWORD: ${MYSQL_PASSWORD}
         MYSQL_DATABASE: ${PROJECT_NAME}
       volumes:
         - ./build/sql:/docker-entrypoint-initdb.d

     redis:
       image: redis:alpine
       restart: always
   ```

   > 如果本地需要开启多个项目环境，则需要修改编排文件中的端口映射，例如改为81:80，访问时域名后加上端口即可。

2. 修改本地hosts。

   ```shell
   sudo vim /etc/hosts
   # 添加127.0.0.1  demo.com
   ```

3. 访问后台`http://demo.com/admin/login/index`  见到如下页面则代表开发环境构建成功

   ![WX20170514-210300@2x](http://ww1.sinaimg.cn/large/006tNbRwly1ffm9wija05j31kw0s7h8k.jpg)

### 如何开发

本地代码是挂载到开发环境中的，即直接修改本地的项目文件, 即可在容器中见到变化。

[php代码开发规范](rule/php.md)

