# 内部服务构建部署

> 非云服务资源部署如下  
> 不使用云服务，无非两点 1.核心数据资产 2.要么就是内部支撑无法使用云服务比如DNS  

### 内网入口 staff.nw.com

### 所有员工的授权认证统一接入集中auth系统



### 6. 企业wiki 172.16.2.6 wiki.nw.com


 

### 8. 内网研发平台 172.16.2.9 rd.nw.com

> 详见文档

### 9. 网络7层HTTP转发 172.16.2.10

> 启用dnsmasq泛解析，通过nginx做内部HTTP转发到指定服务器

```shel
sudo docker run -itd \
--name nginx_proxy \
--net=none \
openresty

pipework br0 nginx_proxy 172.16.2.10/24@172.16.2.1
```
