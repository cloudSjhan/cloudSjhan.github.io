---
title: 记第一个Vue项目台前幕后的经历
tags: [Vue,Frontend]
copyright: true
date: 2019-01-28 15:28:46
permalink:
categories: Frontend
description: 记第一个Vue项目台前幕后的经历
image: https://static001.geekbang.org/resource/image/bb/71/bbc790cb597fbdb617d1857682d7b071.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

0前端开发经验，初次接触Vue，从后端到前端，从开发、打包到部署，完整的历程。

首先粗略通读了一遍[官方文档](https://cn.vuejs.org/)，动手用webpack搭建了一个简单的demo。

看了Echarts的官方[demo](https://echarts.baidu.com/examples/)，了解了几种数据图表的数据结构。因为我要做的项目就是要将后端接口的数据拿到，然后图形化的形式展示出来。

# 对接后端，进行axios二次开发

在构建应用时需要访问一个 API 并展示其数据，调研Vue的多种方式后选择了官方推荐的axiox。

# 从ajax到fetch、axios

前端是个发展迅速的领域，前端请求自然也发展迅速，从原生的XHR到jquery ajax，再到现在的axios和fetch。

### jquery ajax

```
$.ajax({
    type: 'POST',
    url: url,
    data: data,
    dataType: dataType,
    success: function() {},
    error: function() {}
})
复制代码
```

它是对原生XHR的封装，还支持JSONP，非常方便；真的是用过的都说好。但是随着react，vue等前端框架的兴起，jquery早已不复当年之勇。很多情况下我们只需要使用ajax，但是却需要引入整个jquery，这非常的不合理，于是便有了fetch的解决方案。

### fetch

fetch号称是ajax的替代品，它的API是基于Promise设计的，旧版本的浏览器不支持Promise，需要使用polyfill es6-promise

举个例子：

```
// 原生XHR
var xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4 && xhr.status === 200) {
        console.log(xhr.responseText)   // 从服务器获取数据
    }
}
xhr.send()
// fetch
fetch(url)
    .then(response => {
        if (response.ok) {
            response.json()
        }
    })
    .then(data => console.log(data))
    .catch(err => console.log(err))
复制代码
```

看起来好像是方便点，then链就像之前熟悉的callback。

在MDN上，讲到它跟jquery ajax的区别，这也是fetch很奇怪的地方：

> 当接收到一个代表错误的 HTTP 状态码时，从 fetch()返回的 Promise 不会被标记为 reject， 即使该 HTTP 响应的状态码是 404 或 500。相反，它会将 Promise 状态标记为 resolve （但是会将 resolve 的返回值的 ok 属性设置为 false ）， 仅当网络故障时或请求被阻止时，才会标记为 reject。 默认情况下, fetch 不会从服务端发送或接收任何 cookies, 如果站点依赖于用户 session，则会导致未经认证的请求（要发送 cookies，必须设置 credentials 选项）.

突然感觉这还不如jquery ajax好用呢？别急，再搭配上async/await将会让我们的异步代码更加优雅：

```
async function test() {
    let response = await fetch(url);
    let data = await response.json();
    console.log(data)
}
复制代码
```

看起来是不是像同步代码一样？简直完美！好吧，其实并不完美，async/await是ES7的API，目前还在试验阶段，还需要我们使用babel进行转译成ES5代码。

还要提一下的是，fetch是比较底层的API，很多情况下都需要我们再次封装。 比如：

```
// jquery ajax
$.post(url, {name: 'test'})
// fetch
fetch(url, {
    method: 'POST',
    body: Object.keys({name: 'test'}).map((key) => {
        return encodeURIComponent(key) + '=' + encodeURIComponent(params[key]);
    }).join('&')
})
复制代码
```

由于fetch是比较底层的API，所以需要我们手动将参数拼接成'name=test'的格式，而jquery ajax已经封装好了。所以fetch并不是开箱即用的。

另外，fetch还不支持超时控制。

哎呀，感觉fetch好垃圾啊，，还需要继续成长。。

### axios

axios是尤雨溪大神推荐使用的，它也是对原生XHR的封装。它有以下几大特性：

- 可以在node.js中使用
- 提供了并发请求的接口
- 支持Promise API

简单使用

```
axios({
    method: 'GET',
    url: url,
})
.then(res => {console.log(res)})
.catch(err => {console.log(err)})
```

并发请求,官方的并发例子：

```
function getUserAccount() {
  return axios.get('/user/12345');
}

function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}

axios.all([getUserAccount(), getUserPermissions()])
  .then(axios.spread(function (acct, perms) {
    // Both requests are now complete
  }));
```

axios体积比较小，也没有上面fetch的各种问题，我认为是当前最好的请求方式 

详情参考[官方文档](https://cn.vuejs.org/v2/cookbook/using-axios-to-consume-apis.html)

#二次封装axios

首先创建一个request.js,内容如下：

```js
import axios from 'axios';
import Qs from 'qs';


function checkStatus(err) {
    let msg = "", level = "error";
    switch (err.response.status) {
      case 401:
        msg = "您还没有登陆";
        break;
      case 403:
        msg = "您没有该项权限";
        break;
      case 404:
        msg = "资源不存在";
        break;
      case 500:
        msg = "服务器发生了点意外";
        break;
    }
    try {
      msg = res.data.msg;
    } catch (err) {
    } finally {
      if (msg !== "" && msg !== undefined && msg !== null) {
        store.dispatch('showSnackBar', {text: msg, level: level});
      }
    }
    return err.response;
  }
  
  function checkCode(res) {
    if ((res.status >= 200 && res.status < 400) && (res.data.status >= 200 && res.data.status < 400)) {
      let msg = "", level = "success";
      switch (res.data.status) {
        case 201:
          msg = "创建成功";
          break;
        case 204:
          msg = "删除成功";
          break;
      }
      try {
        msg = res.data.success;
      } catch (err) {
      } finally {
        
      }
      return res;
    }

    return res;
  
  }

//这里封装axios的get,post,put,delete等方法
export default {
    get(url, params) {
      return axios.get(
        url,
        params,
      ).then(checkCode).catch((error)=>{console.log(error)});
    },
    post(url, data) {
      return axios.post(
        url,
        Qs.stringify(data),
      ).then(checkCode).catch(checkStatus);
    },
    put(url, data) {
      return axios.put(
        url,
        Qs.stringify(data),
      ).then(checkCode).catch(checkStatus);
    },
    delete(url, data) {
      return axios.delete(
        url,
        {data: Qs.stringify(data)},
      ).then(checkCode).catch(checkStatus);
    },
    patch(url, data) {
      return axios.patch(
        url,
        data,
      ).then(checkCode).catch(checkStatus);
    },
  };
```

创建一个api.js,存放后端的接口：

```js
//导入上面的request模块
import request from './request';

//声明后端接口
export const urlUserPrefix = '/v1/users';
export const urlProductPrefix = '/v1/products';

//使用前面封装好的方法，调用后端接口
export const getUserslInfoLast = data => request.get(`${urlUserPrefix}`, data);
export const getProductsInfo = data => request.get(`${urlProductPrefix}`, data);

```

 

在.vue文件中使用定义的方法，获取后端接口的数据：

```js
export default  {

    components: {
    chart: ECharts,
   
  },
  store,
    name: 'ResourceTypeLine',
    data: () =>({
       seconds: -1,
      
        //define dataset
   apiResponse:{},
   

    initOptions: {
        renderer: options.renderer || 'canvas'
      },
      
    mounted:function() {
   
this.fTimeArray = this.getFormatTime()

//调用method里面的方法
this.getUserInfo()
   
    },
    methods: {
//异步方式调用后端接口
       async getUserInfo() {
        const resThis = await urlUserPrefix({
          params: {
              //get的参数在这里添加
            beginTime: this.fTimeArray[0],
            endTime: this.fTimeArray[1],
          }
         
        });
        this.apiResponseThisMonth = resThis.data
        try {
        } catch (err) {
          console.log(err);
        } 
      },
```



# 开发环境配置跨域

为了更方便地与后台联调，需要在用vue脚手架创建地项目中，在config目录地index.js设置proxytable来实现跨域请求，具体代码如下：

```js
module.exports = {
  build: {
    env: require('./prod.env'),
    index: path.resolve(__dirname, '../dist/index.html'),
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: '.',
    productionSourceMap: false,
    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false,
    productionGzipExtensions: ['js', 'css'],
    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  },
  dev: {
    env: require('./dev.env'),
    port: 8080,
    // hosts:"0.0.0.0",
    autoOpenBrowser: true,
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    //配置跨域请求,注意配置完之后需要重启编译该项目
    proxyTable: {
      //请求名字变量可以自己定义
      '/api': {
        target: 'http://test.com', // 请求的接口域名或IP地址，开头是http或https
        // secure: false,  // 如果是https接口，需要配置这个参数
        changeOrigin: true,// 是否跨域，如果接口跨域，需要进行这个参数配置
        pathRewrite: {
          '^/api':""//表示需要rewrite重写路径  
        }
      }
    },
    // CSS Sourcemaps off by default because relative paths are "buggy"
    // with this option, according to the CSS-Loader README
    // (https://github.com/webpack/css-loader#sourcemaps)
    // In our experience, they generally work as expected,
    // just be aware of this issue when enabling this option.
    cssSourceMap: false
  }
}
```

# vue 项目打包部署，通过nginx 解决跨域问题

​     最近将公司vue 项目打包部署服务器时，产生了一点小插曲，开发环境中配置的跨域在将项目打包为静态文件时是没有用的 ，就想到了用 nginx 通过反向代理的方式解决这个问题，但是其中有一个巨大的坑，后面会讲到。

#### 前提条件

liunx 下 nginx 安装配置（将不做多的阐述，请自行百度）

#### 配置nginx

- 通过 Xshell 连接 liunx 服务器 ，打开 nginx.conf 配置文件，或通过 WinSCP 直接打开并编辑nginx.conf文件 ，这里我选择后者 。（具体配置文件的路径根据你安装时决定）
- 在配置文件中新增一个server

```
# 新增的服务 
	# 新增的服务
	server {
		listen       8086; # 监听的端口

		location / {
			root /var/www;  # vue 打包后静态文件存放的地址
			index index.html; # 默认主页地址
		}
	
		location /v1 {
			proxy_pass http://47.106.184.89:9010/v1; # 代理接口地址
		}
	
		location /testApi {
			proxy_pass http://40.106.197.89:9086/testApi; # 代理接口地址
		}
		error_page   500 502 503 504  /50x.html;
		location = /50x.html {
			root   html;
		}

	}
复制代码
```

- 解释说明

/var/www是我当前将vue 文件打包后存放在 liunx下的路径 ，

 当我们启动 nginx 后 就可以通过http://ip地址:8086/访问到vue 打包的静态文件。

2.`location /v1 指拦截以 `v1` 开头的请求，http请求格式为 `http://ip地址:8086/v1/***`,这里有一个坑！一定要按照上面的配置文件**：proxy_pass http://47.106.184.89:9010/v1; # 代理接口地址**，如果你像我一开始写的**proxy_pass http://47.106.184.89:9010/; # 代理接口地址**，你永远也匹配不到对应的接口！。

proxy_pass http://47.106.197.89:9093/v1;` 当拦截到需要处理的请求时，将拦截请求代理到的 接口地址。

# webpack打包

下面是config/index.js配置文件

```
// see http://vuejs-templates.github.io/webpack for documentation.
var path = require('path')

module.exports = {
  build: {
    env: require('./prod.env'),
    index: path.resolve(__dirname, '../dist/index.html'),
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: '.',
    productionSourceMap: false,
    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false,
    productionGzipExtensions: ['js', 'css'],
    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  },
  dev: {
    env: require('./dev.env'),
    port: 8080,
    // hosts:"0.0.0.0",
    autoOpenBrowser: true,
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    //配置跨域请求,注意配置完之后需要重启编译该项目
    proxyTable: {
      //请求名字变量可以自己定义
      '/api': {
        target: 'http://billing.hybrid.cloud.ctripcorp.com', // 请求的接口域名或IP地址，开头是http或https
        // secure: false,  // 如果是https接口，需要配置这个参数
        changeOrigin: true,// 是否跨域，如果接口跨域，需要进行这个参数配置
        pathRewrite: {
          '^/api':""//表示需要rewrite重写路径  
        }
      }
    },
    // CSS Sourcemaps off by default because relative paths are "buggy"
    // with this option, according to the CSS-Loader README
    // (https://github.com/webpack/css-loader#sourcemaps)
    // In our experience, they generally work as expected,
    // just be aware of this issue when enabling this option.
    cssSourceMap: false
  }
}

```



# Dockerfile

打包Docker镜像

```shell
FROM Nginx:base

MAINTAINER author <hantmac@outlook.com>

WORKDIR /opt/workDir
RUN mkdir /var/log/workDir

COPY dist /var/www

ADD  nginx/default.conf /etc/nginx/conf.d/default.conf

ENTRYPOINT nginx -g "daemon off;"
```

# 后记

这是首次接触前端的第一个项目，期间经历了从后端接口开发，前端框架选型（一度想要用react，后来还是放弃），熟悉Vue，到组件开发，webpack构建，Nginx部署，Docker发布的完整过程。虽然页面比较简单，但是期间的采坑无数，后面还要继续努力！



<hr />

