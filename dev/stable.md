## 内网稳定环境

内网稳定环境部署在内网swarm集群环境，每次有新镜像push到线下代码仓库后则进行内网稳定环境的更新。

### 环境构建

项目的`compose-stack-demo.yaml`配置文件内容如下：

```yaml
version: "3"
services:
  front:
    image: gitlab.nw.com/demo:latest
    networks:
      - servicenet
    deploy:
      replicas: 2
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
  php:
    image: gitlab.nw.com/demo:latest
    networks:
      - servicenet
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '1'
          memory: 1G
        reservations:
          cpus: '0.1'
          memory: 100M
      update_config:
        parallelism: 1
        delay: 30s
    command: ["php-fpm"]
  nginx:
    image: ifintech/nginx-php
    networks:
      - servicenet
    environment:
      APP_NAME: wallet
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
        reservations:
          cpus: '0.1'
          memory: 50M
      update_config:
        parallelism: 1
        delay: 30s
  mysql:
    image: mysql:5.7
    networks:
      - servicenet
    environment:
      MYSQL_ROOT_PASSWORD: Root1.pwd
      MYSQL_DATABASE: wallet
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
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --skip-character-set-client-handshake
    volumes:
      - ./sql:/docker-entrypoint-initdb.d
      - ./data/mysql:/var/lib/mysql

  redis:
    image: redis:alpine
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
networks:
  servicenet:
    external: true
```

### 镜像更新

每次有新镜像push到线下代码仓库后，触发环境的更新操作，基于docker service update的滚动更新。此步骤有gitlab-ci来设置执行。

```shell
docker service update ${CI_PROJECT_NAME}_php --with-registry-auth
```



