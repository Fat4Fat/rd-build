# 构建完整研发内部服务
> 一个完整的研发内部服务（200人以下研发适用）

1. 基础设施搭建。

    - 网络：

      - 网络划分
      - 路由设置
      - 防火墙
      - 企业VPN
      - 海外代理
      - 内网DNS

    - 物理机器

      - 磁盘阵列
      - 小集群管理（1-10台）
      - 容器化

    - 企业软件

      > 核心是组织管理，其他服务是围绕组织管理开展

      - 协同
        - 邮箱
        - IM
        - 网盘
        - wiki
      - 组织管理
        - 通讯录
        - 组织管理
        - 员工统一鉴权（密码+二步认证）
      - OA
        - 财务
        - 行政
        - 人事
        - 审批
        - 报表
      - 研发
        - gitlab
        - 镜像仓库

2.  研发中心

    > 为研发测试提供提升开发效率和保证质量的平台
    >
    > 详见文档 https://github.com/ifintech/rdcenter

3. 持续迭代研发周期管理

    > 详见文档 https://github.com/ifintech/rdflow

4. 规章制度和研发规范

    > 详见文档 https://github.com/ifintech/rddocs




## 内网部署实施

> 非云服务资源部署如下
> 不使用云服务，无非两点 1.价格贵 2.核心数据资产，要么就是内部支撑无法使用云服务比如DNS

### 内网入口 staff.nw.com
### 1. 宿主机 172.16.2.2

>配置16G内存，2T硬盘，1U8核
>环境centos7，使用docker提供服务
>常规配置（时间、DNS、网络）

安装docker服务 https://github.com/ifintech/rdbuild/tree/master/docker



### 2. 内网DNS 172.16.2.2

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



### 3. 代码（镜像）仓库 & 持续集成 172.16.2.3 gitlab.nw.com 

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



### 4. 网盘 172.16.2.4 pan.nw.com

> 安装owncloud https://owncloud.org/  

```shell
sudo docker run -itd \
--name owncloud \
--net=none \
--volume /data/owncloud:/var/www/html \
owncloud

pipework br0 owncloud 172.16.2.4/24@172.16.2.1
```



### 5. 公网接入内网办公VPN 172.16.2.5

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
IP="office_ip"
mkdir ${OVPN_DATA}
docker run -v ${OVPN_DATA}:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://${IP}
docker run -v ${OVPN_DATA}:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
docker run --name openvpn -v ${OVPN_DATA}:/etc/openvpn -itd --privileged kylemanna/openvpn
pipework br0 openvpn 172.16.2.5/24@172.16.2.1
```



### 6.企业wiki 172.16.2.6 wiki.nw.com

### 7.镜像仓库 172.16.2.7 images.nw.com



### 8. 海外办公专线

>访问海外资源（邮箱、代码包、产品）
>
>安装shadowsocks https://github.com/shadowsocks  
>
>服务端：  
>
>客户端：局域网网络内配合PAC使用



### 9. 网络7层HTTP转发 172.16.2.10

> 启用dnsmasq泛解析，通过nginx做内部HTTP转发到指定服务器
> 在rdcenter里scp配置nginx文件给指定服务器

```shel
sudo docker run -itd \
--name nginx_proxy \
--net=none \
nginx

pipework br0 nginx_proxy 172.16.2.10/24@172.16.2.1
```

