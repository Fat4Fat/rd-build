## 内网http代理

需要在公司内部的研发平台配置自己的代理信息。

1. 登录研发平台

   > 研发平台地址：http://rdcenter.xxxx.com
   >
   > 登录方式：密码+OTP认证

2. 添加自己要代理的IP、端口和域名

   进入菜单`代理管理->添加代理`添加自己的代理信息

   ![添加代理](/images/rd/rd-添加代理.png)

   > 域名建议使用`.d`的域名

3. 删除自己的代理信息

   ![添加代理](/images/rd/rd-删除代理.png)

### orange搭建

通过服务编排搭建orange服务，通过orange的openapi提供内网http代理。

> orange镜像地址：https://github.com/ifintech/dockerhub-base/tree/master/orange

编排文件如下：

```yaml
version: "3"
services:
  orange_mysql:
    image: mysql:5.7
    networks:
      - servicenet
    deploy:
      replicas: 1
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
        reservations:
          cpus: '0.1'
          memory: 100M
      update_config:
        parallelism: 1
        delay: 30s
    environment:
      - MYSQL_DATABASE=orange
      - MYSQL_USER=orange
      - MYSQL_PASSWORD=12345678
      - MYSQL_RANDOM_ROOT_PASSWORD=true
    volumes:
      - ./sql:/docker-entrypoint-initdb.d
  orange:
    image: ifintech/orange
    networks:
      - servicenet
    deploy:
      replicas: 1
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
        reservations:
          cpus: '0.1'
          memory: 100M
      update_config:
        parallelism: 1
        delay: 30s
    ports:
      - "7777:7777"
      - "80:9999"
    environment:
      - DATABASE_HOST=orange_mysql
      - DATABASE_PORT=3306
      - DATABASE_USER=orange
      - DATABASE_PWD=12345678
      - DATABASE_NAME=orange
    links:
      - orange_mysql
networks:
  servicenet:
    external: true
```

启动命令：

```shell
docker stack deploy orange -c compose-stack-orange.yml
```

