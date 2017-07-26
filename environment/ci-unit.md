# 单元测试环境

### 简介

单元测试环境是在[CI](/cd/cicd.md)过程中生成的，采用docker服务编排进行构建。执行完单元测试后即销毁。

### 环境构建

通过编排文件生成编排环境，执行命令`docker-compose -p PROJECT_INDEX up php -d`

```yaml
version: "3"
services:
  nginx:
    image: nginx
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./build/nginx/vhosts/${PROJECT_NAME}.conf:/etc/nginx/conf.d/${PROJECT_NAME}.conf
      - ./build/nginx/fastcgi_params:/etc/nginx/fastcgi_params
    links:
      - php

  php:
    build: .
    command: php-fpm
    restart: always
    volumes:
      - .:/data1/htdocs/${PROJECT_NAME}
      - /tmp/logs:/data1/logs
    links:
      - mysql
      - redis

  mysql:
    image: mysql:5.7
    restart: always
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --skip-character-set-client-handshake
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_DATABASE: ${PROJECT_NAME}
    volumes:
      - ./build/sql:/docker-entrypoint-initdb.d

  redis:
    image: redis:alpine
    restart: always
```

执行单元测试用例：

`docker-compose -p $PROJECT_INDEX run --rm -w /data1/htdocs/$CI_PROJECT_NAME/test/ php phpunit`
