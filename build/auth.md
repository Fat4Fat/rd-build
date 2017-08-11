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

auth镜像地址：https://hub.docker.com/r/ifintech/auth/

ldap镜像地址：https://hub.docker.com/r/ifintech/ldap/

编排文件`compose-stack-auth.yml`：

```yaml
version: "3"
services:
  ldap:
    image: ifintech/ldap2http
    ports:
      - "10389:10389"
    environment:
      HOST: 0.0.0.0
      PORT: 10389
      AUTH_URL: https://auth.com
      AUTH_TOKEN: demo_token
    deploy:
	  replicas: 1
	  resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '.1'
          memory: 100M
      update_config:
        parallelism: 1
        delay: 30s
 	networks:
      - servicenet	
  php:
    image: ifintech/auth
    command: php-fpm
    volumes:
      - /data1/auth/security:/data1/htdocs/auth/conf/security
      - /data1/auth/server:/data1/htdocs/auth/conf/server
    deploy:
	  replicas: 1
	  resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '.1'
          memory: 100M
      update_config:
        parallelism: 1
        delay: 30s
    networks:
      - servicenet
networks:
  servicenet:
    external: true
  
```

启动

```
docker stack deploy auth --compose-file compose-stack-auth.yml
```

### 配置

1. 修改钉钉配置文件`/data1/auth/server/production.conf`

   ```php
   <?php
   return array(
       'cache' => array(
           'redis' => array(
               'common' => array(
                   'host'  => 'redis',
                   'port'  => 6379,
                   'timeout'=> 1,
                   'persistent' => 1,  //默认开启长连接
               )
           ),
       ),
       'dd' => array(
           'corpid'     => '',
           'corpsecret' => '',
           'token' => 'bonjour',
       ),
       'mail' => array(
           'otp' => array(
               "host" => '',
               "port" => '25',
               "user" => '',
               "pwd" =>  '',
               "nick" => 'otp密钥信息',
           ),
       ),
       'mysql' => array(
           'auth' => array(
               'master' => array(
                   'username'  => 'root',
                   'password'  => 'Root1.pwd',
                   'host'      => 'mysql',
                   'port'      => '3306',
                   'dbname'    => 'auth',
                   'pconnect'  => false,
                   'charset'   => 'utf8',
                   'timeout'   => 3,
               ),
               'slave' => array(
                   'username'  => 'root',
                   'password'  => 'Root1.pwd',
                   'host'      => 'mysql',
                   'port'      => '3306',
                   'dbname'    => 'auth',
                   'pconnect'  => false,
                   'charset'   => 'utf8',
                   'timeout'   => 3,
               ),
               'backup' => array(
                   'username'  => 'root',
                   'password'  => 'Root1.pwd',
                   'host'      => 'mysql',
                   'port'      => '3306',
                   'dbname'    => 'auth',
                   'pconnect'  => false,
                   'charset'   => 'utf8',
                   'timeout'   => 3,
               )
           )
       ),
       'redis' => array(
           'common' => array(
               'host'  => 'redis',
               'port'  => 6379,
               'timeout'=> 1,
               'persistent' => 1,
               //'db'    => 1,
           ),
       ),
   );
   ```

   ​

2. 修改ldap配置文件`/data1/auth/security/production.conf`

   ```php
   <?php
   return array(
       'aes' => array(
           'common'  => array(
               'method'    => 'aes128',
               'password'  => '',
               'iv'        => '',
               'options'   =>  0,
           ),
           'db'  => array(
               'method'    => 'aes256',
               'password'  => '',
               'iv'        => '6f6e9a4f4c87dfd4',
               'options'   => OPENSSL_RAW_DATA,
           ),
           'dd'  => array(
               'method'    => 'aes-256-cbc',
               'password'  => '',
               'options'   => OPENSSL_ZERO_PADDING,
           ),
       ),
       'api' => array(
           'ldap'  => array(
               'password'  => 'token',
           ),
       ),
   );
   ```