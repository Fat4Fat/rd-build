## 联调

主要介绍前后端人员在自己本地功能开发完成后如何进行联调。

#### 服务端人员

1. 设置联调域名，修改项目代码下的`bulid/nginx/vhosts/app_name.conf`文件，将`server_name`改为联调使用的域名。

   > 一般联调环境的域名以.d进行标识

2. 开启服务

   ```shell
   docker-compose up -d
   ```

3. 配置内网HTTP代理

   参考[内网HTTP代理](/base/nginx.md)

#### 前端人员

1. 修改配置文件`config/webpack.compose.js`，添加联调的api代理，域名为服务端人员给出的域名。

   ```javascript
   const PROXY = {
     // 联调
     '/demo': {
       target:'http://demo.d.ylmf.com',
       changeOrigin: true,
     },
   }
   ```
   如果服务端人员进行了内网HTTP代理的配置，则准备工作结束，可以开始联调。

2. 如果服务端人员未配置内网HTTP代理，则需要前端人员修改本地host，添加服务端人员机器的IP地址与域名映射，此时可以开始联调。