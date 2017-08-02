# 内网DNS

>安装dnsmsq http://www.thekelleys.org.uk/dnsmasq/

```shell
yum install dnsmasq
service dnsmasq start

vim /etc/dnsmasq.conf

listen-address=172.16.2.2,127.0.0.1  
strict-order去掉# 
```
 
解析gitlab.nw.com

```shell
echo `172.16.2.3 gitlab.nw.com` >> /ect/hosts
```
泛域名解析 *.d.nw.com
```shell
echo `address=/d.nw.com/192.168.1.80` >> /ect/dnsmasq.conf
```

每次修改后要重启服务
```shell
service dnsmasq restart
```


添加自定义域名，在路由器里设置172.16.2.2为默认DNS服务器




