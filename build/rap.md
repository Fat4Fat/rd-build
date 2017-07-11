## RAP

RAP是一个由阿里提供的可视化接口管理开源工具。通过分析接口结构，动态生成模拟数据，校验真实接口正确性， 围绕接口定义，通过一系列自动化工具提升我们的协作效率。

### 安装RAP

已经将rap环境做成了docker镜像push到了[dockerhub](https://hub.docker.com/r/ifintech/rap/)上，所以优先采用docker安装。

1. 一键拉取

   ```shell
   docker run -itd --name rap ifintech/rap
   ```

2. 执行初始化命令

   ```shell
   docker exec -i rap /bin/bash -c "/root/init.sh"
   ```

3. 分配ip

   ```shell
   pipework br0 rap 172.16.2.6/24@172.16.2.1
   ```

到这步骤已经可以通过ip:8080进行访问了，接下来分配域名并修改tomcat端口。

1. 解析rap.nw.com

   ```shell
   echo `172.16.2.6 rap.nw.com` >> /ect/hosts
   service dnsmasq restart
   ```

2. 修改tomcat端口

   修改tomcat配置文件：`/usr/local/apache-tomcat-8.5.16/conf/server.xml`，将端口有8080改为80，重启tomcat。

   ```shell
   /usr/local/apache-tomcat-8.5.16/bin/shutdown.sh
   /usr/local/apache-tomcat-8.5.16/bin/startup.sh
   ```

现在可以通过http://rap.nw.com来访问我们本地的rap了