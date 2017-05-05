# 说明
本镜像基于centos6.8,并增加了如下服务
> * sshd
> * mysql 5.7.17
> * redis 3.2.3
> * jdk-headless 1.8
## 构建
在当前目录执行
```
docker build -t java .
```
构建名为centos6.8的镜像
### 创建容器
此命令只作为示例
```bash
docker run -itd --name test java
```