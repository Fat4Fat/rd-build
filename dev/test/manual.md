## 独立部署

独立部署是指，单独部署一套指定git版本的环境。

### 环境构建

1. 首先开发人员需要在本地构建自己需要版本的镜像，并上传到gitlab私有镜像仓库中。

> tag为指定版本的hash值

```
docker build -t demo:commit_hash .
docker push
```

2. 之后再研发平台申请环境，填写相应的镜像名称和tag
3. 配置[内网http代理](/base/nginx.md)。