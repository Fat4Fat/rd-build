# gitlab

> 优先使用docker安装 https://docs.gitlab.com/omnibus/docker/

```Shell
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

pipework br0 gitlab 172.16.2.3/24@172.16.2.1
```

> 使用内置的registry服务
> https://docs.gitlab.com/ce/administration/container_registry.html#configure-container-registry-under-an-existing-gitlab-domain

配置 vim /etc/gitlab/gitlab.rb
```shell
registry_external_url 'https://gitlab.nw.com:4567'
```
添加证书 /data/gitlab/config/ssl/  

> centos安装部署gitlab https://about.gitlab.com/downloads/#centos7

配置 vim /etc/gitlab/gitlab.rb
> 开启ldap

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

重启

```shell
gitlab-ctl reconfigure
```



