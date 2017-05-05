# 说明
本镜像基于centos6.8,并增加了如下服务
> * sshd
> * openresty 1.11.2.2
> * php 7.1.2
> * mysql 5.7.17
> * redis 3.2.3

并且默认开机启动

默认root密码为`Root1.pwd`。

mysql默认密码存放在`/root/mysql_temporary_password`,登陆后可通过`SET PASSWORD = PASSWORD('Root1.pwd');`修改
## 构建
在当前目录执行
```
docker build -t php .
```
构建名为centos的镜像
### 创建容器
此命令只作为示例,实际以rdcenter后台创建为准和实际环境为准
```bash
docker create -it --name centos_test --net=none -v /data1/phplib:/data1/phplib:ro php
```
启动并分配网络
```bash
docker start php
pipework br0 php 172.17.0.61/24@172.17.0.1
```