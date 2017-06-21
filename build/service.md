# 内部服务构建部署

> 非云服务资源部署如下  
> 不使用云服务，无非两点 1.核心数据资产 2.要么就是内部支撑无法使用云服务比如DNS  

## 内网入口 staff.nw.com








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

### 6. 企业wiki 172.16.2.6 wiki.nw.com

### 7. 海外办公专线

> 访问海外资源（邮箱、代码包、产品）
>
> 服务端: 安装shadowsocks https://github.com/shadowsocks  
>
> 客户端：
> - 局域网网络内配合PAC使用
> - mac客户端 https://github.com/shadowsocks/ShadowsocksX-NG
 

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
