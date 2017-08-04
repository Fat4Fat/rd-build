## 内网稳定环境

内网稳定环境现阶段是通过docker compose构建而成，每次有新镜像push到线下代码仓库后则进行内网稳定环境的更新。

### 环境构建

php项目的`docker-compose.yaml`配置文件内容如下：

```yaml
version: "3"
services:
  nginx:
    image: nginx
    container_name: appname-test-nginx
    restart: always
    ports:
      - "10000:80"
    volumes:
      - ./nginx/wallet.conf:/etc/nginx/conf.d/wallet.conf
      - ./nginx/fastcgi_params:/etc/nginx/fastcgi_params
    links:
      - php

  php:
    image: gitlab.nw.com:4567/path/appname
    container_name: appname-test-php
    command: php-fpm
    restart: always
    links:
      - mysql
      - redis

  mysql:
    image: mysql:5.7
    container_name: appname-test-mysql
    restart: always
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --skip-character-set-client-handshake
    environment:
      MYSQL_ROOT_PASSWORD: Root1.pwd
      MYSQL_DATABASE: appname
    volumes:
      - ./data/mysql:/var/lib/mysql

  redis:
    image: redis:alpine
    container_name: appname-test-redis
    restart: always
```

前端项目的`docker-compose.yaml`配置文件内容如下：

```yaml
version: "3"
services:
  client:
    image: gitlab.nw.com:4567/path/client
    container_name: appname-test-client
    restart: always
    ports:
      - "10001:80"
```



### 镜像更新

稳定环境的更新操作如下(php项目)：

```
docker-compose -f /data/test/${CI_PROJECT_PATH}/docker-compose.yml stop php
docker-compose -f /data/test/${CI_PROJECT_PATH}/docker-compose.yml rm -f php
docker-compose -f /data/test/${CI_PROJECT_PATH}/docker-compose.yml up -d php
```

其他语言的项目操作类似，只需要修改后面的services名称即可。