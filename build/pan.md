# 网盘 

使用开源网盘owncloud，如果有的数据可以信任服务商，建议使用钉钉网盘。如果不能信任服务商，搭建开源网盘

### 安装

> 安装owncloud https://owncloud.org/

```shell
sudo docker run -itd \
--name owncloud \
--net=none \
--volume /data/owncloud:/var/www/html \
owncloud

pipework br0 owncloud 172.16.2.4/24@172.16.2.1
```