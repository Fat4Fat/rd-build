# 办公VPN

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
```