## 前端本地开发

## 基础
程序的基础使用，需要 node 环境 v6.9.0 以上，没有的同学，不管你是安装还是起 docker 服务，请自行搞定。

```bash
# 安装
git clone https://github.com/skinTest/webpack-boilerplate.git my-app
cd my-app
npm i

# 开发
npm start

# 构建
npm run build

# 构建并发布到测试服务器
npm run deploy_test
```

---

## 业务组件规范

### 组件层级
本项目将视觉元素分为三个层级，页面 - 容器 - 组件。

* 页面： 和路由对应，是对用户的网站业务呈现。如网站的 about 页面，或是某个表单页面。
* 容器： 包裹基本视觉组件，处理与用户的交互，可以记录部分状态数据。如输入框组成的 panel，菜单显示模板等。
* 组件： 组件本身是无状态的，只执行单纯的显示逻辑。组件显示所需数据由容器提供。如一个输入框，一套 checkbox。

### 组件编写规范
vue spec 请参考 vue [官方文档](https://vue-loader.vuejs.org/en/start/spec.html) 以下是本项目中一些特殊的约定


#### template
模板采用标准 html，标签中如果 attr 值过长，请换行对齐，在 attr 排序过程中先写 class 然后是 html 原生属性；然后写 vue 属性 v-attr（可以理解成一种类 jade || pug 的写法），最后写本组件暴露的属性。另，本项目中请使用 2 空格缩进。

```template
<template>
    <div>
        <label class="at_cell at_check_label" v-for="option in cell.options">
            <div class="at_cell_bd at_cell_primary">
                <p>{{option.label}}</p>
            </div>
            <div class="at_cell_ft">
                <input
                    type="checkbox"
                    class="at_check"
                    v-model="cell.value"
                    :name="cell.name"
                    :value="option.value">
                <i class="at_icon_checked"></i>
            </div>
        </label>
    </div>
</template>
```


#### script
包括三部分

1. 申明依赖
2. 申明该组件的 interface，同时附上默认值
3. 具体的 vue component option，其中顺序如下
  3.1 写 components 的 data 和 props 属性，以便在阅读文件时首先清晰 component 的数据交互
  3.2 写 components 的生命周期函数
  3.3 写 components 的 computed， watch 属性
  3.4 写 components 的 方法

以下代码使用的 vue 1.x 版本，vue 2.0 的 [api 请移步官网](http://vuejs.org/v2/api/)

```js
// 依赖
import _ from 'lodash'
import { cellMethods } from './mixin'

// interface
const cellSchema = {
    name: 'lack_name',
    value: [],
    /** option item shape:
     * {
     *     label: '这是一个默认标签',  type -> String
     *     value: 'default'  type -> String
     * }
     */
    options: [],
}

// vue option
export default {
    props: {
        cell: {
            type: Object,
            default: () => _.assign({}, cellSchema)
        }
    },
    created: function () {
        // type check
        this.cell.value = _.isArray(this.cell.value) ? this.cell.value : []
        this.cell.options = _.isArray(this.cell.options) ? this.cell.options : []

        // default value
        if (_.isObject(this.cell))
            this.cell = _.defaults(this.cell, cellSchema)
    },
    methods: _.assign({}, cellMethods, {
        // component methods goes here
    }),
}
```

style 部分就是正常的 less。 欢迎 scope 爱好者发挥 scope 的好处。另外本项目使用 2 缩进。再次声明。以下列一些具体编码过程中需要参考的代码规范，代码风格暂时不做强制规定，大家自行整改。

- MUST: @mdo -- [link](http://codeguide.co/) -- 主要参考 html 与 样式代码
- SHOULD: @airbnb -- [link](https://github.com/airbnb/javascript) -- 主要参考各代 js/jsx

---

## 依赖管理

### 配置全局依赖库
通过 npm 管理全局库依赖，需要完成三项工作。以引入 zepto（简版 jquery） 为例。

1. 安装：项目内，通过 npm 安装相应的包 `npm i zepto-webpack --save`，zepto-webpack 是为了方便 webpack 打包，对 zepto 进行的包装。
2. 配置：
  2.1 providePlugin 提供提供全局变量，配置 `config/webpack/compose.js` `const GLOBALLIBS` 中 key（全局变量标识） value（全局变量的值）
  2.2 commonChunksPlugin 在打包时将公共代码抽到稳定的包中，配置 `config/webpack/compose.js` `productionConfig.entry.vendor`
  2.3 lint 支持，取消对该全局变量的报警，配置 `eslintrc.js` `globals`
3. 使用，在文件中直接使用 2.1 中配置的标识符即可

_全局库：开发中不需要在文件中单独引入，就可使用的库。_


### 配置 npm 管理的外部依赖
以安装 vue-router 为例

1. 安装： 通过 npm 安装相应的包 `npm i vue-router --save`
2. 配置： 因为 vue-router 在项目开发过程中可以认为将长期保持稳定，所以将 vue-router 打到稳定包(bundle)中，以提高缓存利用效率。 commonChunksPlugin `config/webpack/compose.js` `productionConfig.entry.vendor.push('vue-router')`
3. 使用： 在需要使用 vue-router 库的文件中申明依赖，使用即可
  3.1 支持 commonJS 方式 `var VueRouter = require('vue-router')`
  3.2 也支持 es6 方式 `import VueRouter from 'vue-router'`


### 样式依赖
项目中支持对 less 的编译，同时引入了 postcss 暂时只引入了 autoprefix 特性。项目的外部样式，如 weui 建议拉入源代码， 直接通过项目进行编译。在开发中，将依赖的样式存放在 app/style 中，在 app/style/style.less 的图谱中，最终引入，完成编译。


### 前端组件依赖
建议将依赖的浏览器视觉组件，如 swipe 等，通过 vue 封装成统一的组件，由外部传入配置参数，统一管理。将依赖的的文件选定版本后手工管理，存放在 app/containers/xx-container/ 目录下即可。


### 样式资源加载
在 .less .css .vue/style 等通过 css-loader 编译的额资源，使用 url() 或 @import 引入资源时。如果使用别名，请使用 `~` 模式

```js
// webpack.config
{
  resolve: {
    alias: {
      Assets: path.resolve(__dirname, 'app', 'assets')
    }
  }
}
```

```less
@import '~Assets/images/logo.png'
/* 和以下语句等价 */
@import '../../assets/images/logo.png'
```

关于这个例子，大家自行意淫一下路径关系。关于 [alias 可以参考官方文档](https://webpack.js.org/configuration/resolve/#resolve-alias)

---

## api
分为全局工作，与单个 api 两方面。

* 开发配置
  1. 文档与 mock 服务器维护 -- 这个事儿 @后端同学
  2. 本地服务器代理配置  `webpack.compose.js` `const PROXY` ，[地址配置](https://webpack.js.org/configuration/dev-server/#devserver-proxy)，代理原理是在本地开发服务器（WDS 是个基于 express 框架的服务器），中间加了个[代理中间键](https://github.com/chimurai/http-proxy-middleware)
* TODO 需要和前端资源服务器进行调试，确定在前端服务器中，请求能够正常。
* 全局
  1. 封装 [ajax](http://zeptojs.com/#ajax) 方法
  2. 提供 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) 交互支持，解决嵌套噩梦；
  3. 提供公共的错误处理方法，视觉交互封装；
  4. 提供统一的请求地址管理，如果所有的 api 都在一个地址下面，然后靠参数区分，那就没这个事儿了。
* 单个 api 方面，是对处理工程的封装，包括接口默认参数配置，参数传递方式封装；返回值格式校验；返回数据格式调整；错误分类，错误数据整理。建议将接口按照业务逻辑组织，最终在 api/index.js 中统一输出；或者按业务逻辑分文件组织。


### ajax 封装
Promise 使用了 Babel 提供的 runtime ，无需再单独入。 ajax 方法使用 zepto 库中提供的。需要封装如下伪代码。

```js
export function post (url, data) {
  return new Promise (function (resolve, reject) {
    $.ajax({
      method: 'POST',
      url,
      data,
      success: function (res) {
        resolve(res)
      },
      error: function (err) {
        reject({
          type: 'network',
          msg: '网络开小差了，重连重试吧',
          err,
        })
      },
    })
  })
}
```


### api 编写
丑话说在前面，都是伪代码。

```js
// api/index
export { apiFunc1A, apiFunc1B } from './api-bussiness-1';
export { apiFunc2A, apiFunc2B } from './api-bussiness-2 ';
```

```js
// api/address
export default {
  // bussiness-1
  feature_1: '/api/action_business_1',
  ...
}
```

```js
// api/api-bussiness-1 中
import addr from './address'

export function apiFunc1A (parameters) {
  // 1. 检查参数格式，合法性
  // 2. 配置默认参数
  _.defaults(parameters, { defaults })

  // 3. 进行 ajax 请求
  return new Promise || functionReturnPromise
    // 3.1 判定返回类型，整理返回值
    .then()
    // 3.2 错误处理，整理错误信息，抛出错误
    .catch()
}
```

```js
// 约定的错误抛出格式
{
  api: '出错的 api'
  msg: '给用户或返回报错信息时的描述'
  type: 'network || server || parameter ...'
  err: errorObject,
  handler: '处理函数名称' // optional
}
```

---

## deploy

### 新增发布地址

1. 在 config/deploy.js 中添加静态资源在服务器上的访问路径 `_.set(webpackConfig, 'output.publicPath', '/webpack-boilerplate/')`
2. 配置 gh-pages 发布选项中的远程仓库、分支，这个 @后端大哥， 和他们协调一下。然后配到

```js
var deployConfig = {
  repo: 'https://github.com/skinTest/webpack-boilerplate.git'
}
```

3. 手动跑脚本或者写到 package.json 中，参考 scripts.deploy， 其中的 dest 参数是和发布环境的那几个 case 之一对应的


### 原理
原理是在 webpack 编译完成后，回调函数内用 gh-pages 发送到了一个远程的 git 仓库，可以理解为如下命令，实现上参考 config/deploy.js

```bash
webpack
gh-pages -d dest
```
