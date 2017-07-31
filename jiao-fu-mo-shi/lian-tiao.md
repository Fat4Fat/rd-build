## 联调

主要介绍前后端人员在自己本地功能开发完成后如何进行联调。

#### 服务端人员

1. 设置联调域名，修改项目代码下的`bulid/nginx/vhosts/app_name.conf`文件，将`server_name`改为联调使用的域名。

   > 一般联调环境的域名以.d进行标识

2. 开启服务

   ```shell
   docker-compose up -d
   ```

#### 前端人员

1. 修改配置文件`config/webpack.compose.js`，添加联调的api代理。

   ```javascript
   const PROXY = {
     // 联调
     '/demo': {
       target:'http://demo.d.ylmf.com',
       changeOrigin: true,
     },
   }
   ```

2. 修改本地host，添加服务端人员机器的IP地址与域名映射。

   ```
   192.168.30.103	demo.d.ylmf.com
   ```

   准备工作结束，可以开始联调。