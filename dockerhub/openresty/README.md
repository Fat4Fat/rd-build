# 说明
本镜像是在cento6.8的基础上增加了sshd服务,默认root密码为`Root1.pwd`
## 构建
在当前目录执行
```
docker build -t centos6.8 .
```
构建名为centos6.8的镜像
### 创建容器
此命令只作为示例
```bash
docker run -itd --name test centos
```