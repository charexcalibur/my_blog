---
title: 实现vue-cli3根据环境在打包时自动替换请求地址baseUrl
date: 2019-07-26 21:42:47
tags:
  - vue-cli3
thumbnail: https://cdn.axis-studio.org/UNADJUSTEDNONRAW_thumb_79c.jpg
---

解决了一个项目在开发构建时的痛点。根据项目构建时的 `env` 不同，自动去替换请求的域名地址。

# 环境
  - vue-cli 3.1.1
  - vue-admin-template 4.1.0

# 问题复现

目前的项目是基于 vue-admim-template 进行开发的。由于前后端都我一个人承包。所以环境分的很简单，一个本地开发环境，同时起 vue 项目和 django server。一个 vps 做线上环境。

问题出现在 vue 项目本地开发的时候 api 用的也是本地 django server 提供的 api。所以地址就是 `127.0.0.1` 。每次上到线上环境都要手动去修改每个 api 的地址。虽然可以使用搜索全局匹配替换，但还是感觉很难受，很蠢。

故，想办法在项目构建的时候去根据环境自动更换请求地址。

# 解决思路

进过一番查找，在 vue-admim-template 中的 `/src/utils/requests.js` 中发现创建的 axios 实例中 `baseURL` 引用的是 `process.env.VUE_APP_BASE_API` 。所以当打包时根据 `env` 判断环境然后改变 `process.env.VUE_APP_BASE_API` 就可以了。

```js
const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API, // url = base url + request url
  withCredentials: false, // send cookies when cross-domain requests
  timeout: 5000 // request timeout
})
```

# 具体实现

在 `src` 目录下新建一个 `setBaseUrl.js` 文件

```js
module.exports = () => {
  let baseUrl = ""

  switch(process.env.NODE_ENV) {
    case 'development':
      baseUrl = 'http://127.0.0.1:8000/'
      break
    case 'production':
      baseUrl = 'http://47.98.243.73:8080/'
      break
  }

  return baseUrl
}
```

然后在 `vue.config.js` 中引入, 执行并赋值给 `process.env.VUE_APP_BASE_API`

```js
const baseUrl = require('./src/setBaseUrl.js')

process.env.VUE_APP_BASE_API = baseUrl()

```

最后将原先请求方法中的 api 地址中的域名以及端口移除，只保留请求路径即可。

# 本文参考

- 封面图：miku
- [vue-cli 文档](https://cli.vuejs.org/zh/)
- [vue-cli3实现分环境打包步骤（给不同的环境配置相对应的打包命令）](https://www.cnblogs.com/XHappyness/p/9337229.html)