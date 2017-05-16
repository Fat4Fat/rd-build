# 说明
本镜像基于centos6.8,并增加了如下服务
> * sshd
> * mysql 5.7.17
> * redis 3.2.3
> * jdk-headless 1.8
> * maven

并且默认开机启动

默认root密码为`Root1.pwd`。  
默认mysql密码为`Root1.pwd`。  


## 构建
在当前目录执行
```
docker build -t gitlab.yilumofang.com:4567/dockerhub/java .

docker push gitlab.yilumofang.com:4567/dockerhub/java
```
构建名为java的镜像

### 创建容器
此命令只作为示例
```bash
docker run -itd --name test gitlab.yilumofang.com:4567/dockerhub/java
```