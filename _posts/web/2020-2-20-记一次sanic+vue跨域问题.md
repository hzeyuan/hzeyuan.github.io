---
layout: post
title: "[web]记一次sanic+vue跨域问题"
subtitle: '记一次sanic+vue跨域问题'
author: "Hzy"
header-style: text
tags:
  - web
  - Sanic
---


### 这两天在琢磨用sanic配合vue来写一个网站，果然马上就遇到问题了...


> Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' he

>跨域问题..


#### 话不多说，刚解决，我现在已经累了... 直接进入正题。


### 1. vue 前端 如何处理跨域问题

> 搜索了很久，发现通过下面的方法可以解决这个前端的问题。

> 参考文章：https://www.cnblogs.com/s313139232/p/10598071.html

* 1.在vue.config.js里配置代理，类似于下面这样

```
module.exports = {
  runtimeCompiler: true,
  publicPath: '/', // 设置打包文件相对路径
  devServer: {
    // open: process.platform === 'darwin',
    // host: 'localhost',
    port: 8071,
    // open: true, //配置自动启动浏览器
    proxy: {
      '/api': {
        target: 'http://127.0.0.1:8100/', //对应自己的接口
        changeOrigin: true,
        ws: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    }
   },
}
```

* 2. main.js 里设置超时和baseurl

```
axios.defaults.timeout = 5000 // 请求超时
axios.defaults.baseURL = '/api/'  // api 即上面 vue.config.js 中配置的地址
```

### 2.后端sanic如何处理跨域

> 一开始呢，我是设置响应头，像下面这样

```
return HTTPResponse(status=200,headers={"Access-Control-Allow-Origin": "*",
"Cache-Control": "no-cache"}, body="13")
```

> 结果没用

-----

> 后来想了一下，flask有cors,sanic应该也有，果然发现了sanic_cors

```
pip install -U sanic_cors #装上
from sanic_cors import CORS  #加入扩展
CORS(app)
# route记得允许options方式，然后就能正常响应了。
```

> 好啦，大致就这样，明天继续努力。



## 继续补充，http跨域时为什么有options请求
>在正式请求前，会通过options方式请求来进行一次预检，询问是否在服务器的白名单里，允许之后，才能有接下来的正式通信。
