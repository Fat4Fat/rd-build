## 认证中心

对员工的密码和OTP进行统一的接入方式和安全认证，确保公司员工的认证的安全。

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

框架地址：https://github.com/ifintech/phplib.git

环境采用：[PHP基础环境](https://hub.docker.com/r/ifintech/php7/)

1. 生成auth容器：

   ````shell
   docker run -itd --name auth ifintech/php7
   ````

2. 部署代码和phplib：

   ```shell
   docker exec -i auth /bin/bash -c "mkdir /data1/htdocs;git clone https://github.com/ifintech/auth.git /data1/htdocs/auth;git clone https://github.com/ifintech/phplib.git /data1/htdocs/phplib"
   ```

3. 初始化环境：

   ```shell
   docker exec -i auth /bin/bash -c "chmod +x /data1/htdocs/auth/build/build.sh;sh /data1/htdocs/auth/build/build.sh"
   ```

### 配置

1. 钉钉配置，配置文件位置`/data1/htdocs/auth/conf/server/dev/dd.php`

   ```php
   <?php
   return array(
       'corpid'     => '',
       'corpsecret' => '',
       'token' => '',
   );
   ```

2. auth认证配置，配置文件位置`/data1/htdocs/auth/conf/security/dev/api.php`

   ```php
   <?php
   return array(
       'ldap'  => array(
           'password'  => '',
       ),
   );
   ```