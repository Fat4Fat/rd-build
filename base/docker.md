# docker

### 简介

*Docker* 是一个开源的应用容器引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。

### docker 安装

1. mac本地安装

   > 官方安装文档：https://docs.docker.com/docker-for-mac/install/

2. centos安装

   > 官方安装文档：https://docs.docker.com/engine/installation/linux/docker-ce/centos/

   中国镜像安装最新稳定版

   ```Shell
   curl -sSL https://get.daocloud.io/docker | sh
   ```

   键入docker -v命令，若显示如下返回则表示安装成功

   ```shell
   [root@localhost vagrant]# docker -v
   Docker version 17.03.0-ce, build 60ccb22
   ```

   docker服务启动 重启 停止

   ```Shell
   [root@localhost vagrant]# service docker start
   Redirecting to /bin/systemctl start  docker.service

   [root@localhost vagrant]# service docker restart
   Redirecting to /bin/systemctl restart  docker.service

   [root@localhost vagrant]# service docker stop
   Redirecting to /bin/systemctl stop  docker.service
   ```


  在命令行中运行以下命令, 成功返回则代表安装成功

  ```shell
  $ docker --version
  Docker version 17.03.0-ce, build 60ccb22

  $ docker-compose --version
  docker-compose version 1.11.2, build dfed245

  $ docker-machine --version
  docker-machine version 0.10.0, build 76ed2a6
  ```

### docker容器网段设置

#### 分配独立IP

安装pipework

```Shell
wget https://github.com/jpetazzo/pipework/archive/master.zip
unzip master.zip
cp pipework-master/pipework  /usr/bin/
chmod +x /usr/bin/pipework
```

给宿主机创建网桥

```shell
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-eno16777736
TYPE=Ethernet
#BOOTPROTO=static
#IPADDR=172.16.146.116
#NETMASK=255.255.255.0
#GATEWAY=172.16.146.1
#DNS1=8.8.8.8
#DNS2=192.168.252.3
BOOTPROTO="none"
DEFROUTE="yes"
PEERDNS="yes"
PEERROUTES="yes"
NAME=eno16777736
ONBOOT=yes
BRIDGE="br0"
~
```

设置物理网卡桥接到网桥

```shell
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-br0
TYPE="Bridge"
BOOTPROTO=static
IPADDR=172.16.146.116
NETMASK=255.255.255.0
GATEWAY=172.16.146.1
PREFIX=24
DNS1=172.16.2.2
DNS2=114.114.114.114
NAME=br0
ONBOOT=yes
DEVICE=br0
```

重启网络  

```shell
service network restart
```

给容器分配一个独立IP的例子  

```shell
docker run -d -it --net=none --name test01 centos
pipework br0 test01 172.16.2.3/24@172.16.2.1
```

### docker镜像仓库

#### 本地仓库搭建

* 使用本地gitlab仓库（优先）

  参看软件服务[gitlab](/build/gitlab.md)章节

* 搭建本地仓库（不建议）

  ```
  docker run -d -p 5000:5000 --restart=always --name registry \
    -v /data/registry:/var/lib/registry \
    registry:2
  ```

#### 在私有仓库上传、下载、搜索镜像

> https://yeasy.gitbooks.io/docker_practice/content/repository/local_repo.html

commit镜像

当我们运行一个容器的时候（如果不使用卷的话），我们做的任何文件修改都会被记录于容器存储层里。而 Docker 提供了一个 docker commit 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。

`docker commit` 的语法格式为：

    docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

    $ docker commit \
        --author "lvyalin" \
        --message "修改了默认网页" \
        webserver \
        nginx:v2
    sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214

其中 --author 是指定修改的作者，而 --message 则是记录本次修改的内容。这点和 git 版本控制相似，不过这里这些信息可以省略留空。

dockerfile镜像

>https://yeasy.gitbooks.io/docker_practice/content/image/dockerfile/

### 常用命令

#### 容器操作

1. 运行一个新的容器

   ```
   docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
   ```

   - **--name** 为容器指定一个名称
   - **-d** 后台运行容器，并返回容器ID
   - **-i** 以交互模式运行容器，通常与 -t 同时使用
   - **-t** 为容器重新分配一个伪输入终端，通常与 -i 同时使用
   - **-p** 建立本机到容器的端口映射 `80:80`代表将本机80端口映射到容器的80端口上
   - **-v** 建立本机到容器的目录映射

2. 在运行的容器中执行命令

   ```
   docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
   ```

   - **-i** 以交互模式运行，通常与 -t 同时使用
   - **-t** 分配一个伪输入终端，通常与 -i 同时使用
   - **-d** 在后台运行

3. 列出容器

   ```
   docker ps [OPTIONS]
   ```

   - **默认** 列出所有在运行的容器信息
   - **-a** 显示所有的容器，包括未运行的  

4. 启动一个已经被停止的容器

   ```
   docker start CONTAINER [CONTAINER...]
   ```

5. 停止一个运行中的容器

   ```
   docker stop CONTAINER [CONTAINER...]
   ```

6. 删除一个容器

   ```
   docker rm [OPTIONS] CONTAINER [CONTAINER...]
   ```

   * **-f** 强制删除一个运行中的容器

#### 镜像操作

1. 从镜像仓库中拉取或者更新指定镜像

   ```
   docker pull NAME[:TAG|@DIGEST]
   ```

2. 列出本地镜像

   ```
   docker images
   ```

3. 删除本地镜像

   ```
   docker rmi IMAGE [IMAGE...]
   ```

   * **-f** 强制删除