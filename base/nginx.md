## 内网http代理

通过orange配置内网代理信息

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
      - "80:80"
      - "7777:7777"
      - "9999:9999"
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

### 代理配置

登录orange后台`orange.dev.com:9999`，启用`代理&分流`插件，配置代理信息：

![orange](/images/orange.png)

相关orange使用文档：http://orange.sumory.com/docs/