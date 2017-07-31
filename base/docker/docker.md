# Docker

## 简介

*Docker* 是一个开源的应用容器引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。

## 安装

- 依赖

  - 下载并安装 `VirtualBox`

- 下载

  - 地址 `https://download.docker.com/mac/stable/Docker.dmg`

- 安装

  在命令行中运行以下命令, 成功返回则代表安装成功

  ```shell
  $ docker --version
  Docker version 17.03.0-ce, build 60ccb22

  $ docker-compose --version
  docker-compose version 1.11.2, build dfed245

  $ docker-machine --version
  docker-machine version 0.10.0, build 76ed2a6
  ```

## 测试

- 运行一个自带的测试容器(web服务)

  ```shell
  docker run -d -p 80:80 --name webserver nginx
  ```

- 访问`http://localhost` 见到如下页面则代表容器运行成功

  ![](https://docs.docker.com/docker-for-mac/images/hello-world-nginx.png)

- 删除容器

  ```shell
  docker ps #列出运行中的容器
  docker rm -f webserver #删除容器
  ```

## 常用命令

### 容器操作

####  运行一个新的容器

 ```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
 ```

- **--name** 为容器指定一个名称
- **-d** 后台运行容器，并返回容器ID
- **-i** 以交互模式运行容器，通常与 -t 同时使用
- **-t** 为容器重新分配一个伪输入终端，通常与 -i 同时使用
- **-p** 建立本机到容器的端口映射 `80:80`代表将本机80端口映射到容器的80端口上
- **-v** 建立本机到容器的目录映射

#### 在运行的容器中执行命令

 ```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
 ```

- **-i** 以交互模式运行，通常与 -t 同时使用
- **-t** 分配一个伪输入终端，通常与 -i 同时使用
- **-d** 在后台运行

#### 列出容器

```
docker ps [OPTIONS]
```

- **默认** 列出所有在运行的容器信息
- **-a** 显示所有的容器，包括未运行的  

#### 启动一个已经被停止的容器

```
docker start CONTAINER [CONTAINER...]
```

#### 停止一个运行中的容器

```
docker stop CONTAINER [CONTAINER...]
```

#### 删除一个容器

```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

- **-f** 强制删除一个运行中的容器

### 镜像操作

#### 从镜像仓库中拉取或者更新指定镜像

```
docker pull NAME[:TAG|@DIGEST]
```

#### 列出本地镜像

```
docker images
```

#### 删除本地镜像

 ```
docker rmi IMAGE [IMAGE...]
 ```

- **-f** 强制删除

## 参考资料

- [官方安装文档](https://docs.docker.com/docker-for-mac)
- [Docker 命令大全](http://www.runoob.com/docker/docker-command-manual.html)
