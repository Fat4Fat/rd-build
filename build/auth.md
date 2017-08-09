## 认证中心

基于钉钉的组织架构，对员工的密码和OTP进行统一的接入方式和安全认证，确保公司员工的认证的安全。

### 功能

- 添加用户
  钉钉回调通知新增用户，获取到用户信息后入库，同时将用户初始化密码、otp密钥信息二维码和google二步认证软件下载二维码通过邮件发送给用户，用户下载并安装软件后扫码生成对应的动态二次码

- 删除用户
  钉钉回调通知用户离职，获取用户id后从认证中心删除相关用户信息

- 更新用户信息
  钉钉回调通知用户信息修改，获取并更新用户最新信息

- 修改密码
  员工通过手机验证码可以修改密码

- 身份认证
  员工认证方式为密码+OTP认证。提供单独OTP认证服务。

- 用户信息查询

  提供用户信息查询的功能。

### 搭建

项目代码地址：https://github.com/ifintech/auth.git

镜像地址：

执行构建命令完成项目搭建：

```shell
docker run -itd \
--net=none \
--name auth \
--restart always \
--volume /data/auth/conf/nginx/auth.conf:/usr/local/openresty/nginx/conf/vhosts/auth.conf \ 
--volume /data/auth/conf/server/dd.php:/data1/htdocs/auth/conf/server/dev/dd.php \ 
--volume /data/auth/conf/security/api.php:/data1/htdocs/auth/conf/security/dev/api.php \ 
ifintech/auth
```

### 配置

1. 修改nginx配置文件`/data/auth/conf/nginx/auth.conf`

   ```
   server {
           listen  80;
           server_name auth.nw.com;
           root /data1/htdocs/auth/public;

           access_log  logs/auth.access.log  main;
           error_log   logs/auth.error.log;

           location / {
               fastcgi_pass   web;
               fastcgi_index  index;
               include        fastcgi_params;
               rewrite ^(.*)$ /index.php$1 break;
           }

           location ~ /admin {
               fastcgi_pass   web;
               fastcgi_index  index;
               include        fastcgi_params;
               rewrite ^(.*)$ /admin.php$1 break;
           }
   }
   ```

   重启nginx：

   ```
   docker exec -i gitlab /bin/bash -c "/usr/local/openresty/nginx/sbin/nginx -s reload"
   ```

   ​

2. 修改钉钉配置文件`/data/auth/conf/server/dd.php`

   ```
   <?php
   return array(
       'corpid'     => '',
       'corpsecret' => '',
       'token' => '',
   );
   ```

   ​

3. 修改ldap配置文件`/data/auth/conf/security/api.php`

   ```
   <?php
   return array(
       'ldap'  => array(
           'password'  => '',
       ),
   );
   ```