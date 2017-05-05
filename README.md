# 构建完整研发内部服务


### 内网入口 staff.nw.com
### 宿主机 172.16.2.2

>建议配置16G内存，2T硬盘，1U8核，开发团队人数50人以下
>环境centos7，使用docker提供服务
>常规配置（时间、DNS、网络）

### 内网DNS （宿主机）172.16.2.2
>安装dnsmsq http://www.thekelleys.org.uk/dnsmasq/

```shell
yum install dnsmasq
vim /etc/dnsmasq.conf
service dnsmasq start
```
添加自定义域名，在路由器里设置默认DNS服务器
listen-address=172.16.2.2,127.0.0.1  
strict-order去掉#  
vim /ect/hosts

```shell
172.16.2.3 gitlab.nw.com
```

### 代码（镜像）仓库 & 持续集成 172.16.2.3 gitlab.nw.com 

>优先使用docker安装 https://docs.gitlab.com/omnibus/docker/

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

>centos安装部署gitlab https://about.gitlab.com/downloads/#centos7

配置 vim /etc/gitlab/gitlab.rb

```shell
gitlab_rails['ldap_enabled'] = true
gitlab_rails['ldap_servers'] = YAML.load <<-EOS # remember to close this block with 'EOS' below
main: # 'main' is the GitLab 'provider ID' of this LDAP server
  label: 'LDAP'
  host: '42.159.123.20'
  port: 10389
  uid: 'sAMAccountName'
  method: 'plain' # "tls" or "ssl" or "plain"
  base: 'OU=People,DC=auth,DC=jrmf360,DC=com'
  bind_dn: 'CN=sAMAccountName@jrmf360.com,OU=People,DC=auth,DC=jrmf360,DC=com'
  password: 'password'
  user_filter: ''
  active_directory: false
EOS
```

重启

```shell
gitlab-ctl reconfigure
```



### 网盘 172.16.2.4 pan.nw.com

> 安装owncloud https://owncloud.org/  

```shell
sudo docker run -itd \
--name owncloud \
--net=none \
--volume /data/owncloud:/var/www/html \
owncloud

pipework br0 owncloud 172.16.2.4/24@172.16.2.1
```



### 公网接入内网办公VPN 172.16.2.5

> 安装openvpn
>
> 客户端： mac tunnelblick
>
> 服务端：https://github.com/kylemanna/docker-openvpn

安装

```shell
#!/bin/bash
docker pull kylemanna/openvpn
OVPN_DATA="/data/ovpn-data"
IP="124.65.192.206"
mkdir ${OVPN_DATA}
docker run -v ${OVPN_DATA}:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://${IP}
docker run -v ${OVPN_DATA}:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
docker run --name openvpn -v ${OVPN_DATA}:/etc/openvpn -itd --privileged kylemanna/openvpn
pipework br0 openvpn 172.16.2.5/24@172.16.2.1
```



### 研发中心 172.16.2.6 rd.nw.com

> 安装研发中心 rdcenter
> 研发中心下属网段172.16.2.100-172.16.2.200

```shell
sudo docker run -itd \
--name rdcenter \
--net=none \
rdcenter

pipework br0 rdcenter 172.16.2.5/24@172.16.2.1
```



### 企业wiki 172.16.2.7 wiki.nw.com



### 海外办公专线

>访问海外资源（邮箱、代码包、产品）
>
>安装shadowsocks https://github.com/shadowsocks  
>
>服务端：  
>
>客户端：局域网网络内配合PAC使用



### 网络7层HTTP转发 172.16.2.10

> 启用dnsmasq泛解析，通过nginx做内部HTTP转发到指定服务器
> 在rdcenter里scp配置nginx文件给指定服务器

```shel
sudo docker run -itd \
--name nginx_proxy \
--net=none \
nginx_proxy

pipework br0 nginx_proxy 172.16.2.6/24@172.16.2.1
```

