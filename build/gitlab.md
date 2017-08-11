# gitlab

### 安装

> 优先使用docker安装 https://docs.gitlab.com/omnibus/docker/

```Shell
# 构建gitlab容器
sudo docker run -itd \
--detach \
--hostname gitlab.nw.com \
--net=none \
--name gitlab \
--restart always \
--volume /data/gitlab/config:/etc/gitlab:Z \
--volume /data/gitlab/logs:/var/log/gitlab:Z \
--volume /data/gitlab/data:/var/opt/gitlab:Z \
gitlab/gitlab-ce:latest

# 分配ip
pipework br0 gitlab 172.16.2.3/24@172.16.2.1
```

### 配置

> 每次修改完配置后需要重启gitlab使配置生效，重启命令为：
>
> ```shell
> docker exec -i gitlab /bin/bash -c "gitlab-ctl reconfigure"
> ```

1. 使用内置的registry服务，[官方参考链接](https://docs.gitlab.com/ce/administration/container_registry.html#configure-container-registry-under-an-existing-gitlab-domain)。

   配置`gitlab.rb`，编辑host上的配置文件`vim /data/gitlab/config/gitlab.rb`，修改配置信息：

   ```shell
   registry_external_url 'https://gitlab.nw.com:4567'
   ```

   添加证书文件到 /data/gitlab/config/ssl/ 

2. 开启ldap

   配置`gitlab.rb`，编辑host上的配置文件`vim /data/gitlab/config/gitlab.rb`，添加配置信息：

   ```shell
   gitlab_rails['ldap_enabled'] = true
   gitlab_rails['ldap_servers'] = YAML.load <<-EOS # remember to close this block with 'EOS' below
   main: # 'main' is the GitLab 'provider ID' of this LDAP server
     label: 'LDAP'
     host: '{$ip}'
     port: '{$port}'
     uid: 'sAMAccountName'
     method: 'plain' # "tls" or "ssl" or "plain"
     base: 'OU=People,DC=auth,DC=company,DC=com'
     bind_dn: 'CN=sAMAccountName@company.com,OU=People,DC=auth,DC=company,DC=com'
     password: 'password'
     user_filter: ''
     active_directory: false
   EOS
   ```

3. 配置邮件通知

   配置`gitlab.rb`，编辑host上的配置文件`vim /data/gitlab/config/gitlab.rb`，添加邮件配置：

   ```ruby
   # 开启邮件配置
   gitlab_rails['gitlab_email_enabled'] = true

   # 为了防止邮件被当做垃圾邮件拦截，使用smtp
   # https://docs.gitlab.com/omnibus/settings/smtp.html#testing-the-smtp-configuration
   gitlab_rails['smtp_enable'] = true
   gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
   gitlab_rails['smtp_port'] = 465
   gitlab_rails['smtp_user_name'] = "gitlab@nw.com"
   gitlab_rails['smtp_password'] = "smtp password"
   gitlab_rails['smtp_authentication'] = "login"
   gitlab_rails['smtp_enable_starttls_auto'] = true
   gitlab_rails['smtp_tls'] = true
   gitlab_rails['gitlab_email_from'] = 'gitlab@nw.com'
   ```

### gitlab-ci-multi-runner

gitlab-ci-multi-runner用于运行CI作业。

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